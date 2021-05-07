### 1.js数据类型都有哪些
基本数据类型：Undefined, Null, Boolean, Number, String, Symbol(es6)

引用数据类型：Object, Array, Function

**主要区别**

- 声明变量时不同的内存分配：前者由于占据的空间大小固定且较小，会被存储在**栈**当中，也就是变量访问的位置；后者则存储在**堆**当中，变量访问的其实是一个指针，它指向存储对象的内存地址。
- 也正是因为内存分配不同，在复制变量时也不一样。前者复制后2个变量是独立的，因为是把值拷贝了一份；后者则是复制了一个指针，2个变量指向的值是该指针所指向的内容，一旦一方修改，另一方也会受到影响。

**注意**

这里有个规定，原始数据类型的值，也称原始值是无法通过param[key] = value这种形式修改的。即只有引用数据类型才可以以param[key] = value形式修改。
```javascript
let str = '123'
str[0] = 2
console.log(str) // 打印的还是123
```

### 2.typeof运算符和instanceof运算符以及isPrototypeOf()方法的区别
#### (1)typeof
语法：typeof a !== 'xxx'(返回字符串)

typeof能检测**几乎所有**的基本数据类型（typeof null 返回object），分别返回number, string, boolean, undefined, symbol等字符串

typeof检查所有函数返回字符串function

typeof检查除上述外的其他所有的类型(Null, Object, Array等)，都会返回字符串object。
#### (2)instanceof
语法：A instanceof B(返回boolean值)

instanceof运算符用于判断的是B.prototype是否存在A的原型链之中(即：A是不是B的实例？)
```javascript
// 定义构造函数
function C(){} 
function D(){} 

var o = new C();
o instanceof C; // true，因为 Object.getPrototypeOf(o) === C.prototype
o instanceof D; // false，因为 D.prototype 不在 o 的原型链上

o instanceof Object; // true，因为 Object.prototype.isPrototypeOf(o) 返回 true
C.prototype instanceof Object // true，同上

C.prototype = {};
var o2 = new C();

o2 instanceof C; // true

o instanceof C; // false，C.prototype 指向了一个空对象,这个空对象不在 o 的原型链上.

D.prototype = new C(); // 继承
var o3 = new D();
o3 instanceof D; // true
o3 instanceof C; // true 因为 C.prototype 现在在 o3 的原型链上

```
#### (3)isPrototypeOf
语法：  A.isPrototypeOf(B)

参数：  object  在该对象的原型链上搜寻

返回值：  Boolean，表示调用对象是否在另一个对象的原型链上。

isPrototypeOf判断的是A对象是否存在于B对象的原型链之中

### 3.请描述一下 cookies sessionStorage和localstorage区别
相同点：都存储在客户端

不同点：
#### （1）存储大小
- cookie数据大小不能超过4k。
- sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。
#### （2）有效时间
- localStorage       存储持久数据，浏览器关闭后数据不丢失除非主动删除数据；
- sessionStorage     数据在当前浏览器窗口关闭后自动删除。
- cookie             设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭

#### （3）数据与服务器之间的交互方式
- cookie的数据会自动的传递到服务器，服务器端也可以写cookie到客户端
- sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存。

### 4.call和apply简单介绍一下
#### apply：

接受两个参数，第一个参数是要绑定给this的值，第二个参数是一个参数数组。当第一个参数为null、undefined的时候，默认指向window。

#### call：

第一个参数是要绑定给this的值，后面传入的是一个参数列表。当第一个参数为null、undefined的时候，默认指向window。

#### apply和call:

事实上apply 和 call 的用法几乎相同。 唯一的差别在于：当函数需要传递多个变量时, apply 可以接受一个数组作为参数输入, call 则是接受一系列的单独变量。

### 5.js异步操作介绍一下

### 6.es6新特性

### 7.防抖和节流
#### （1）防抖(debounce)
对于短时间内连续触发的事件，在某个时间期限内，只执行最后一次触发的事件。

比如监听滚动条滚动事件，然后打印滚动的距离，如果不加防抖的话，打印的会特别频繁，但是在实际应用中，我们并不需要这么频繁的触发事件，此时可以为滚动事件加一个防抖函数，只在停止滚动xx（xx是设置的延迟时间）毫秒后，触发一次滚动事件，代码如下：
```javascript
/*
* fn [function] 需要防抖的函数
* delay [number] 毫秒，防抖期限值
*/
function debounce(fn, delay) {
    let timer = null // 借助闭包
    return function() {
        clearTimeout(timer) 
        timer = setTimeout(fn, delay)
    }
}

function showTop  () {
    var scrollTop = document.body.scrollTop || document.documentElement.scrollTop;
　　console.log('滚动条位置：' + scrollTop);
}
window.onscroll = debounce(showTop, 1000) // 在停止滚动1秒以后，会打印出滚动条位置
```
上面是一个最简单的示例代码，实际使用过程中还需要考虑到参数的传递问题，以及this指向问题，下面是实际使用过程中的代码：
```javascript
// 防抖方法
function debounce(fn, delay) {
    let timer = null
    return function () {
        let that = this
        let args = arguments
        clearTimeout(timer) // 清除重新计时
        timer = setTimeout(() => {
            fn.apply(that, args)
        }, delay || 500)
    }
}
// 声明事件处理函数时就可以加上事件的参数，因为debounce方法里面已经将参数携带了过来
function showTop  (event) {
    console.log(event) // event是滚动条滚动的事件参数，可以根据需要使用
    var scrollTop = document.body.scrollTop || document.documentElement.scrollTop;
　　console.log('滚动条位置：' + scrollTop);
}
```
#### （2）节流(throttle)
如果短时间内大量触发同一事件，那么在函数执行一次之后，在一定时间段内重复触发该函数会失效，过了这段时间之后该函数会重新生效（类似技能冷却时间）

比如点击按钮提交表单，为了防止用户连续点击触发多次提交请求，我们需要节流方法，首先设置一个时间段，在这个时间段内用户只有第一次点击时发起请求，后面的连续点击不会被处理
```javascript
// 这里只写节流方法，调用和防抖是一样的，就不重复写了
function throttle(fn, delay){
    let timer = null
    return function() {
       if(timer){
           //休息时间 暂不接客
           return false 
       }
       // 工作时间，执行函数并且在间隔期内把状态位设为无效
       fn()
       timer = setTimeout(() => {
           clearTimeout(timer)
           timer = null
       }, delay)
    }
}
/* 请注意，节流函数并不止上面这种实现方案,
   例如可以完全不借助setTimeout，可以把状态位换成时间戳，然后利用时间戳差值是否大于指定间隔时间来做判定。
   也可以额外创建一个开关变量，在触发函数后将其关闭，关闭期间不处理，然后设置定时器，定时器到期后再将开关打开，原理都是一样的
 */
```
同样的，下面是处理了参数传递的代码
```javascript
function throttle(fn, delay){
    let timer = null
    return function() {
       if(timer){
           //休息时间 暂不接客
           return false 
       }
       // 工作时间，执行函数并且在间隔期内把状态位设为无效
       let that = this
       let args = arguments
       fn.apply(that, args)
       timer = setTimeout(() => {
           clearTimeout(timer)
           timer = null
       }, delay || 500)
    }
}
```
#### （3）实现原理
从上面的代码可以看出，无论是防抖还是节流，都应用了闭包的特性（即一个函数A return了另一个函数B，则在函数B中可以使用函数A中的变量），在事件注册的时候，防抖和节流的方法只执行了一次，每次事件触发执行的是防抖节流方法里面return的方法
#### （4）应用场景
防抖和节流多使用于可能会频繁触发js事件或者请求的地方，为了提升浏览器的性能，常见的使用场景比如：

- 页面resize事件，常见于需要做页面适配的时候。需要根据最终呈现的页面情况进行dom渲染（这种情形一般是使用**防抖**，因为只需要判断最后一次的变化情况）
- 搜索框input事件，例如要支持输入实时搜索可以使用**节流方案**（间隔一段时间就必须查询相关内容），或者实现输入间隔大于某个值（如500ms），就当做用户输入完成，然后开始搜索（**防抖方案**），具体使用哪种方案要看业务需求。
- 表单提交，防止用户频繁点击，多次发起请求，这种情况一般使用**节流**

### 8.回流和重绘
#### （1）回流
当我们对 DOM 的修改引发了 DOM 几何尺寸的变化（比如修改元素的宽、高或隐藏元素等）时，浏览器需要重新计算元素的几何属性（其他元素的几何属性和位置也会因此受到影响），然后再将计算的结果绘制出来。这个过程就是回流（也叫重排）。

当你要用到像这样的属性：offsetTop、offsetLeft、 offsetWidth、offsetHeight、scrollTop、scrollLeft、scrollWidth、scrollHeight、clientTop、clientLeft、clientWidth、clientHeight 时，你就要注意了！

“像这样”的属性，到底是像什么样？——这些值有一个共性，就是需要通过即时计算得到。因此浏览器为了获取这些值，也会进行回流。

除此之外，当我们调用了 getComputedStyle 方法，或者 IE 里的 currentStyle 时，也会触发回流。原理是一样的，都为求一个“即时性”和“准确性”。
#### （2）重绘
当我们对 DOM 的修改导致了样式的变化、却并未影响其几何属性（比如修改了颜色或背景色）时，浏览器不需重新计算元素的几何属性、直接为该元素绘制新的样式（跳过了上图所示的回流环节）。这个过程叫做重绘。

#### （3）对比
重绘不一定导致回流，回流一定会导致重绘

#### （4）如何规避回流和重绘
- 将引起回流的操作或值用变量存起来，避免频繁改动
有时我们想要通过多次计算得到一个元素的布局位置，我们可能会这样做：
```vue
<template>
    <div ref="el" id="el"></div>
</template>
<script>
export default {
    mounted() {
        // 获取el元素
        // const el = document.getElementById('el')
        // vue中使用ref代替DOM操作获取元素
        const el = this.$refs.el
        // 这里循环判定比较简单，实际中或许会拓展出比较复杂的判定需求
        for(let i=0;i<10;i++) {
          el.style.top  = el.offsetTop  + 10 + "px";
          el.style.left = el.offsetLeft + 10 + "px";
        }
    }
}
</script>
```
这样做，每次循环都需要获取多次“敏感属性”，是比较糟糕的。我们可以将其以 JS 变量的形式缓存起来，待计算完毕再提交给浏览器发出重计算请求：
```vue
<template>
    <div ref="el" id="el"></div>
</template>
<script>
export default {
    mounted() {
        // 缓存offsetLeft与offsetTop的值
        const el = this.$refs.el
        let offLeft = el.offsetLeft
        let offTop = el.offsetTop

        // 在JS层面进行计算
        for(let i=0;i<10;i++) {
          offLeft += 10
          offTop  += 10
        }

        // 一次性将计算结果应用到DOM上
        el.style.left = offLeft + "px"
        el.style.top = offTop  + "px"
    }
}
</script>
```
- 避免逐条改变样式，使用类名去合并样式
比如我们可以把这段单纯的代码：
```javascript
const container = document.getElementById('container')
container.style.width = '100px'
container.style.height = '200px'
container.style.border = '10px solid red'
container.style.color = 'red'
```
优化成一个有 class 加持的样子：
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    .basic_style {
      width: 100px;
      height: 200px;
      border: 10px solid red;
      color: red;
    }
  </style>
</head>
<body>
  <div id="container"></div>
  <script>
  const container = document.getElementById('container')
  container.classList.add('basic_style')
  </script>
</body>
</html>
```
- 将 DOM “离线”
我们上文所说的回流和重绘，都是在“该元素位于页面上”的前提下会发生的。一旦我们给元素设置 display: none，将其从页面上“拿掉”，那么我们的后续操作，将无法触发回流与重绘——这个将元素“拿掉”的操作，就叫做 DOM 离线化。

仍以我们上文的代码片段为例：
```javascript
const container = document.getElementById('container')
container.style.width = '100px'
container.style.height = '200px'
container.style.border = '10px solid red'
container.style.color = 'red'
...（省略了许多类似的后续操作）
```
离线化后是这样的：
```javascript
let container = document.getElementById('container')
container.style.display = 'none'
container.style.width = '100px'
container.style.height = '200px'
container.style.border = '10px solid red'
container.style.color = 'red'
...（省略了许多类似的后续操作）
container.style.display = 'block'
```
- Flush 队列：浏览器并没有那么简单
其实现代浏览器是很聪明的。浏览器自己也清楚，如果每次 DOM 操作都即时地反馈一次回流或重绘，那么性能上来说是扛不住的。于是它自己缓存了一个 flush 队列，把我们触发的回流与重绘任务都塞进去，待到队列里的任务多起来、或者达到了一定的时间间隔，或者“不得已”的时候，再将这些任务一口气出队。因此我们看刚刚的逐条改变样式代码，就算我们进行了 4 次 DOM 更改，也只触发了一次 回流 和一次 重绘。
```javascript
// 我们即使这么写代码，浏览器的flush队列依旧会帮我们只执行一次回流和一次重绘
let container = document.getElementById('container')
container.style.width = '100px'
container.style.height = '200px'
container.style.border = '10px solid red'
container.style.color = 'red'
```
那是不是意味着我们就可以为所欲为，完全交给浏览器帮助优化了呢？当然是不可以的，这里尤其小心这个“不得已”的时候。前面我们在介绍回流的“导火索”的时候，提到过有一类属性很特别，它们有很强的“即时性”。当我们访问这些属性时，浏览器会为了获得此时此刻的、最准确的属性值，而提前将 flush 队列的任务出队——这就是所谓的“不得已”时刻。

所以我们还是要养成良好的编码习惯，不能过多依赖于浏览器的优化帮忙，上面的代码即便浏览器可以帮忙处理，我们也最好不要那么写。

### 9.关于性能优化的一些思路
#### （1）减少DOM操作
js引擎和渲染引擎是互相独立的，每操作一次DOM（**修改DOM属性**和**访问DOM属性值**都算是操作DOM），就需要js引擎和渲染引擎进行一次“跨界交流”，这种“跨界交流”的开销是比较高的，如果多次操作DOM，就会产生比较明显的性能问题。

事实上，考虑JS 的运行速度，比 DOM 快得多这个特性。我们减少 DOM 操作的核心思路，就是让 JS 去给 DOM 分压。这个思路，在 DOM Fragment 中体现得淋漓尽致。
```
DocumentFragment 接口表示一个没有父级文件的最小文档对象。它被当做一个轻量版的 Document 使用，用于存储已排好版的或尚未打理好格式的XML片段。因为 DocumentFragment 不是真实 DOM 树的一部分，它的变化不会引起 DOM 树的重新渲染的操作（reflow），且不会导致性能等问题。
```
举一个例子说明，需要很多的插入操作和改动，继续使用类似于下面的代码则会很有问题
```javascript
var ul = document.getElementById("ul");
for (var i = 0; i < 20; i++) {
    var li = document.createElement("li");
    li.innerHTML = "index: " + i;
    ul.appendChild(li);
}
```
由于每一次对文档的插入都会引起重新渲染（计算元素的尺寸，显示背景，内容等），所以进行多次插入操作使得浏览器发生了很多次渲染，效率是比较低的。这是我们提倡通过减少页面的渲染来提高DOM操作的效率的原因。一个优化的方法是将要创建的元素写到一个字符串上，然后一次性写到innerHTML上，这种利用浏览器对innerHTML的解析确实是相比上面的多次插入快了很多。但是构造字符串灵活性上面比较差，很难符合创建各种各样的DOM元素的需求。利用DocumentFragment，可以弥补这两个方法的不足。
```javascript
var ul = document.getElementById("ul");
var fragment = document.createDocumentFragment();
for (var i = 0; i < 20; i++) {
    var li = document.createElement("li");
    li.innerHTML = "index: " + i;
    fragment.appendChild(li);
}
ul.appendChild(fragment);
```

#### （2）防抖和节流的应用
对于多次触发的操作需要应用防抖和节流，具体参见本文第7点
#### （3）回流和重绘的优化
尽量减少或避免会产生回流与重绘的操作
