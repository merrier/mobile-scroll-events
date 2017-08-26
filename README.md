# 移动端滚动事件大起底

最近在做移动端的项目，其中有个需求就是滚动监听标签页，实现用户滚动到不同位置时点亮相应tab按钮；这其实是个很简单的需求，但是根据之前的项目经验，移动端的滚动事件会有各种坑，所以就花时间做了一些功课，对移动端滚动事件中的坑进行了总结，同时提供了一些解决方案。

## 移动端滚动事件介绍

我们这里要讲的是`onscroll`事件，具体参见[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/GlobalEventHandlers/onscroll)

## 滚动事件应用

我做了几个简单的demo，可以直接点击下面的链接或者扫描二维码查看（在手机上效果更佳~）:

| 下拉刷新 | 吸顶效果 | 图片懒加载 | 侧边浮动导航栏 |
|:------:|:---------:|:------------:|:-------------:|
| [在线实例](https://merrier.github.io/mobile-scroll-events/drop-and-refresh.html) | [在线实例](https://merrier.github.io/mobile-scroll-events/ceiling.html) | [在线实例](https://merrier.github.io/mobile-scroll-events/lazy-load.html) | [在线实例](https://merrier.github.io/mobile-scroll-events/side-nav-bar.html) |
| ![下拉刷新](./qrcode/drop-and-refresh.png) | ![吸顶效果](./qrcode/ceiling.png) | ![图片懒加载](./qrcode/lazy-load.png) | ![侧边浮动导航栏](./qrcode/side-nav-bar.png) |

## 滚动事件分类和兼容性

按照实际滚动的dom类型分为全局滚动和局部滚动

### 全局滚动

滚动条在body节点或者更顶层，一般是这样调用：

```js
window.onscroll = function() {
    var bHeight = document.body.clientHeight,  // body对象高度，如果有滚动高度也包括
        wHeight = window.innerHeight,  // 浏览器窗口的视口
        sTop = document.body.scrollTop;  // body距离滚动顶部的距离
    var isScrollBottom = bHeight - (wHeight + sTop) === 0 ? true : false;
    if (isScrollBottom) {
    // 执行相关代码
    }
}
```
也可以通过addEventListener的方式

### 局部滚动

滚动条在body下某一个dom节点，在移动端如果使用局部滚动，意思就是我们的滚动在一个固定宽高的div内触发，将该div设置成`overflow:scroll/auto;`来形成div内部的滚动，这时我们监听div的`onscroll`发现触发的时机；调用方式和全局滚动类似：

```js
（document.getElementById('div1').onscroll = function(){}）
```

### 兼容性

整体来看，<b>全局滚动的兼容性要不如局部滚动，安卓比IOS兼容性更好</b>：

|                  |   body滚动  |   局部滚动  |
|:----------------:|:----------:|:----------:|
| ios              | 不能实时触发 | 不能实时触发 |
| android          |   实时触发  |   实时触发   |
| ios wkwebview内核 |   实时触发  |   实时触发   |

为什么IOS下的滚动事件会有各种bug呢，通过查阅资料，得出如下结论：

ios的webview内核设定了其在进行momentum scrolling(弹性滚动， 设置`-webkit-overflow-scrolling:touch`可以达到弹性滚动效果，安卓无效)时，会<b>停止所有的事件响应及DOM操作引起的页面渲染</b>，故`onscroll`不能实时响应，具体可以[点击这里](https://www.tjvantoll.com/2012/08/19/onscroll-event-issues-on-mobile-browsers/)查看某位大牛写的实例

> 这里说明一下关于ios的wkwebview内核是ios从ios8开始提供的新型webview内核，和之前的uiwebview相比，性能要好，具体大家可以自行查看关于wkwebview的相关概念

### 解决方案

既然onscroll事件在ios和安装上的表现并不统一，同时根据浏览器内核的不同会有潜在的bug出现，就出现了针对于移动端滚动事件的各种兼容方案，总结如下：

* 使用 `ontouchmove` 去替代 onscroll，虽然能更频繁的触发事件，但是这边的项目需求是实时响应滚动事件的同时，还要对页面元素进行重定位的DOM操作，由上述原因可知，在滚动过程中，页面会停止一切关于DOM方面的操作，所以若使用 ontouchmove 去实现的话，在按住屏幕进行滑动的时候，屏幕会出现元素抖动的情况(事件触发与DOM操作间具有几十毫秒的时间差)，兼容失败
* 使用 `iscroll` 的probe版本，该版本能实时探查到滚动的距离，但该钩子函数是实时去关注 requestAnimationFrame 下的状态，所以对浏览器的版本性能消耗很大，安卓机根本动不了，兼容失败
* 使用 `swiper` 插件，在启动 freeMode 模式时模拟原生的弹性滚动( swiper 模拟原生滚动的方案能兼容较多的安卓机型不出现bug，推荐), 因为 swiper 没有实时监听滚动位置的功能,故我监听滚动开始及结束后的事件，通过 setInterval 及一些计算去实现滚动条的监听，但因为 react 元素的变化量比较大，导致 swiper 在移动端时对父容器的计算速率达到了一个瓶颈，依旧出现很卡顿的现象，兼容失败

通过以上的兼容性尝试，可以发现<b>其实并没有一个完美的解决方案</b>，所以如果真的需要达到某些移动端滚动效果的话，可以采取fallback方案：

* Android用scroll方案，因为兼容性很客观；
* IOS如果不需要兼容8以下版本的话，就也直接用scroll方案，因为`wkwebview`已经对滚动事件进行了优化，而如果需要兼容8以下版本的话，可以考虑isroll或JRroll这两种插件，同时需要真机测试查看效果是否达到要求（尤其是QQ浏览器和搜狗浏览器）。

由于查询到的资料比较老旧，对于滚动事件的兼容性描述可能已经过时了，我就在最近（2017-8-24）用各种浏览器测试了一下ios中的滚动事件的兼容性，总结如下：

|                 | 是否可以弹性滚动 | 是否需要设置overflow:scrolling才能弹性滚动 | 设置overflow:scrolling之后，滚动期间是否监听事件 | 未设置overflow:scrolling，滚动期间是否监听事件 |
|:---------------:|:-------------:|:-------------------------------------:|:------------------------------------------:|:----------------------------------------:|
|  safari(v10.0)  |       否      |                   -                   |                      是                     |                     是                    |
|  chrome(v60.0)  |       是      |                   否                   |                     是                     |                     是                    |
|  firefox(v8.2)  |       否      |                   -                   |                     是                     |                      是                    |
| weixin(v6.5.14) |       是      |                   否                   |                     是                     |                     是                    |
|   QQ(v7.7.2)    |       是      |                   否                   |                     是                     |                     是                    |
|   搜狗(v5.8.1)   |       是      |                   否                   |                     否                     |                     否                    |

从上面的图片可以看到，最新版的ios浏览器其实并不需要`overflow:scrolling`就可以实现弹性滚动，同时除了搜狗浏览器之外，其他浏览器在滚动期间都会监听事件，由此可见截止到目前（2017-8-24），ios和浏览器对滚动事件的兼容性已经做了很多优化和改进了，之后有时间的话再在android手机上做一下测试……

## 滚动事件性能优化

除了兼容性问题以外，由于滚动事件和resize事件同属于会频繁触发的事件。如果事件中涉及到大量的位置计算、DOM 操作、元素重绘等工作且这些工作无法在下一个 scroll 事件触发前完成，就会造成浏览器掉帧。

### 防抖和节流

scroll 事件本身会触发页面的重新渲染，同时 scroll 事件的 handler 又会被高频度的触发, 因此事件的 handler 内部不应该有复杂操作，例如 DOM 操作就不应该放在事件处理中。

针对此类高频度触发事件问题（例如页面 scroll ，屏幕 resize，监听用户输入等），下面介绍两种常用的解决方法，防抖和节流（underscore和lodash里面有封装好的这两种方法，感兴趣的话可以研究一下源码）。

#### 防抖

防抖技术即是可以把多个顺序地调用合并成一次，也就是在一定时间内，规定事件被触发的次数。

#### 节流

防抖函数确实不错，但是也存在问题，譬如图片的懒加载，我希望在下滑过程中图片不断的被加载出来，而不是只有当我停止下滑时候，图片才被加载出来。又或者下滑时候的数据的 ajax 请求加载也是同理。

这个时候，我们希望即使页面在不断被滚动，但是滚动 handler 也可以以一定的频率被触发（譬如 250ms 触发一次），这类场景，就要用到另一种技巧，称为节流函数（throttling）。

节流函数，只允许一个函数在 X 毫秒内执行一次。

与防抖相比，节流函数最主要的不同在于它保证在 X 毫秒内至少执行一次我们希望触发的事件 handler。

### 使用rAF（requestAnimationFrame）触发滚动事件

上面介绍的抖动与节流实现的方式都是借助了定时器 setTimeout ，但是如果页面只需要兼容高版本浏览器或应用在移动端，又或者页面需要追求高精度的效果，那么可以使用浏览器的原生方法 rAF（requestAnimationFrame）。

window.requestAnimationFrame() 这个方法是用来在页面重绘之前，通知浏览器调用一个指定的函数。这个方法接受一个函数为参，该函数会在重绘前调用。

rAF 常用于 web 动画的制作，用于准确控制页面的帧刷新渲染，让动画效果更加流畅，当然它的作用不仅仅局限于动画制作，我们可以利用它的特性将它视为一个定时器。（当然它不是定时器）

通常来说，<b>rAF 被调用的频率是每秒 60 次，也就是 1000/60 ，触发频率大概是 16.7ms 。</b>（当执行复杂操作时，当它发现无法维持 60fps 的频率时，它会把频率降低到 30fps 来保持帧数的稳定。）

### 总结一下

* 防抖动：防抖技术即是可以把多个顺序地调用合并成一次，也就是在一定时间内，规定事件被触发的次数。
* 节流函数：只允许一个函数在 X 毫秒内执行一次，只有当上一次函数执行后过了你规定的时间间隔，才能进行下一次该函数的调用。
* rAF：16.7ms 触发一次 handler，降低了可控性，但是提升了性能和精确度。

> 从本质上而言，我们应该尽量去精简 scroll 事件的 handler ，将一些变量的初始化、不依赖于滚动位置变化的计算等都应当在 scroll 事件外提前就绪。建议：避免在scroll 事件中修改样式属性 / 将样式操作从 scroll 事件中剥离

#### 参考文章

* [吸顶效果解决方案](http://www.ayqy.net/blog/%E5%90%B8%E9%A1%B6%E6%95%88%E6%9E%9C%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)
* [onscroll Event Issues on Mobile Browsers](https://www.tjvantoll.com/2012/08/19/onscroll-event-issues-on-mobile-browsers/)
* [前端: 移动端onscroll事件在部分浏览器内不能实时触发](https://segmentfault.com/q/1010000004453730)
* [移动web之滚动篇](http://www.alloyteam.com/2017/04/secrets-of-mobile-web-scroll-bars-and-drop-refresh/)
* [高性能滚动 scroll 及页面渲染优化](http://web.jobbole.com/86158/)