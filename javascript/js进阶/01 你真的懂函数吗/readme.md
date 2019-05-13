## 你真的懂函数吗？
> 函数是`javascript`中的基本组件之一。一个函数是`javascript`过程—一组任务或计算值的语句。要使用一个函数，你必须将其定义在你希望调用它的作用域内。

### 函数声明
首先我们先从函数的声明方式说起。

函数的常用声明方式大概有如下几种：
* 匿名函数: 函数名称被省略的函数
* 具名函数: 有名字的函数
* 箭头函数: 定义时使用箭头来连接，有着更短的语法

一个匿名函数大概是下面这个样子，这样的函数也叫做函数表达式：
```js
var myFunction = function() {
  // statement
}
```
上面是一个有名字的匿名函数，只不过函数名被省略，默认函数名为表达式名。而一般没有名字的匿名函数会这样使用：  
```js
// 事件绑定
document.addEventListener('click',function() {
  // some code
})

// 自执行匿名函数
(function() {
  // some code
})()

```

接下来我们认识一下具名函数：  
```js
function fn () {
  // some code
}

// 要注意这里fn2的作用域只是作用于函数内部
const fn1 = function fn2 () {
  // some code
}
console.log(fn,fn2) // error: Uncaught: fn2 is not defined
```

最后我们介绍一下箭头函数。这是一个`es6`新提出的语法，相较于普通函数，箭头函数中的`this`指向最近父级作用域内的`this`。常用的写法如下：  
```js
// 只有一个参数可以省略参数的括号
const fn = i => i+1

const fn1 = (i,j) => i+j
```

