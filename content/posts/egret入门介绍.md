---
title: egret入门介绍
date: 2017-11-17 11:19:21
tags: [egret]
---

介绍一下egret的容器概念、资源管理、纹理与位图、事件以及tween的使用

# Egret入门指南
简单介绍下egret~

## 目录
1. [容器](#容器)
2. [资源管理](#资源管理)
1. [纹理和位图](#纹理与位图)
3. [事件](#事件)
4. [Tween的使用](Tween的使用)

## 容器
```
DisplayObject
    |--- DisplayObjectContainer
        |--- Sprite
        |--- Stage 舞台
    |--- Shape 矢量绘图
    |--- Bitmap 位图
    |--- TextField 文字
```


```
//stage是所有显示对象的'根'，stage下面是一个树状的显示列表
//显示对象容器，使用DisplayObjectContainer
var container = new egret.DisplayObjectContainer();
//容器的缩放，旋转，位移将影响到它下面的子节点(即包含的显示对象)
container.scaleX = 0.2;
container.scaleY = 0.2;
//用addChild方法把一个显示对象添加到显示列表
this.addChild(container);
//位图是显示对象，纹理不是
var bitmap1 = new egret.Bitmap(RES.getRes("egretIcon"));
container.addChild(bitmap1);
//显示对象的位置和尺寸，相对于stage来说，也受到父容器的影响，也就是说每个显示对象容器，拥有自己的坐标系
bitmap1.x = bitmap1.y = 50;
```

注意这几个特性：
```
DisplayObjectContainer.touchChildren
```
> 这个属性在实施性能优化的时候应该很有用。

```
DisplayObjectContainer.removeChildren
```
> 移除所有显示对象，一个方便使用的快捷方法

其实Egret也提供了Sprite类，用法：

```
var sprite:egret.Sprite = new egret.Sprite();//Sprite与DisplayObjectContainer基本类似
sprite.graphics.beginFill(0xFF0000,1);//区别是Sprite拥有graphics对象可用于画图
sprite.graphics.drawRect(0,0,100,100);
sprite.graphics.endFill();
sprite.y = 300;
this.addChild(sprite);
```

如何选择：如果容器不需要画图功能，我们可以直接用DisplayObjectContainer；需要画图的时候就用Sprite。

- - -


## 资源管理
### 资源文件
首先打开项目目录下的resource/default.res.json文件，这个您可以认为就是资源的配置文件，里面定义了resource目录下资源的名称和对应的url，甚至可以把资源划分成若干个group，这样来实现分批加载。

```
{
"resources":
    [
        {"name":"bgImage","type":"image","url":"assets/bg.jpg"},
        {"name":"egretIcon","type":"image","url":"assets/egret_icon.png"},
    ],

"groups":
    [
        {"name":"preload","keys":"bgImage,egretIcon"},
    ]
}
```

> 您可以手工编辑这个配置文件，也可以使用egret推出的ResDepot，这是一个强大的专门管理resource.json的工具，而且是免费的哦，有了它，您就可以在可视化界面中修改resource.json了。

### 加载实现
RES.ResourceLoader(注意ResourceLoader的命名空间是RES，而不是egret)
```
//使用RES模块，侦听GROUP_COMPLETE事件和GROUP_PROGRESS事件，可以同步显示加载进度，并继续执行加载完成后的逻辑
RES.addEventListener(RES.ResourceEvent.GROUP_COMPLETE,this.onResourceLoadComplete,this);
RES.addEventListener(RES.ResourceEvent.GROUP_PROGRESS,this.onResourceProgress,this);
RES.loadConfig("resource/default.res.json","resource/");//加载资源配置文件
RES.loadGroup("preload");//加载某个资源group
```

> loadConfig的第二个参数用于指定资源根目录。
> 真实项目中可能有很多资源，我们可以在游戏启动之前先做加载，加载完毕后再进行游戏的初始化。


- - -

## 纹理与位图

在onResourceLoadComplete方法里，我们完成位图的创建和显示。代码如下：

```
var stage = egret.MainContext.instance.stage;//获取Stage引用
this.logo = new egret.Bitmap();//创建位图
this.logo.texture = RES.getRes("egretIcon");//设置纹理
stage.addChild(this.logo);//添加到显示列表
```
getRes 返回的数据并不是位图，而是内容数据，也就是位图纹理，对应 egret.Texture。需要设置位图的texture属性为 getRes 返回的数据，图片才能显示。

- - -

## 事件

1. 事件的基类是Event，所有的事件类从Event扩展而来, 来看一下Event的构造函数：
```
public constructor(type:string, bubbles:boolean = false, cancelable:boolean = false) {
    this._type = type;
    this._bubbles = bubbles;
    this._cancelable = cancelable;
}
```
> 这就说明Egret中的事件支持冒泡机制，您可以在创建事件的时候决定它是否冒泡，同样也就有了target和currentTarget之分。

## Tween的使用

用于实现缓动动画，除了实现复杂的动画队列之外，Tween可以用各种Ease函数，控制缓动的细节。
Egret库中默认集成了一些Ease实现，要使用它们，只需在参赛中传入即可，比如：

```
var tw = egret.Tween.get(this.logo,{loop:true});
tw.to({x:360,y:600},1000,egret.Ease.sineIn);
```
- - -