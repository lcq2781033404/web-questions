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
#### （2）重绘
当我们对 DOM 的修改导致了样式的变化、却并未影响其几何属性（比如修改了颜色或背景色）时，浏览器不需重新计算元素的几何属性、直接为该元素绘制新的样式（跳过了上图所示的回流环节）。这个过程叫做重绘。

#### （3）对比
重绘不一定导致回流，回流一定会导致重绘

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
