---
title: 如何在react中封装echarts
author: 黄云
date: 2017-12-18 20:37:18
categories:
- 经验总结
tags:
- css
- react
- echarts
---

[echarts](http://echarts.baidu.com/index.html)一款非常不错的可视化图表库，本文以echarts3.x版本为例，通过分析[echarts-for-react](https://github.com/hustcc/echarts-for-react)源码介绍如何在react中封装使用echarts

## 目录
1. [为什么要封装](#为什么要封装)
2. [如何实现](#如何实现)
3. [总结](#总结)
- - -

## 为什么要封装

### 旧的思维
习惯了在jquery开发模式下导入echarts、swiper、d3等需要操作到真实dom的第三方库，突然切换到React，会按照旧的思维模式使用，导致一些问题产生。先看看不封装的情况下在React中使用echarts:
```javascript
import React, { Component } from 'react'
import echarts from 'echarts'
export default class OldChart extends Component {
  componentDidMount () {
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('old-chart'))
    // 绘制图表
    myChart.setOption({
      title: { text: 'ECharts 入门示例' },
      tooltip: {},
      xAxis: {
        data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
      },
      yAxis: {},
      series: [{
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
      }]
    })
  }
  render () {
    return (
      <div id='old-chart' style={{ width: 400, height: 400 }} />
    )
  }
}
```
结果截图如下：
![](https://i.imgur.com/vK9T4yg.png)

这种方式先通过document.getElementById()的形式先获取到真实的dom，然后通过echarts.init()方法实例化。

例子中给出的是静态数据，如果数据是动态获取的，那就要把渲染echarts的数据放入state中，然后render()中通过echarts.setOption(this.state.option)方法来更新图表

### 存在的问题
1. echarts初始化需要访问DOM，为了整合到React中，我们需要在componentDidMount获取准备好的DOM，然后传递给echarts，再通过echarts的api来生成图，可以看出，实例化的方式有些繁琐
2. 代码完全不能复用，每实例化一个图表，我们都需要做一些重复的事情：在componentDidMount上获取准备好的dom，传递给echarts
3. 拓展性差，如果要监听windows.resize事件或者主题变更，我们可能需要一个个类操作过去进行变更

### 需要达到的目的
1. 简单好用，调用起来越简单越好
2. 复用性强，拥有基本的能力：比如监听resize
3. 封装后也能轻易调用[echarts API]( http://echarts.baidu.com/api.html#echartsInstance)

- - -

## 如何实现

### 第一步、最简单的功能

```javascript
/* test.js */
import React, {Component} from 'react'
import PropTypes from 'prop-types'
import echarts from 'echarts'
export default class ReactEcharts extends Component {
  constructor (props) {
    super(props)
    this.echartsInstance = echarts // echarts object 
    this.echartsElement = null // echarts dom
  }
  componentDidMount () {
    // 获取dom容器上的实例，如果没有，则实例化
    const echartObj = this.echartsInstance.getInstanceByDom(this.echartsElement) || this.echartsInstance.init(this.echartsElement)
    // echarts的万能接口，设置图表实例的配置项以及数据，所有参数和数据的修改都通过setOption完成
    echartObj.setOption(this.props.option)
  }
  render() {
    const style = this.props.style || {
      height: '300px'
    }
    // for render
    return (
      <div
        ref={(e) => { this.echartsElement = e }}
        style={style}
        className={this.props.className}
      />
    )
  }
}

ReactEcharts.propTypes = {
  option: PropTypes.object.isRequired, // eslint-disable-line react/forbid-prop-types
  style: PropTypes.object, // eslint-disable-line react/forbid-prop-types
  className: PropTypes.string
}

ReactEcharts.defaultProps = {
  style: {height: '300px'},
  className: ''
}

```

这里我们做了这几件事：
1. 创建一个名为`ReactEcharts`的react组件，包含两个属性`echartsInstance`、`echartsElement`
> `echartsInstance`属性建立与echarts的连接，能通过该属性调用echarts的api
> `echartsElement`获取图表所在的dom，我们给一个初始值null
2. 在render()中，通过ref设置一个回调函数，来获取dom元素，并将值赋予echartsElement属性
> 挂到react组件上的ref表示对组件实例的引用，而挂载到dom元素上时表示具体的dom元素节点。
3. 在componentDidMount()中实例化echarts，判断当前dom元素是否已经有echarts实例，如果有则用已有的，没有则init，然后调用echarts的setOption()来渲染图，参数我们通过this.props.option获取

ok, 搞定，理论上我们只需要调用这个组件时传递一个option就可以在页面上看到效果了，我们试一下：
```javascript
import ReactEcharts from './test'
const option = {
  title: { text: 'ECharts 入门示例' },
  tooltip: {},
  xAxis: {
    data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
  },
  yAxis: {},
  series: [{
    name: '销量',
    type: 'bar',
    data: [5, 20, 36, 10, 10, 20]
  }]
}
export default class testChart2 extends Component {
  render () {
    return (
      <ReactEcharts
        option={option}
        style={{ width: 400, height: 400 }} />
    )
  }
}
```

运行起来没有问题，在这里我们只要修改option的配置，就可以轻松更新图表了，option是动态的话也是没有问题的，只要你能将它的更新传递到`ReactEcharts`中即可

### 第二步、为组件增加一些常用的功能

接下来，我们加快进度，一次性为`ReactEcharts`增加一系列功能：
1. theme主题定制
2. 完善echarts.setOption()方法能力，增加`notMerge`, `lazyUpdate`参数，[官方文档链接](http://echarts.baidu.com/api.html#echartsInstance.setOption)
> `notMerge`可选，是否不跟之前设置的option进行合并，默认为false，即合并。

> `lazyUpdate`可选，在设置完option后是否不立即更新图表，默认为false，即立即更新。
3. 组件unmount时销毁相关echarts实例
4. 实现加载动画效果，当数据量大或者异步的时候，显示Loading

其中，这四点实现起来比较简单，参照echarts文档，在我们的componentDidMount()中作一些处理即可，这里就不详细讲了
我们直接上代码，大家看看就明白了：
```javascript
export default class ReactEcharts extends Component {
  constructor (props) {
    super(props)
    this.echartsInstance = echarts // echarts object 
    this.echartsElement = null // echarts dom
  }

  // first add
  componentDidMount () {
    const echartObj = this.renderEchartDom()
  }

  // update
  componentDidUpdate () {
    this.renderEchartDom()
  }

  // 3. 组件unmount时销毁相关echarts实例
  componentWillUnmount () {
    if (this.echartsElement) {
      this.echartsInstance.dispose(this.echartsElement)
    }
  }
  // 1. theme主题定制
  getEchartsInstance = () => this.echartsInstance.getInstanceByDom(this.echartsElement) ||
    this.echartsInstance.init(this.echartsElement, this.props.theme);

  // render the dom
  renderEchartDom = () => {
    // init the echart object
    const echartObj = this.getEchartsInstance()
    // 2. 完善echarts.setOption()方法能力，增加`notMerge`, `lazyUpdate`参数
    echartObj.setOption(this.props.option, this.props.notMerge || false, this.props.lazyUpdate || false)
    // 4. 实现加载动画效果，当数据量大或者异步的时候，显示Loading
    if (this.props.showLoading) echartObj.showLoading(this.props.loadingOption || null)
    else echartObj.hideLoading()

    return echartObj
  }

  render () {
    // 和上面一样，这里就不重复了...
    }
}
ReactEcharts.propTypes = {
  option: PropTypes.object.isRequired, // eslint-disable-line react/forbid-prop-types
  echarts: PropTypes.object.isRequired, // eslint-disable-line react/forbid-prop-types
  notMerge: PropTypes.bool,
  lazyUpdate: PropTypes.bool,
  style: PropTypes.object, // eslint-disable-line react/forbid-prop-types
  className: PropTypes.string,
  theme: PropTypes.string,
  showLoading: PropTypes.bool,
  loadingOption: PropTypes.object, // eslint-disable-line react/forbid-prop-types
}

ReactEcharts.defaultProps = {
  echarts: {},
  notMerge: false,
  lazyUpdate: false,
  style: {height: '300px'},
  className: '',
  theme: null,
  showLoading: false,
  loadingOption: null,
}
```

### 第三步、为组件封装事件
到目前为止，我们已经封装了一个挺好用的“静态”图表了，这里的静态指的是图表完全不能交互，dom尺寸变动图表也不会更新，显然这不太友好。

所以我们需要实现以下两个功能
1. echarts的事件处理
2. 监听窗口变动事件，使得dom尺寸发生变化时，图表能够更新

直观的，我们只要对外暴露一个echarts实例的接口，让外部自己通过该接口去绑定事件就可以。

但是这样对于外部调用者来说就比较繁琐，从使用者的角度，希望能够只要传入一事件对象，就能帮忙监听所有事件

为了达到这个目的，我们可以这样做：

#### 第一个功能、echarts的事件处理
1. 先定义一个事件对象，数据结构如下:
```javascript
{
  'eventName': callback, // callback是回调函数
  // 例如：
  click: function () {...},
  legendselectchanged:  function () {...},
  ...
}
```
其中属性值对应echarts支持的事件名[echarts event api](http://echarts.baidu.com/api.html#events)

2. 假设`ReactEcharts`组件接受一个props.onEvents的事件对象，我们为`ReactEcharts`组件增加一个方法`bindEvent()`
```javascript
componentDidMount () {
  const echartObj = this.renderEchartDom()
  const onEvents = this.props.onEvents || {}
  this.bindEvents(echartObj, onEvents)
}

bindEvents = (instance, events) => {
  const _loopEvent = (eventName) => {
    if (typeof eventName === 'string' && typeof events[eventName] === 'function') {
      instance.off(eventName)
      instance.on(eventName, (param) => {
        // 触发回调
        events[eventName](param, instance)
      })
    }
  }

  for (const eventName in events) {
    if (Object.prototype.hasOwnProperty.call(events, eventName)) {
      _loopEvent(eventName)
    }
  }
}
```
代码理解起来不难，遍历`onEvents`对象，并通过echarts.on()接口将`eventName`和回调函数callback绑定，除了保留echarts自身事件的事件参数param，我们还加上一个`instance`参数，提供更强的能力

3. 接下来看一下外部如何调用：
```javascript
export default class YourComponentName extends Component {
  componentDidMount () { /** ... **/ }
  getOption () { /** ... **/ }
  onChartClick (param, echart) {
    console.log('onChartClick', param, echart)
  }
  onChartLegendSelectChanged (param, echart) {
    console.log('onLegendSelect', param, echart)
  }
  render () {
    const onEvents = {
      click: this.onChartClick,
      legendselectchanged: this.onChartLegendSelectChanged
    }
    return (
      <div>
        <EchartsReact
          option={this.getOption()}
          style={{height: '800px', width: '100%'}}
          onEvents={onEvents}
          className='react-echarts'
        />
      </div>
    )
  }
}
```

这样我们就完成了事件处理

#### 第二个功能、resize事件监听
接下来再处理一下resize，当外部容器尺寸变化或者用户窗口变化的时候图表能够自动更新，如何监听dom尺寸的变化，我们在本文就不详细讲解了，这里我们直接调用一个npm包[element-resize-event](https://www.npmjs.com/package/element-resize-event)来帮助我们监听dom尺寸变化

引入这个包，将`ReactEcharts`组件的componentDidMount()修改如下：
```javascript
import elementResizeEvent from 'element-resize-event'
// ...
  componentDidMount () {
    const echartObj = this.renderEchartDom()
    const onEvents = this.props.onEvents || {}
    this.bindEvents(echartObj, onEvents)

    // on resize
    elementResizeEvent(this.echartsElement, () => {
      echartObj.resize()
    })
  }
  
  componentWillUnmount () {
    if (this.echartsElement) {
      // if elementResizeEvent.unbind exist, just do it.
      if (typeof elementResizeEvent.unbind === 'function') {
        elementResizeEvent.unbind(this.echartsElement)
      }
      this.echartsInstance.dispose(this.echartsElement)
    }
  }
// ...
```
> 同时别忘了在unmount时解绑事件

好了，到目前为止，我们完成了大部分工作，我们的`ReactEcharts`组件能够应付大部分的应用场景，但是还有一些细节没有处理（诸如chartReady，onResizeCallback）
> chartReady图表实例化之前触发

> onResizeCallback窗口变动后触发，可用于图表内部坐标点的重新计算

本文就不详细讲了，最新代码可以到[这里](http://git.sdp.nd/112356/ae-example-echarts/tree/master/src/components/echarts)查看

- - -

## 总结
1. 当使用React遇到需要使用第三方库的时候，可以根据第三方库的api以及从使用者的角度来进行一些简单的封装，使得这些第三方库结合React使用起来更加方便
2. 封装的时候需要预见一些可能需要实现的功能，即时现在不用，将来也需要扩展