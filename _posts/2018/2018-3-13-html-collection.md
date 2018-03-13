---
layout: post
title:  如何将HTMLcollection伪数组转换成数组
date:   2018-03-13 17:00:00 +0800
categories: document
tag: 问题
#大类配置
categories: 技术文章
---

* content
{:toc}

### 前言

> 通常我们在获取dom元素集合的时候会产生一些问题，特别是获取的是一个数组的时候，我们想使用数组的join、pop、shift等方法的时候会报错，提示我们没有该方法

{% highlight bash %}
例如：var list = document.getElementsByClassName('myclass')
      console.log(list.join(',')) 报错，没有join方法
{% endhighlight %}

##### 我们把list打印出来看下`console.log('list')`,发现长这样`HTMLCollection`，是这个类型，并不是传统的数组类型，所以它并没有传统操作数组的一些方法，但是他有length属性，也就是可以获取到数组的长度

### 解决办法

{% highlight bash %}

我们使用js数组原型链的call方法改变它
`Array.prototype.slice.call(list, 0)`
这样改造之后就转化成一个真正的数组了，可以使用数组的相关方法，如果不能使用Array的话就var array = new Array()，array.prototype.slice.call(list, 0)就可以了，可以简化写成[].slice.call(list, 0)，
我们可以封装成一个方法，方便使用。

var toArray = function(s){
    try{
        return Array.prototype.slice.call(s);
    } catch(e){
        var arr = [];
        for(var i = 0,len = s.length; i < len; i++){
            //arr.push(s[i]);
               arr[i] = s[i];  //据说这样比push快
        }
         return arr;
    }
}

{% endhighlight %}

