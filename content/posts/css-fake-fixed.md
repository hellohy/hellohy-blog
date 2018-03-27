---
title: 用css模拟有滚动条的容器内部元素fixed效果
date: 2017-08-14 21:05:16
tags: ["css", "fixed"]
---

通常，大家都知道fixed属性是相对于屏幕视口（viewport)的位置来指定元素位置，元素的位置在屏幕滚动时不会改变。

## 需求
在实际项目中，遇到一个问题，如下，有个自适应高度的区域div.container，且该区域在屏幕的位置不固定，它的内部右下角有个固定的按钮Button，当container滚动时，button要固定不动。

很显然，看到这个需求，大家的第一反应就是用position: fixed，可是fixed是相对于整个viewport的位置，如果container位置是固定的，那么我们只要计算好Button相对于viewport的位置就可以了。但是这样不太友好，实际上Button是相对于container的，如果后期container有改动，Button的位置就要重新计算。而且，container的位置可能在布局用中会变化。为了灵活应对需求的变化，我们需要模拟Button相对于container的fixed效果

## 解决方案

### 1.js
（不推荐）js，无需多言，对Button设置position: absolute，right和bottom的位置由js计算

### 2.css
（推荐) 换个思路，让Button相对于一个窗口大小不会改变的容器absolute定位即可，那么，只需要在外面加层dom就可以了.这样子，l-container固定，position设置relative; container不变，尺寸与l-container一样，可以滚动; Button设置position: absolute。
详见demo: https://codepen.io/hellohy/pen/QMOyWR

```html
<div class="l-container">
  <div class="container">
    containercontainercontainercontainercontainercontainercontainercon
    <button>Hello!</button>
  </div>
</div>
```

```css
.l-container {
	position: relative;
	border: 1px solid yellow;
	width: 200px;
}
.container {
	border: 1px solid red;
	width: 200px;
	height: 200px;
	overflow: auto;
}
button{
	border:1px solid #ccc;
	cursor:pointer;
    display:block;
    margin:auto;
    position:absolute;
    bottom:20px;
	right: 10px;
}
```