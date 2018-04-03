title: 对vue-desktop的进一步思考
date: 2016-04-05 23:11:00 +0800
update: 2016-04-05 23:11:00  +0800
author: me
#cover: -/images/default.png
tags:
    - Vue
    - Vue-desktop
    - Skia
    - Vuejs
    - Qt
    - GacUI
    - Unreal Engine
    
 ---
    
上周的xxx-native-deskop的思考是一个非常初级的想法，最近又结合虚幻引擎界面设计和vczh的GacUI的思路进行了进一步思考。

<!--more-->

UI这块使用Qt最为后端的想法写完上篇之后就被我彻底摒弃了

1. 和Qt窗口很难作数据映射，和MVVM的方式适配起来比较麻烦
2. 如果这样做了可能就是另一个Qt Quick，但是quick这个东西实在是无感
3. 窗口系统不可控，这也是虚幻、chromium、火狐各家实现自己的窗口系统的原因

后来又想用vczh的GacUI，去github速览了一下源码，GacUI的设计和特性无疑是优秀的，能满足MVVM的方式，但是还是过于复杂，且还有一些坑没填完（TODO），最终还是舍弃了。然后我在想，既然了解Qt和虚幻的窗口设计，又熟悉web这一套界面实现方式，为什么不自己造个轮子呢，如此一来，上述所有问题应该能徒手解决吧。

但这个轮子也不是凭空造的（全部凭空造的那是轮子哥），还是必须借用一些三方库来实现这些，初步选型：

1. 脚本使用javascript，选择v8自然是没什么可说的了
2. 后端2D绘制选用skia（chromium和火狐在使用，有部分使用经验，绘制性能和效果都不错）
3. 前端vm库使用vuejs
4. dom解析使用pugixml
5. 窗口系统和前端绑定自己实现，窗口体系参考html5标准，样式使用css-layout库

这样我们就有了一个最小的webkit实现，同时拥有扩展本地控件的能力（比如地图这类复杂控件）。

[nova.js](http://novajs.com/index.html)给出的一个web component的例子

控件代码
``` html
<!-- 注册自定义元素<markdown-editor> -->
<template is="dom-module">
    <style>
        :host {display:block;padding:15px 20px;border:1px solid rgba(16,16,16,0.1);background:white;}
        textarea { width:100%; height:100px; resize:none;padding:5px;}
    </style>
    <template>
        <h3>Markdown Editor</h3>
        <textarea value="{{content::input}}"></textarea>
        <p></p>
    </template>
    <script require-src="components/nova-markdown/marked"></script>
    <script>
        Nova({
            is: 'markdown-editor',
            props: {
                content: {
                    type: String,
                    value: 'Hello Nova! '
                }
            },
            createdHandler: function() {
                this.on('_contentChanged', this.contentObserver);
                this.render();
            },
            contentObserver: function(ev, oldVal, newVal) {
                this.render();
            },
            render: function() {
                this.querySelector('p').innerHTML = marked(this.content);
            }
        });
    </script>
</template>
```
使用Markdown控件
``` html
<markdown-editor content="# Hello \n Type some markdown here."></markdown-editor>
```

vue-loader的一个例子

![vue component template](-/images/vue-component-with-pre-processors.png)

其实Qt Designer的思路和上面有点类似，.ui文件有点像web component的声明，在编译期通过uic编译器编译为本地代码，在ui开发体验和性能上都照顾到了，但是有几个问题

1. 忽略了数据的绑定过程，一旦生成界面文件(ui_xx.h，相当于view部分)，剩下的数据逻辑和显示逻辑几乎是混在在一块的，整个数据流在开发中没有很好的体现出来，当然Qt也提供Model模型，用于和View型的控件进行数据绑定，解耦显示和模型，实际用起来还是糅杂得比较紧密
2. 控件的复用度是通过提升来实现的，也就是首先在Designer中有对应的控件基类，然后提升到某个用户自定义控件类，所见即所得的ui设计方式有助于减少上手的简易性，却无法降低程序逻辑组织的复杂度
3. 显示样式没有和控件本身没有进行分离，导致部分样式设置放置在.ui文件中，部分放置在控件代码内（.ui文件的样式配置最终也是以代码的方式体现），显示配置没有很好地分离出来，导致修改主题要么重写style，要么使用支持有限的stylesheet，两者结合起来使用往往会出现和预期不一致的效果（这个看经验了）

前面还没有提及对数据变化的管理，从目前前端去世来看，单一状态组件确实是比较好的思路，对于撤销重做这样的特性简直信手拈来 ，对于组件来说，有状态传入和传出，并不涉及对状态的直接修改，对于传出状态的回写需要在于状态管理方处理（store）。这点我后端的框架是能够非常好地支持，基本上前端的一些解决思路在现有的后端框架中都有所体现，所以前端这么玩也没有问题，部分对性能要求极高的部分可能还会放在后端来实现。

前端主要负责显示逻辑，数据逻辑还是来自后端，前端可以通过js绑定的获取数据，后端会建立数据对象监视器，对新数据进行检测，初始化时前段根据数据类型编写数据模板填充代码，进行首次渲染，后端出现数据变化，会触发前段数据变化通知回调，前段再根据变化编写对象的显示变化代码。

使用web作为前段主要是为了调试和编写方便，应用发布的理想方式是在正式发布前将前段代码转换为c++代码（根据js的ast树进行代码转换），这部分工作依赖性能测试的结果，如果性能在可接受的范围内，可以忽略代码转换，如果需要将性能提升到机制，那么在代码转换这块必须作出投入来减少开发便利带来的性能损失。


