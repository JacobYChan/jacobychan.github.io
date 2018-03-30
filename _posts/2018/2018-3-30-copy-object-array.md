---
layout: post
title: 对象与数组的拷贝
date: 2018-03-30 09:00:00 +0800
categories: document
tag: 问题，教程
#大类配置
categories: 技术文章
---

* content
{: toc}



问题
----------------------------------

 最近在做项目的时候想偷懒复制一下已经存在的对象属性，因为对象的属性比较多，不想一个个又写一遍，所以就想到了复制，于是我使用了以下的方法：

    let obj = {a: 1, b: 2, c:3}
    let obj2 = obj
    obj.a = 5
    console.log(obj.a)  // 5

 问题出现了，当我改变第二个对象的内容时，第一个对象对应的属性也发生了变化，这不是我想要的结果，我的初衷是改变复制的这个对象内容而不影响第一个对象，后来查了资料才知道，这种复制其实是浅拷贝，`他们指向的是同一块内存，只是引用而已`，所以就不能这样复制了，查了相关的资料，总结了几种复制的方法。

------------------------------------

对象的复制
------------------------------------

1、传统的方法是使用for循环进行遍历，然后把属性一个个复制到新的对象里面，可以封装成一个方法进行调用，这是最传统的方法，当然也是最简单粗暴的方法，在这里就不详细介绍了

2、使用`Object.assign(target, ...sources)`方法，Object.assign()用于将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。

    const object1 = {
      a: 1,
      b: 2,
      c: 3
    };

    const object2 = Object.assign({}, object1);

    console.log(object2.c);
    // expected output: 3

`注意：如果目标对象中的属性具有相同的键，则属性将被源中的属性覆盖。后来的源的属性将类似地覆盖早先的属性。`
<br><br>
针对深拷贝，需要使用其他方法，因为 Object.assign()拷贝的是属性值。假如源对象的属性值是一个指向对象的引用，它也只拷贝那个引用值。

----------------------------------

3、对象的深拷贝我们使用JSON.parse()和JSON.stringify()方法。

    obj1 = { a: 0 , b: { c: 0}};
    let obj3 = JSON.parse(JSON.stringify(obj1));
    obj1.a = 4;
    obj1.b.c = 4;
    console.log(JSON.stringify(obj3)); // { a: 0, b: { c: 0}}


4、Object.assign()还具有合并对象的功能，将多个对象作为第二个参数传入，以,分割，则可以将所有对象的属性合并到一个对象里面。

    var o1 = { a: 1, b: 1, c: 1 };
    var o2 = { b: 2, c: 2 };
    var o3 = { c: 3 };

    var obj = Object.assign({}, o1, o2, o3);
    console.log(obj); // { a: 1, b: 2, c: 3 }

`注意：继承属性和不可枚举属性是不能拷贝的`

    var obj = Object.create({foo: 1}, { // foo 是个继承属性。
      bar: {
          value: 2  // bar 是个不可枚举属性。
      },
      baz: {
          value: 3,
          enumerable: true  // baz 是个自身可枚举属性。
      }
    });

    var copy = Object.assign({}, obj);
    console.log(copy); // { baz: 3 }
    
--------------------------------------------------

### 数组的复制

1、传统的方法还是一样遍历数组，然后依次复制

    var arr = ["a", "b"], arrCopy = [];
    for (var item in arr) arrCopy[item] = arr[item];
    arrCopy[1] = "c";
    arr   // => ["a", "b"]
    arrCopy   // => ["a", "c"]

多维数组这样复制

    function arrDeepCopy(source){
    var sourceCopy = [];
    for (var item in source) sourceCopy[item] = typeof source[item] === 'object' ? arrDeepCopy(source[item]) : source[item];
    return sourceCopy;
    }

2、使用`slice()`进行复制，arrayObject.slice(start,end)方法返回一个新的数组，包含从 start 到 end （不包括该元素）的 arrayObject 中的元素。该方法并不会修改数组，而是返回一个子数组。

在这里我们的思路是直接从数组开头截到尾

    arrCopy = arr.slice(0);
    arrCopy[1] = "c";
    arr   // => ["a", "b"] 
    arrCopy   // => ["a", "c"]

3、使用concat()进行复制，该方法不会改变现有的数组，而仅仅会返回被连接数组的一个副本。使用这种方法的思路是我们用原数组去拼接一个空内容，放回的便是这个数组的拷贝：

    arrCopy = arr.concat();
    arrCopy[1] = "c";
    arr   // => ["a", "b"] 
    arrCopy   // => ["a", "c"]

4、也可以使用`JSON.stringify(array) 然后再 JSON.parse()`，这个方法同样适用于数组
    var arr1 = [1, 2, [3, 4], {a: 5, b: 6}, 7],
    arr2 = JSON.parse(JSON.stringify(arr1));
    console.log(arr1, arr2);
    arr2[1] = 10;
    arr2[3].a = 20;
    console.log(arr1[1], arr2[1]);
    console.log(arr1[3], arr2[3]);
    var arr1 = [1, 2, [3, 4], {a: 5, b: 6}, 7],
    arr2 = JSON.parse(JSON.stringify(arr1));
    console.log(arr1, arr2);
    arr2[1] = 10;
    arr2[3].a = 20;
    console.log(arr1[1], arr2[1]);
    console.log(arr1[3], arr2[3]);

------------------------------------------------------------------

### 总结

最后针对对象和数组的深入拷贝，我们可以封装成一个方法，方便以后使用，可以复制对象和数组

`考虑到特殊情况，例如我们定义：`

    var obj = [{ "a": { "a1": ["a11", "a12"], "a2": 1 }, "b": 2 }, ["c", { "d": 4, "e": 5 }]];

这是一个由对象、数组杂合成的奇葩数组，虽然我们平时写程序基本不可能这么折腾自己，但是可以作为一种特殊情况来考虑，这样我们就可以结合之前说的方法去拓展拷贝函数：

    var objDeepCopy = function (source) {
    var sourceCopy = source instanceof Array ? [] : {};
      for (var item in source) {
          sourceCopy[item] = typeof source[item] === 'object' ? objDeepCopy(source[item]) : source[item];
      }
      return sourceCopy;
    }
    var objCopy = objDeepCopy(obj);
    objCopy[0].a.a1[1] = "a13";
    objCopy[1][1].e = "6";
    obj   // => [{ "a": { "a1": ["a11", "a12"], "a2": 1 }, "b": 2 }, ["c", { "d": 4, "e": 5 }]]
    objCopy   // => [{ "a": { "a1": ["a11", "a13"], "a2": 1 }, "b": 2 }, ["c", { "d": 4, "e": 6 }]]


-----------------------------------------------------------
