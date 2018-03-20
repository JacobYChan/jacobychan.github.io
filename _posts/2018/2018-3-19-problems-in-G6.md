---
layout: post
title: Antv/G6的一些使用心得
date: 2018-3-19 17:00:00 +0800
categories: document
tag: 问题
#大类配置
categories: 技术文章
---

* content
{: toc}

### 前言

#####   最近公司有一个可视化项目需要画一些图形，阿里的antv/g2这个工具用于画图还是不错的，也比较成熟，里面包含了很多传统的图形 包括柱状图，折线图，饼图，点图等等，也包括热力图，仪表盘，地图雷达图等等，能够满足绝大多数的图形要求，但是项目里有一个数列结构的关系图，有点像组织架构图，就类似于下面这样：

<br/>

<!-- ![示例](/styles/images/example.png '示例') -->
<div style="text-align:center"><img src="/styles/images/example.png" style="width:500px;"></div>
<br/>

##### 很遗憾，G2里并没有类似的图形案例，找了`Echarts`相关的案例也没有相关的图形，无奈之余在阿里的antv的另一款产品G6中发现了曙光，`G6是antv团队推出的另一款产品，主打关系图，感觉跟G2形成了很好的互补`。


##### 看了一下官方文档介绍后，发现antv/G6并没有G2成熟，后来了解了一下由于G6的团队太小，人手不够，所以并没有完善。


{% highlight bash %}
* 相关文档不健全，比较混乱
* 方法说明的不完全，相关参数说明也不够详细
{% endhighlight %}



-----------------------------
### 学习心得

##### G6总体而言分为三大块，Node/Edge/Net，首先要先建立一个Net，作为整个画布的容器，Net有一些属性和方法
<br>

#### 1、net

##### net属性：

    this.net = new G6.Net({
      id: 'mountNode', // 要渲染的画布id
      width: 600, // 宽度
      height: window.innerHeight, // 高度
      fitView: 'tc', // 画布适配的对齐方式
      plugins: [dagre], // 使用dagred插件
      shape: '', // 形状
      label: '',    // 标注
      color: '',   // 颜色
      style: '',   // 样式
      grid: {
         forceAlign: true, // 是否支持网格对齐
         cell: 10          // 网格大小
      },
      grid: null,  // 不显示网格
      mode: 'none', // 整个画布不能拖动，包括图形
    })

<br>
##### net方法：
    /**
    * 更新锚点
    * @param {Object}  node          节点
    * @param {Object}  anchorIndex   目标锚点索引
    * @param {Object}  cfg           配置项
    * @return {Object} self
    */

    net.showAnchor(node, anchorIndex, {
      linkable,  // {Boolean} 是否可以连接
      style,     // {Object}  锚点样式
      hoverStyle, // {Object}  鼠标悬浮锚点样式
    });

    /**
    * 显示锚点
    * @param {Object} node 子项
    * @return {Object} self
    */
    net.showAnchor(node);


    /**
    * 隐藏锚点
    * @param {Object} node 子项
    * @return {Object} self
    */
    net.hideAnchor(node);


    /**
    * 增加节点、边
    * @param  {String} type 类型：node ,edge
    * @param  {Object} model
    * @return {Item}   item 子项
    */
    net.add(type, model);


    /**
    * 渲染数据
    */
    net.source(this.nodes, this.edges)


net在开发的时候，如果是在Vue里使用，不妨把net定义成全局的变量，方便在不同方法里引用。

    this.net = new G6.Net({
      id: 'mountNode',
      height: window.innerHeight,
      fitView: 'tc',
      grid: null
    })

-------------------------------------

#### 2、node

node作为G6中最为重要的知识点之一，每一个节点的样式和配置都是通过node来定义的

    let newNode = {
      'id': 'node' + Number(this.nodeNum + 1), // id是定义节点的唯一标识
      'name': '易购销售额', // 自定义的
      'count': 2221902, // 自定义的
      'x': model.x, // 节点的x轴坐标
      'y': model.y + 120, // 节点的y轴坐标
      'status': 0,  // 自定义的
      'precent': 100, // 自定义的
      'shape': 'customNode2',  // 节点采用的模板
    }

注册节点模板使用`G6.registerNode(name,  {// 配置选项}`来定义，如果有多种node模板的话可以事先定义多个，配置选项里有一些内置的方法和属性
* 使用`getAnchorPoints () {}`来定义锚点，里面可以配置锚点的属性，可以定义连接线是从节点的哪个位置来连接
        return [
          [0, 0.5],   // 左边中点 索引为 0
          [1, 0.5]    // 右边中点 索引为 1
          [0.5, 0, invalidCfg], // 上边中点 索引为 0
          [0.5, 1, invalidCfg] // 下边中点 索引为 1
        ]
* `getHtml: (cfg) => {let model = cfg.model}`，model就是这个模型的实体，里面可以获取到这个模型的所有属性和方法，包括自定义的属性

--------------------------

#### 3、edge

edge作为连接每个节点的连接线，也是非常重要，edge内部有自己的一些属性和方法。

1、定义edge

    let newEdge = {
      'source': model.id, // 源节点的id
      'target': 'node' + Number(this.nodeNum + 1), // 目标节点的id
      'id': 'edge' + this.nodeNum,  // 作为每个edge的唯一标识
      'precent': 100,
      'shape': 'VHV', // 连接线的形状，使用这种折线不用管起始折点和终点的坐标位置
      "color":'red',  // 颜色定义
      'label':{text:'吴 肖 2×K311',fill: 'green',fontSize:14},  // 连接线上的文字
      'style':{lineDash: [8, 5]}, // 定义连接线的样式
      'controlPoints': [  // 连接线可添加多个折点，定义每个折点的坐标
        {
          'x': model.x,
          'y': model.y
        },
        {
          'x': model.x,
          'y': model.y + 120
        }
      ]
    }

  2、edge方法

     显示： show

     隐藏： hide

     获取关键形： getKeyShape

     获取图形图组： getGroup

     获取包围盒： getBBox

     获取数据模型： getModel

     获取绘制配置项： getShapeCfg

---------------------------------

### 遇到的坑

##### &nbsp;&nbsp;在使用过程中难免会遇到一些坑，毕竟这个项目还不成熟，相关文档也不是太完善，所以使用起来只能靠自己去摸索，还好在使用过程中经过自己的探索和研究还算顺利，如果大家遇到一些在文档上找不到的问题，不妨去github上看看前人有没有帮你踩过坑。附：[github问题](https://github.com/antvis/g6/issues)

------------------------------------------

##### 1、数据的存储与读取

在存储数据时使用的是net的save方法，读取数据使用的是net的read方法，因为数据的格式是json类型的，所以存储时下意识的使用`json.stringify()`格式化一下，但是此时读取的时候要使用`json.parse()`转换一下，否则读取数据的时候会报错

    // 数据的存储
    const saveData = this.net.save()
    const json = JSON.stringify(saveData, null, 2);

    // 数据的读取
    let data = JSON.parse(json)
    this.net.read(data)

-----------------------------------

##### 2、渲染数据的两种方法

前面我们提到可以用read()来读取数据，同时也可以使用net.source(nodes, edges)来渲染数据，但是两种渲染的方式是不一样的，read读取数据时内部还有source字段，如果没有这个字段就会报错

    {
      'source': {
        'edges': this.edges,
        'nodes': this.nodes
      }
    }

------------------------------------

##### 3、net在使用插件时，很多功能就会受限，有些配置会不起作用，比如this.net.edge().shape('VHV')这个规定连接线形状的设置就会失效，所以除非插件的效果能完全满足需求，否则尽量不要用插件。

    plugins: [dagre], // 使用dagred插件


-----------------------------------

##### 4、在使用`this.net.edge().shape('VHV')` 来定制线段形状时能够保证新添加的节点按照要求来设置形状，但是当把数据保存后再读取时发现之前的线段形状并没有生效，又还原成之前的直线了，这个时候需要在每条边添加的时候就要设置好边的形状，否则读取出来的数据是不会生效的。

    let newEdge = {
      'shape': 'VHV', // 连接线的形状，使用这种折线不用管起始折点和终点的坐标位置
    }

----------------------------------

### 最终效果

![关系图](/styles/images/relation.png '关系图效果')