---
title: js实现观察者模式
date: 2017-04-13 17:22:21
tags: [JavaScript,设计模式]
categories: JavaScript
---

js实现观察者模式
　　观察者模式：设计该模式背后的主要动力是促进形成松散耦合。在这种模式中，并不是一个对象调用另一个对象的方法，而是一个对象订阅另一个对象的特定活动并在状态改变后获得通知。订阅者也称为观察者，而补观察的对象称为发布者或主题。当发生了一个重要的事件时，发布者将会通知（调用）所有订阅者并且可能经常以事件对象的形式传递消息。

　　思路:发布者对象需要一个数组类型的属性，以存储所有的订阅者。订阅（即注册）行为就是将新的订阅者加入到这个数组中去，则注销即是从这个数组中删除某个订阅者。此外，发布消息，就是循环遍历订阅者列表并通知他们。

　　

　　这里我的大体思路是对的，但是在发布者之外定义了一个新的类即订阅者。在订阅者中定义了一个方法getNews以便在发布者发布消息时调用该方法。然后面试官说这样太麻烦了，万一订阅者没有这个方法呢？然后我不是很懂……于是在发布消息时直接传递了参数：obj.news = msg; 然后面试官说这样不是更麻烦了吗？这样的话如果订阅者没有news这个属性怎么办？还得判断订阅者是否有news这个属性，没有的话就会出现undifined的报错。然后我就不知道该怎么做了……然后面试官为人特别nice，告诉我说可以用继承，或者是在注册时候就传入一个function。

 

下来后上网查相关，注意点如下：

1. 发送消息即通知，意味着调用订阅者对象的某个方法。故当用户订阅信息时，该订阅者需要向paper的subscribe()提供它的其中一个方法。--------这应该就是面试官所说的注册时候就传入一个方法。

2. 发布对象paper需要具有以下成员：

　　　　a、 subscribers：一个数组，存储订阅者

　　　　b、 subscribe()：注册/订阅，将订阅者添加到subscribers数组中

　　　　c、 unsubscribe()： 取消订阅。从subscribers数组中删除订阅者

　　　　d、 publish() 循环遍历subscribers数组中的每一个元素，并且调用他们注册时所提供的方法

　　　　所有这三种方法都需要一个type参数，因为发布者可能触发多个事件（比如同时发布一本杂志和一份报纸）而用户可能仅选择订阅其中一种，而不是另外一种。

代码如下：

{% codeblock lang:javascript %}
//由于这些成员对于任何发布者对象都是通用的，故将它们作为独立对象的一个部分来实现是很有意义的。那样我们可将其复制到任何对象中，并将任意给定对象变成一个发布者。

//如下实现一个通用发布者

//定义发布者对象...{}是定义一个对象
var publisher = {
    subscribers: {
        any: []         //event type: subscribers
    },
    subscribe: function(fn,type){
        type = type || 'any';
        if(typeof this.subscribers[type] === "undefined"){
            this.subscribers[type] = [];
        }
        this.subscribers[type].push(fn);
    },
    unsubscribe: function(fn,type){
        this.visitSubscribers('unsubscribe', fn, type);
    },
    publish: function(publication, type){
        this.visitSubscribers('publish',publication,type);
    },
    visitSubscribers:function(action,arg,type){
        var pubtype = type ||'any',
            subscribers = this.subscribers[pubtype],
            i,
            max = subscribers.length;
        for(i=0;i<max;i++){
            if(action == "publish"){
                subscribers[i](arg);
            } else {
                if(subscribers[i] === arg){
                    subscribers.splice(i,1);
                }
            }
        }
    }
};
//定义一个函数makePublisher()，它接受一个对象作为对象，通过把上述通用发布者的方法复制到该对象中，从而将其转换为一个发布者
function makePublisher(o){
    var i;
    for(i in publisher) {
        if(publisher.hasOwnProperty(i) && typeof publisher[i] === "function"){
            o[i] = publisher[i];
        }
    }
    o.subscribers = {any: []};
}
//实现paper对象
var paper = {
    daily: function(){
        this.publish("big news today");
    },
    monthly: function(){
        this.publish("interesting analysis","monthly");
    }
};
//将paper构造成一个发布者
makePublisher(paper);
//已经有了一个发布者。看看订阅对象joe，该对象有两个方法：
var joe = {
    drinkCoffee: function(paper) {
        console.log('Just read' + paper);
    },
    sundayPreNap : function(monthly){
        console.log('About to fall asleep reading this' + monthly);
    }
};
//paper注册joe（即joe向paper订阅）
paper.subscribe(joe.drinkCoffee);
paper.subscribe(joe.sundayPreNap,'monthly');
//即joe为默认“any”事件提供了一个可被调用的方法，而另一个可被调用的方法则用于当“monthly”类型的事件发生时的情况。现在让我们来触发一些事件：
paper.daily();      //Just readbig news today
paper.daily();      //Just readbig news today
paper.monthly();    //About to fall asleep reading thisinteresting analysis
paper.monthly();    //About to fall asleep reading thisinteresting analysis
paper.monthly();
{% endcodeblock %}

试着用函数实现：

{% codeblock lang:javascript %}
function Publisher(subscribers){
    this.subscribers = subscribers || {'any': []};
    Publisher.prototype.subscribe = function(fn, type){
        var type = type || 'any';
        if(typeof this.subscribers[type] === 'undefined'){
            this.subscribers[type] = [];
        }
        this.subscribers[type].push(fn);
    };
    Publisher.prototype.unsubscribe = function(fn, type){
        var type = type || 'any';
        for(var i=0, len = this.subscribers[type].length;  i<len; i++){
            if(this.subscribers[type][i] === fn){
                this.subscribers[type].splice(i,1);
            }
        }
    };
    Publisher.prototype.publish = function(publication, type){
        var type = type || 'any';
        for(var i=0, len = this.subscribers[type].length; i<len; i++){
            this.subscribers[type][i](publication);
        }
    };
}


var paper = new Publisher();
paper.daily = function(){
    this.publish(' this is Olympic ! ');
};
paper.monthly = function(){
    this.publish(' last month is the 28th Olympic! ', 'monthly');
};

var Jack = {
    readInMorning: function(news){
        console.log('Jack reads ' + news + ' in the morning');
    },
    readInSunday: function(news){
        console.log('Jack reads ' + news + ' on Sunday');
    }
};

var Amy = {
    readInMorning: function(news){
        console.log('Amy reads ' + news + ' in the morning');
    },
    readInSunday: function(news){
        console.log('Amy reads ' + news + ' on Sunday');
    }
};

paper.subscribe(Jack.readInMorning);
paper.subscribe(Jack.readInSunday, 'monthly');
paper.subscribe(Amy.readInMorning);
paper.subscribe(Amy.readInSunday, 'monthly');

paper.daily();          //Jack reads  this is Olympic !  in the morning
                        //Amy reads  this is Olympic !  in the morning
paper.monthly();        //Jack reads  last month is the 28th Olympic!  on Sunday
                        //Amy reads  last month is the 28th Olympic!  on Sunday

paper.unsubscribe(Jack.readInSunday,'monthly');
paper.daily();              //Jack reads  this is Olympic !  in the morning
                            //Amy reads  this is Olympic !  in the morning
paper.monthly();            //Amy reads  last month is the 28th Olympic!  on Sunday
{% endcodeblock %}