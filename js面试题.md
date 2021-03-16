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

instanceof运算符用于判断的是B.prototype是否存在A的原型链之中
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
1.存储大小

· cookie数据大小不能超过4k。

· sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。

2.有效时间

· localStorage    存储持久数据，浏览器关闭后数据不丢失除非主动删除数据；

· sessionStorage  数据在当前浏览器窗口关闭后自动删除。

· cookie          设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭

3. 数据与服务器之间的交互方式

· cookie的数据会自动的传递到服务器，服务器端也可以写cookie到客户端

· sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存。

### 3.call和apply简单介绍一下
#### apply：

接受两个参数，第一个参数是要绑定给this的值，第二个参数是一个参数数组。当第一个参数为null、undefined的时候，默认指向window。

#### call：

第一个参数是要绑定给this的值，后面传入的是一个参数列表。当第一个参数为null、undefined的时候，默认指向window。

#### apply和call:

事实上apply 和 call 的用法几乎相同。 唯一的差别在于：当函数需要传递多个变量时, apply 可以接受一个数组作为参数输入, call 则是接受一系列的单独变量。

### 3.js异步操作介绍一下
### 4.es6新特性

