---
title: Vue 介绍
date: 2017-10-26 11:42:53
tags:
- Vue
categories:
- Tech
---

> 官方文档：[https://cn.vuejs.org/v2/guide/](https://cn.vuejs.org/v2/guide/)

简介
----------

* Vue.js（后面简称Vue）是国人开发的，用于Web前端开发，可以做UI、界面路由等，利用插件还可以实现HTTP请求等更加丰富的功能
* Vue比较轻量，而且使用简单，上手很容易，同时功能也很丰富
* 目前感觉开发大型前端项目可能比较费力，当然也很可能是我目前应用经验不足，认知有限

#### 与其它常见框架/工具/名词的关系
> 后记：写完这段后才发现文字太多了，这些和学习Vue没啥太大关系，所以，可以不看……

* `Node.js`是一个Javascript独立运行环境，可以理解成一个Javascript解释器，就像Python语言和Python的运行环境一样。不过Node.js也不仅仅是个运行环境，他还包含了丰富的类库，而且总能以最快的速度支持最新的Javascript标准。Node.js目前基本上可以完成像Java、Python等能干的所有事情了。浏览器中不会支持Node.js的这些类库。这玩意和Vue没啥直接联系，不是一个层面的。
* `npm`是一个Node.js下的工具，性质有点像Python的`pip`，或者Java的`Maven`，不过npm可以指定将某个库安装到系统、还是安装给当前用户、甚至只安装给当前的项目。
* `webpack`可以理解成是一个发布工具，类似Maven的`mvn package`指令。在这个过程中可以自定义很多处理过程，比如将js脚本混淆之类的。Vue的单文件组件（就是.vue文件）就可以通过webpack来实现，如果不用webpack，我们就得把组件写到js代码里，这样看上去会比较乱。类似webpack的工具，还有browserify。另外，还有一些功能上有些重合的工具，比如gulp/grunt等，还有百度开发的FIS，这个圈子有点乱，我经验也不多，所以大家想有更多了解的话还是自己查些资料吧，别被我误导。
* `AngularJS`和`React`，这两个和Vue是同一类东西了。AngularJS是Google出品，一般简称为Angular，目前的主流应该是Angular2；React是Facebook出品，这两个看上去都比较复杂，我没有尝试过。
* `ECMAScript`是Javascript语言的核心，是由**ECMA**这个组织维护的。我们可能平时会见到`ES5`、`ES6`这样的字眼，这指的就是ECMAScript的第5和第6个版本。这就有点像C++有个标准委员会在制定C++的标准一样。Node.js最新的版本对ES6的支持度是很高的。各浏览器对ES标准的支持情况可以[参考这里](http://kangax.github.io/compat-table/es6/)，顺便说一句，由于IE支持跟不上，所以很多新版ES的特性在平时开发时都不得不放弃使用。其实我们还经常看到ES2015这个词，这都是不同版本的标准而已，要是想了解这些标准更详细的历史，可以参考[http://es6.ruanyifeng.com/#docs/intro](http://es6.ruanyifeng.com/#docs/intro)。Vue只是个工具，所以和ES版本的关系不是很大。
* `jQuery`和Vue相比，灵活性更大一些，看上去会更底层一点，可以直接操作DOM，但由于Vue在内部处理时很多时候也要直接操作DOM，所以，jQuery与Vue如果同时使用的话，经常会有一些奇怪的现象发生，因此，不建议这两个东西同时使用。如果非要同时使用的话，一般来讲，要先初始化Vue，再使用jQuery，尤其是很多基于jQuery的组件（比如我们用到的daterangepicker、EChart），如果在Vue之前初始化，很可能被后续的Vue操作影响。

### 开始介绍Vue

> * Vue官方文档其实介绍得很好，我就是看官方文档入门的，不过我现在要是照着官方文档读一遍就太没意思了，所以，我以我的角度再来介绍一下吧。
> * 我介绍的内容比较肤浅，以应用为主要目的，对Vue内部的运行机制说得比较少，所以，大家如果想深入学习，请移步到[官网](https://cn.vuejs.org/v2/guide/)。
> * 配套工程代码在[这里下载](http://onpyrjcca.bkt.clouddn.com/02e90f70ae084da3a22fc110f30b4069.zip)，或是SVN目录：/server/demos/vue-demo
> * Vue有很多的插件可以使用，可以参考[https://github.com/vuejs/awesome-vue](https://github.com/vuejs/awesome-vue)，后文中用到的Vue-resource，就是这其中的一个。这些插件构成了Vue的生态系统。

##### 最简单的例子

* 实例文件：basic.html

这个例子主要完成的功能：

* 引入Vue
* 初始化Vue对象
* 模板信息显示
* 修改Vue成员

##### 表单

* 实例文件：input.html

这个例子主要完成的功能：

* 将Vue成员绑定到表单控件
* 表单控件到Vue成员的双向绑定
* 不同控件双向绑定的效果

##### 条件渲染

* 实例文件：if.html

这个例子主要完成的功能：

* v-show的效果
* v-if的效果
* "if-else"语法

> v-show与v-if指令区别，在于v-show只是控制了元素的display属性，而元素本身无论是否显示，都存在于DOM序列中；v-if指令则控制元素是否存在，而不是要不要显示。

##### 列表渲染

* 实例文件：for.html

这个例子主要完成的功能：

* 循环获取列表中每一个对象
* 获取列表中对象时，同时获取数组下标

##### 样式绑定

* 实例文件：style.html

这个例子主要完成的功能：

* 以字典的方式绑定Style属性
* 以字典的方式绑定Class属性
* 以数组方式绑定Style或Class属性

##### 方法、计算属性、观察者

* 实例文件：methods.html

这个例子主要完成的功能：

* 方法是如何定义和使用的
* 计算属性是如何定义和使用的
* 观察者是如何定义和使用的

> 方法和计算属性的区别，在于方法每次都会被执行，而计算属性会被缓存，直到成员值被改变。这一点可以通过控制台来做测试

##### 事件绑定

* 实例文件：event.html

这个例子主要完成的功能：

* 事件绑定的语法

##### 请求远程数据

* 实例文件：remote.html

这个例子主要完成的功能：

* 通过vue-resource请求远程数据，并将数据赋给vue对象

vue-resource是一个第三方的Vue库，用来进行HTTP请求，使用方法可以参考[官网介绍](https://github.com/pagekit/vue-resource/blob/develop/docs/http.md)

##### 一个较为完成的例子

* 实例文件：full-eg.html

这个例子综合使用了上述很多的技巧，使用很少的代码就实现了平时开发中常见的列表页功能，开发速度快到飞起！
