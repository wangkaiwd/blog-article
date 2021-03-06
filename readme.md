## 原生`js`实现一个轮播组件
最近自己使用`vue`实现了`carousel`轮播组件，整体上的思路和之前的原生`js`有很大的区别，对应的动画效果的实现是一个比较大的难点。这我们用原生`js`来实现轮播组件，方便我们对框架和原生实现方式有一个认知和对比，也正好复习一下原生和`DOM`相关的`API`。

在样式方面，大家可以发挥自己的想象力，也可以参考社区其它的类似组件，这里我实现了一个简易样式版本，供大家参考：  
* 预览地址：https://wangkaiwd.github.io/vue-example/src/views/vueTransition/slider1.html
* 源码地址：https://github.com/wangkaiwd/vue-example/blob/master/src/views/vueTransition/slider1.html

`vue`版本的在这里，并且支持移动端滑动切换： 
* 预览地址：https://wangkaiwd.github.io/smile-ui/views/carousel.html
* 源码地址：https://github.com/wangkaiwd/smile-ui/blob/master/src/components/smileCarousel/carousel.vue

在实现组件之前，我们需要先了解一下有逢轮播和无缝轮播。  

有缝轮播在轮播项正向移动到第一项或反向移动到最后一项时，会有一个回退效果。而无缝轮播会让人感觉不到这个切换效果。

接下来我们一步步通过原生`js`先实现一个有缝轮播，之后升级到无缝轮播，实现最终效果。
### 有缝轮播
有缝轮播相对来说在实现思路上会简单一些，主要是利用了`css3`提供的`transiton`、`translate`属性来实现切换的动画效果。

首先搭建初始的页面结构，效果大概是这样的：
![slider-init](./screenshots/slider-init.png)  
之后我们会在这个基础上让1,2,3动起来

现在假设我们是使用组件的用户，我们理想中的使用方式大概是这样的：  
```html
<div class="slider">
  <div class="slider-item">1</div>
  <div class="slider-item">2</div>
  <div class="slider-item">3</div>
</div>
```
```js
const slider = new Slider('.wk-slider',options);
```

为了页面的美观，我们添加一些初始化样式  
```css
/*初始化样式*/
* {
  margin: 0;
  padding: 0;
}
*,
*::after,
*::before {
  box-sizing: inherit;
}
/*
  通配符的css权重是最低的，html的标签都会继承box-sizing
  并且对应元素进行盒模型更改的时候，对应的子元素也会更改
*/
html {
  box-sizing: border-box;
}
```
用户自己的样式是这样的： 
```css
/*用户样式*/
.demo-wrapper {
  margin: 40px;
}
.slider {
  margin: 0 auto;
  border: 4px solid black;
  overflow: hidden;
}
.slider-item {
  width: 400px;
  height: 200px;
  background-color: pink;
  font-size: 80px;
  text-align: center;
  line-height: 200px;
  color: #fff;
}
```

接下来我们进行`js`逻辑的实现。  

由于组件在使用时第一个参数为`element`，第二个参数为`options`配置项，所以我们的构造函数需要这样写：
```js
class Slider {
  constructor (element, options) {
    this.slider = document.querySelector(element);
    this.options = options;
  }
}
```
之后我们要获取到轮播项的对应的宽度赋值到`slider`元素上，并为对应的元素添加特定前缀的`css`类名，防止样式冲突。而且由于要让整个子元素进行平移，还要将所有子元素放到一起，然后再进行平移，并通过`transition`设置过渡动效。

我们在构造函数的原型上添加对应的方法
```js
initSliderStyle () {
  this.items = [...this.slider.children];
  // 这里获取宽度时要小心异步加载
  this.itemWidth = this.items[0].offsetWidth;
  this.slider.classList.add('wk-slider');
  this.slider.style.width = `${this.itemWidth}px`;
  this.createItemsWrapper();
}

createItemsWrapper () {
  this.itemsWrapper = document.createElement('div');
  this.itemsWrapper.classList.add('wk-slider-items-wrapper');
  this.slider.appendChild(this.itemsWrapper);
  this.items.map(item => {
    this.itemsWrapper.appendChild(item);
    item.classList.add('wk-slider-item');
  });
}
```

对应的组件`css`： 
```css
/*组件样式*/
.wk-slider {
  display: flex;
}
.wk-slider-item {
  flex-shrink: 0;
}
.wk-slider-items-wrapper {
  display: flex;
  flex-shrink: 0;
  transition: all 1s;
}
```

最后我们为子元素容器设置定时器，让子元素动起来： 
```js
autoPlay () {
  if (!this.options.autoPlay) return;
  setInterval(() => {
    this.index++;
    this.go(this.index);
  }, 2000);
}

go (index) {
  const lastIndex = this.items.length - 1;
  if (index > lastIndex) {this.index = 0;}
  if (index < 0) {this.index = lastIndex;}
  this.itemsWrapper.style.transform = `translateX(${-this.itemWidth * this.index}px)`;
}
```
最终效果：  
![hasSpace](./screenshots/hasSpaceSliders.gif)

完整代码如下： 
```js
class Slider {
  constructor (element, options) {
    this.slider = document.querySelector(element);
    this.options = options;
    this.index = 0;
    this.initSliderStyle();
    this.autoPlay();
  }

  initSliderStyle () {
    this.items = [...this.slider.children];
    // 这里获取宽度时要小心异步加载
    this.itemWidth = this.items[0].offsetWidth;
    this.slider.classList.add('wk-slider');
    this.slider.style.width = `${this.itemWidth}px`;
    this.createItemsWrapper();
  }

  createItemsWrapper () {
    this.itemsWrapper = document.createElement('div');
    this.itemsWrapper.classList.add('wk-slider-items-wrapper');
    this.slider.appendChild(this.itemsWrapper);
    this.items.map(item => {
      this.itemsWrapper.appendChild(item);
      item.classList.add('wk-slider-item');
    });
  }

  autoPlay () {
    if (!this.options.autoPlay) return;
    setInterval(() => {
      this.index++;
      this.go(this.index);
    }, 2000);
  }

  go (index) {
    const lastIndex = this.items.length - 1;
    const { itemsWrapper, itemWidth } = this;
    if (this.index > lastIndex) { this.lastToFirst(); }
    if (this.index < 0) { this.firstToLast(lastIndex); }
    itemsWrapper.style.transform = `translateX(${-itemWidth * this.index}px)`;
  }
}

const slider = new Slider('.slider', { autoPlay: true });
```
```css
/*初始化样式*/
* {
  margin: 0;
  padding: 0;
}
*,
*::after,
*::before {
  box-sizing: inherit;
}
/*
  通配符的css权重是最低的，html的标签都会继承box-sizing
  并且对应元素进行盒模型更改的时候，对应的子元素也会更改
*/
html {
  box-sizing: border-box;
}

/*用户样式*/
.demo-wrapper {
  margin: 40px;
}
.slider {
  margin: 0 auto;
  border: 4px solid black;
  overflow: hidden;
}
.slider-item {
  width: 400px;
  height: 200px;
  background-color: pink;
  font-size: 80px;
  text-align: center;
  line-height: 200px;
  color: #fff;
}

/*组件样式*/
.wk-slider {
  display: flex;
}
.wk-slider-item {
  flex-shrink: 0;
}
.wk-slider-items-wrapper {
  display: flex;
  flex-shrink: 0;
  transition: all 1s;
}
```

这样一个简单的有缝轮播组件就初步完成了。但是这里我们忽略了一个问题：当我们的`slide-item`是图片的时候，由于图片的异步加载，会导致获取的宽度并不准确，我想到的解决方法是在`onload`事件之后再进行组件的使用：  
```js
window.onload = () => {
  const slider = new Slider('.slider', { autoPlay: true });
}
```
小伙伴也可以自己想一些其它的解决方法，展现奇思妙想的时候到了。
### 无缝轮播
有缝轮播其实是无缝轮播的一个升级版本，当图片轮播到最后一项的时候，可以继续像之前一样轮播到第一项，并不会有回退效果。

大概的一个思路如下：  
* 分别复制轮播的第一项和最后一项，然后再分别插入到最后一项和第一项
* 如果是正向轮播：轮播到最后一项时，继续轮播会进入复制的第一项，之后再闪动到真正的第一项，继续轮播
* 如果是逆向轮播：轮播到第一项时，会继续轮播到复制的最后一项，之后闪动到真正的最后一项，继续轮播

这里我们先复习几个原生`js`语法：
* `Node.cloneNode()`: 克隆一个节点，返回调用该方法的节点的一个副本。如果参数传入`true`，会克隆当前节点的所有后代节点。如果为`false`,则只克隆该节点本身。默认参数为`false`。
* `parentNode.insertBefore(newNode,referenceNode)`: 在参考节点之前插入一个拥有指定父节点的子节点。
  
接下来我们一步一步来操作。

首先我们分别复制第一项和最后一项插入到对应的位置：
```js
appendCloneNode() {
  const first = this.items[0];
  const last = this.items[this.items.length - 1];
  const firstClone = first.cloneNode(true);
  const lastClone = last.cloneNode(true);
  this.slider.insertBefore(lastClone, first);
  this.slider.appendChild(firstClone);
  this.items = [...this.slider.children];
}
```
页面布局如下： 
![sliderNoSpace](./screenshots/slider-noSpace.png)
  
之后，我们要分别对最后一项到第一项以及第一项到最后一项进行处理。
```js
// 最后一张到第一张
lastToFirst () {
  const { itemsWrapper, itemWidth } = this;
  // 将transition效果取消
  itemsWrapper.style.transition = 'none';
  // 将索引设置为1
  this.index = 1;
  // 直接移动到第二张(没有过渡效果)
  itemsWrapper.style.transform = `translateX(${-itemWidth * this.index}px)`;
  // 索引+1
  this.index++;
  itemsWrapper.offsetWidth;
  // 添加动画效果，通过go方法移动到第三章
  itemsWrapper.style.transition = 'all 1s';
}
// 第一张到最后一张（代码含义同上）
firstToLast (lastIndex) {
  const { itemsWrapper, itemWidth } = this;
  // 将transition效果取消
  itemsWrapper.style.transition = 'none';
  this.index = lastIndex - 1;
  itemsWrapper.style.transform = `translateX(${-itemWidth * this.index}px)`;
  this.index--;
  itemsWrapper.offsetWidth;
  itemsWrapper.style.transition = 'all 1s';
}
```
上边的代码是轮播的一个重要思路：**将最后一张/第一张“闪动”到自己复制出来的哪一项，之后恢复过渡，继续轮播**

而这里特别令人疑惑的一个地方：`itemsWrapper.offsetWidth`。整体上来看，这一行代码和整个的逻辑并没有任何关系，但是这里涉及到有关浏览器重排、重绘的一个重要知识点：  
> DOM变动和样式变动，都会触发浏览器的重新渲染。但是，浏览器比较智能，会尽量把所有的变动集中在一起，排成一个队列，然后一次性执行，尽量避免多次重新渲染。

接下来我们回到上边的代码：浏览器智能的将`itemsWrapper.style.transition="none"`和`itemsWrapper.style.transition = "all 1s"`进行合并后执行，而我们要做的是强制浏览器进行重新渲染，让这俩行代码分别执行，而`itemsWrapper.offsetWidth`就可以强制浏览器进行重新渲染。类似的属性和方法还有很多，以下是我收集的一些文章，希望对大家的知识理解有帮助：   
* [What forces layout / reflow](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)
* [Force browser to trigger reflow while changing CSS](https://stackoverflow.com/questions/21664940/force-browser-to-trigger-reflow-while-changing-css)
* [网页性能管理详解](https://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)

到这里，主要的逻辑功能已经基本上完成了。接下来的工作相对来说就会很简单了，我这里不再赘述，大概的工作是这样的：  
* 添加左右箭头
* 添加下侧点击切换的控制条
* 在鼠标移入轮播区域的时候停止轮播，鼠标离开轮播区域时继续轮播

之后我们可以通过传入的`options`来让用户自己控制是否支持自动轮播，是否需要左右箭头和下侧控制条，也可以让用户自定义轮播的时间间隔。



