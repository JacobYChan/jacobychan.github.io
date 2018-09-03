---
layout: post
title: js原型链总结
date: 2018-9-02 19:00:00 +0800
categories: document
tag: 总结
#大类配置
categories: 技术文章
---

* content
{: toc}

#### 1.什么是原型

-----------------------------------

通常在定义类的时候会声明一些属性和方法，通常在对这个类进行实例化的时候，实例对象会拥有这个类的属性和方法，但是每实例化一个类都会单独开辟一块内存空间，它们的方法虽然是一样的，但是本身并不是指向同一个对象方法。

这个时候我们可以把公共的方法挂载到原型上面去，也就是`prototype`属性，这样做的好处是可以解决数据共享的问题，节省内存空间！

-----------------------------------

`使用示例如下:`

    function Person(name, age) {
      this.name = name;
      this.age = age;
      this.say = function() {
        console.log('说话')
      }
    }

    const p1 = new Person()
    const p2 = new Person()

    console.log(p1.say === p2.say) // false

虽然都是从Person实例化的两个对象，但是他们的say方法却并不相等，每个实例对象中都有自己的say方法。说明实例化的时候开辟了两块内存，如果有100个实例化对象，就需要开辟100个内存空间，效率是非常低下的。

为了解决这个问题，我们可以把他们共有的方法挂载到原型`prototype`下，来看下面这段代码：

    Person.prototype.say = function () {
      console.log('说话')
    }

    const p1 = new Person()
    const p2 = new Person()

    console.log(p1.say === p2.say) // true

这个时候是相等的，说明是共用的同一个方法，方法被共享了。这个prototype就是原型对象。




-------------------------------------------

#### 2.原型对象，构造函数和实例对象的关系

构造函数：通过自定义类，类里面可以有一些属性和方法。构造函数中的prototype中有constructor叫构造器,指向的是该原型对象所在的构造函数。

原型对象：构造函数中有一个属性叫prototype，该属性也是一个对象，该属性是标准属性，是给程序员用的，这个属性叫做原型对象 <br>

实例对象：通过构造函数实例化出来的对象叫做实例对象，实例对象中有一个__proto__属性，这个属性也是一个对象，这个属性是非标准属性（隐式原型），给浏览器使用，这个属性也叫做原型对象。

关系：实例对象的__proto__指向的是所在的构造函数的prototype。构造函数中的prototype是对象，里面有__proto__也是对象，这个对象指向的肯定是某个构造函数的prototype。

---------------------------------------

`Object.prototype`中的`__proto`__是`null`

`注意：不同的浏览器对__proto__属性的支持不一样，通常我们使用getPrototypeOf方法来获取`

-----------------------------------------

#### 3.原型链

js中的函数实际上都是通过Function这个构造函数来创建的(来实例化出来的)，Function中有__proto__指向的是Function的prototype。

`Function`中的`prototype`中的`__proto__`指向的是`Object.prototype`。实际上所有的构造函数的`__proto__`都是指向`Function`的`prototype`。

`Object`中的`__proto__`指向的也是`Function`的`prototype`，所以所有的构造函数的`prototype`的`__proto__`都是指向`Object.prototype`。

------------------------------------------

来看下面的代码：

     console.log(Object.__proto__===Function.prototype); // true
     console.log(Function.prototype.__proto__ === Object.prototype) // true

构造函数是函数也是对象,所以,里面有prototype,也有__proto__，原型链就是实例对象和原型对象之间的一种关系而已，最终为继承做准备


--------------------------------------

#### 4.继承

js不是真的面向对象语言,继承是面向对象中的一个特性,继承是针对面向对象语言说的。继承是类与类之间的关系,js中没有类,通过构造函数来模拟类。

----------------------------------------

看下面的代码：

    //构造函数
    function Person(name,age) {
      this.name=name;
      this.age=age;
    }
    //原型添加方法
    Person.prototype.eat=function () {
      console.log("我要吃东西");
    };
    //学生的构造函数
    function Student(sex) {
      this.sex=sex;
    }
    Student.prototype.play=function () {
      console.log("来一起玩啊");
    };
    //希望学生也有吃的方法
    //通过原型实现继承
    Student.prototype=new Person("小明",20);
    var stu=new Student("男");
    console.log(stu.name);
    console.log(stu.age);
    console.log(stu.sex);
    //此时,父级的构造函数中的name和age属性被继承过来了,所以,实例对象stu是可以使用的
    //此时父级的构造函数的原型中的eat方法也 被继承过来了
    stu.eat();

---------------------------------

如果实例对象中的属性或者方法和原型对象中的重复了,那么此时使用的是实例对象中,但是不会修改原型对象中的属性的值,那么该对象就有这个属性了
* 实例对象访问属性的时候,先在该实例对象中查找,有则使用,没有就去原型中找,找到了则直接使用,如果没有找到,则是`undefined`

* 继承:集成后,原型的指向改变,实例对象中的`__proto__`的指向也会发生改变,这样从原型对象中的属性和方法就继承过来了

但是上面通过`Student.prototype = new Person('小明', 20)`来实现继承时，固定传值都是小明和20，实例化很多对象时这两个值都不会变化，所以我们需要用到`call`的方式来实现继承

    function Person(name, age) {
      this.name = name;
      this.age = age;
    }

    Person.prototype.eat = function () {
      console.log('吃')
    }

    function Student(name, age, sex) {
      Person.call(this, name, age) // 借用构造函数,call方法是改变了this的指向,改变this的指向，继承父元素的属性
      this.sex = sex
    }
    // 继承父元素的方法
    Student.prototype = Person.prototype
    // 或者使用下面这种方式
    Student.prototype=new Person();

    var student1 = new Student('小明', 20, '男')
    var student2 = new Student('小红', 22, '女')

    console.dir(student1) // 小明，20， 男
    console.dir(student2) // 小红，22， 女


