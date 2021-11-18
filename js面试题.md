### 1.js数据类型都有哪些
基本数据类型：Undefined, Null, Boolean, Number, String, Symbol(es6)

引用数据类型：Object, Array, Function

**主要区别**

- 声明变量时不同的内存分配：前者由于占据的空间大小固定且较小，变量名和值都会被存储在**栈**当中；后者变量名存储在**栈**中，值存储在**堆**中，变量访问的其实是一个指针，它指向存储对象的内存地址。
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

### 3. 请描述一下深拷贝和浅拷贝怎么实现
这里首先抛出一个问题：使用数组的slice和concat方法和对数组进行截取和合并，并返回一个新的数组，那么问题来了：如果调用slice和concat的数组是一个嵌套了对象的数组，那么返回的新数组，如果修改对象里面的值，会影响调用slice或者concat方法的数组吗？**答案是会：因为slice和concat方法是浅拷贝，如果只是一个数字类型的数组，那么使用slice或concat生成的新数组，修改新数组里面的值，是不会影响老数组的，因为进行了浅拷贝，而如果数组里面嵌套了对象，对象是没有进行拷贝的，而直接是引用，所以使用slice或concat生成的新的数组对象，修改对象里面的值，新旧数组都会改变**
#### （1）浅拷贝
浅拷贝只会克隆最外部的一层，至于更深层的对象，则依然是通过引用指向同一块堆内存。

浅拷贝的常见实现方式
##### ① 使用对象解构
```javascript
let obj2 = { ...obj }
```

##### ② 使用循环
```javascript
let obj2 = {};
for(let i in obj) {
    // for in 循环会遍历到对象的继承属性，我们只需要它的私有属性，所以可以加一个判断方法：hasOwnProperty 只拷贝对象私有属性
    if(!obj.hasOwnProperty(i)) continue;
    obj2[i] = obj[i];
}
```

##### ③ Object.assign(target,source)
这是ES6中新增的对象方法
```javascript
let obj2 = {};
Object.assign(obj2, obj); // 将 obj 拷贝到 obj2
```

#### （2）深拷贝
##### ① JSON.parse()和JSON.stringify
```javascript
let obj2 = JSON.parse(JSON.stringify(obj));

obj2.schoole.name= 'susan';
console.log(obj.school.name); // 'cherry'
// obj 中属性值并没有改变,说明是深拷贝
```
这种方法是比较简单的深拷贝，在对象属性的类型比较简单的时候，我们可以采取这种方法快速深拷贝。这个方法有下面几个问题
- 值为 **undefined** 的属性在转换后丢失；
- 值为 **Symbol 类型**的属性在转换后丢失；
- 值为 **RegExp 对象**的属性在转换后变成了空对象；
- 值为**函数对象**的属性在转换后丢失；
- 值为 **Date 对象**的属性在转换后变成了字符串；
- **会抛弃对象的 constructor,所有的构造函数会指向 Object**；
- 对象的**循环引用**会抛出错误。

##### ② 自己手写
实现思路：

1. 遍历待拷贝的对象，判断是不是原始值，若是，使用浅拷贝的方式进行赋值。

2. 若是引用值，将**特殊类型逐一进行过滤**，并且兼容引用值是数组的情况。

3.待拷贝的对象里面的若是原始值，则浅拷贝即可实现，若还有引用值，则还需要重复进行上述一系列的判断（递归赋值）。

这里需要注意的两个点就是：使用递归，**特殊类型特殊处理**

```javascript
let obj = {
    a: '100',
    b: undefined,
    c: null,
    d: Symbol(2),
    e: /^\d+$/,
    f: new Date,
    g: true,
    arr: [10,20,30],
    school:{
        name: 'cherry',
    },
    fn: function fn() {
        console.log('fn');    
    }
}

function deepClone(obj) {
    // 先把特殊情况全部过滤掉 null undefined date reg
    if (obj === null || obj === undefined) return obj;  // null 和 undefined 都不用处理
    if (obj instanceof Date) return new Date(obj.getTime());
    if (obj instanceof RegExp) return new RegExp(obj);
    if (typeof obj !== 'object') return obj;  // 普通常量直接返回，函数也直接返回

    // 不直接创建空对象的目的：克隆的结果和之前保持相同的所属类，
    // 同时也兼容了数组的情况
    let newObj = new obj.constructor;
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {  // 不拷贝原型链上的属性
            newObj[key] = deepClone(obj[key]);  // 递归赋值
        }
    }
    return newObj;
}
let obj2 = deepClone(obj);
console.log(obj2);
```

代码写到这里，我们就实现了一种比较简单的深拷贝，面试的时候如果你能写出上面的实现方法，应该算是及格啦！但是，这个方法还有如下的问题：
- 为对象添加一个 Symbol 类型的**key**时，不会拷贝成功。解决方法：**使用 Object 提供的 getOwnPrepertySymbols()方法来枚举对象中所有 key 是 symbol 类型的属性**
- 如果在我们拷贝的对象被循环引用，deepClone就会一直执行下去导致爆栈。解决方法：**使用 es6 的 WeakMap 来生成一个 hash 表，在赋值之前判断当前值是否已经存在，避免循环引用**

```javascript
let s1 = Symbol('s1');

let obj = {
    a: '100',
    b: undefined,
    c: null,
    d: Symbol(2),
    e: /^\d+$/,
    f: new Date,
    g: true,
    arr: [10,20,30],
    school:{
        name:'cherry',
        [s1]: 's1'
    },
    fn: function fn() {
        console.log('fn');    
    }
}

obj.h = obj;

function deepClone(obj, hash = new WeakMap()) {
    //先把特殊情况全部过滤掉 null undefined date reg
    if (obj === null || obj === undefined) return obj;  // null 和 undefined 都不用处理
    if (obj instanceof Date) return new Date(obj.getTime());
    if (obj instanceof RegExp) return new RegExp(obj);
    if (typeof obj !== 'object') return obj;  // 普通常量直接返回
    
    //  防止对象中的循环引用爆栈，把拷贝过的对象直接返还即可
    if (hash.has(obj)) return hash.get(obj);

    // 不直接创建空对象的目的：克隆的结果和之前保持相同的所属类
    // 同时也兼容了数组的情况
    let newObj = new obj.constructor;

    hash.set(obj, newObj)  // 制作一个映射表
    
    //判断是否有 key 为 symbol 的属性
    let symKeys = Object.getOwnPropertySymbols(obj);
    if (symKeys.length) { 
        symKeys.forEach(symKey => {
            newObj[symKey] = deepClone(obj[symKey], hash);   
        });
    }

    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {  // 不拷贝原型链上的属性
            newObj[key] = deepClone(obj[key], hash);  // 递归赋值
        }
    }
    return newObj;
}
let obj2 = deepClone(obj);
console.log(obj2);
```

##### ③ 使用lodash库
lodash库里面有deepClone方法的实现，在生产中最好使用lodash



### 4.请描述一下 cookies sessionStorage和localstorage区别
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

### 5.作用域的概念
提到作用域，不得不提一下es6新增的let和const的变量声明方式，在es6之前，通过var声明的变量具有**变量提升**的特性，但是使用es6的let和const声明的变量不会变量提升，且具备**块级作用域**的特性，即只在所处的块级内部有效，而且不能在声明之前使用（官方术语：暂时性死区）。
```javascript
// 关于变量提升的一段代码
var a = 99;            // 全局变量a
f();                   // f是函数，虽然定义在调用的后面，但是函数声明会提升到作用域的顶部。 
console.log(a);        // a=>99,  此时是全局变量的a
function f() {
  console.log(a);      // 当前的a变量是下面变量a声明提升后，默认值undefined
  var a = 10;
  console.log(a);      // a => 10
}

// 输出结果：
undefined
10
99
```

### 6.对闭包的理解
闭包就是：一个函数里面return了一个子函数，子函数可以访问父函数定义的变量。

应用场景：防抖和节流会使用到闭包，封装私有变量的时候也会用到闭包

**闭包是如何产生的**

因为Javascript语言特有的"链式作用域"结构（chain scope），

子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。所以只要在函数a里面再定义一个函数b，函数b就可以访问函数a的变量，再将函数b返回，就可以实现函数外部访问函数内部的变量

**注意**

闭包在使用完毕后记得释放，否则会一直在内存中
```javascript
function findGf(){
	 this.name = '博主是帅哥';
     this.leg = 160;
 }
 findGf.prototype.selectName = function(){
     let name = this.name;
     return function(){
         return name
     };
     // 闭包里的name是不会被回收的，一直存在于内存中，所以用完要及时销毁name，销毁的方法很简单，置为null就行了
     name = null;
 };
 let gf = new findGf();
 console.log(gf.selectName()());
```

### 7.原型和原型链介绍一下（__proto__, prototype, constructor的关系）

#### （1）首先需要牢记两点
##### ①__proto__和constructor属性是对象所独有的；
##### ② prototype属性是函数所独有的，因为函数也是一种对象，所以函数也拥有__proto__和constructor属性。

#### （2）__proto__
__proto__属性的作用就是当访问一个对象的属性时，如果该对象内部不存在这个属性，那么就会去它的__proto__属性所指向的那个对象（父对象）里找，以此类推，直到__proto__属性的终点null，再往上找就相当于在null上取值，会报错。通过__proto__属性将对象连接起来的这条链路即我们所谓的**原型链**。

#### （2）prototype
prototype属性的作用就是让该函数所实例化的对象们都可以找到公用的属性和方法，即f1.__proto__ === Foo.prototype。

#### （3）constructor
constructor属性的含义就是**指向该对象的构造函数**，所有函数（此时看成对象了）最终的构造函数都指向Function。

### 8. js中继承的几种方式，有什么优缺点
现在都使用es6语法写class类，用extends继承了

### 9.call和apply简单介绍一下
#### apply：

接受两个参数，第一个参数是要绑定给this的值，第二个参数是一个参数数组。当第一个参数为null、undefined的时候，默认指向window。

#### call：

第一个参数是要绑定给this的值，后面传入的是一个参数列表。当第一个参数为null、undefined的时候，默认指向window。

#### apply和call:

事实上apply 和 call 的用法几乎相同。 唯一的差别在于：当函数需要传递多个变量时, apply 可以接受一个数组作为参数输入, call 则是接受一系列的单独变量。

### 10.js异步操作介绍一下

#### （1）promise
通常通过promise实现，promise有三种状态，pending（等待中）、resolve（执行成功）、reject（执行失败），在执行promise的时候，状态总会从pending到resolve或者reject，可以用then接收成功的回调，catch接收失败的回调，**另外：如果不想写catch也可以只用then实现接收失败回调，因为then回调有两个参数，第一个参数代表成功的回调，第二个参数代表失败的回调**

**Promise.all方法**

Promise.all方法接收一个数组作为参数，数组里面的每一项都是一个promise对象，这个方法最后返回一个promise对象，表示参数promise对象都被处理完之后的结果。
```javascript
var p = Promise.all([p1,p2,p3]);
```
上面的例子，p的状态由p1、p2、p3决定，分成两种情况。

- 只有p1、p2、p3的状态都变成resolve，p的状态才会变成resolve，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
- 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

**Promise.race方法**

Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例。
```javascript
var p = Promise.race([p1,p2,p3]);
```
上面代码中，只要p1、p2、p3之中**有一个实例率先改变状态，p的状态就跟着改变**。那个率先改变的Promise实例的返回值，就传递给p的返回值。

如果Promise.all方法和Promise.race方法的参数，不是Promise实例，就会先调用Promise.resolve方法，将参数转为Promise实例，再进一步处理。

#### （2）async、await
使用es7的 async和await也可以实现异步，而且可以做到**以同步写代码的方式实现异步功能，代码书写更加简洁**，async和await主要是解决了promise的then链过多问题，如果promise没有那么多then链，用promise也可以

#### （3）手写promise
https://blog.csdn.net/William_bb/article/details/103169558?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control

上面这个文章讲的比较透彻

主要注意的几个点就是：
- new Promise 里面的代码是**同步**的，Promise.then.catch里面的方法才是异步的，所以在手写promise的时候，自定义的promise的构造函数需要**立即执行传入promise的方法**
- new Promise 的then和catch回调都是**微任务**，我们使用setTimeout模拟实现的，但是setTimeout是**宏任务**，所以，我们的**模拟实现无法百分百还原**，这个要注意。
- 如何实现在new promise里面**异步调用resolve或reject**？
    - 异步调用resolve可能**then方法会在调用resolve之前执行**，这时候由于还没调用resolve，所以状态还是pending，此时调用then自然不会触发里面的回调函数。
    - 解决方法就是：在new Promise的时候创建两个数组保存then和catch里面的回调函数，**在调用then方法的时候，如果状态还是pending，就把then的回调函数放入数组中缓存起来**。然后在执行resolce方法的时候，把数组里面的方法依次执行即可。
- 如何模拟实现**then方法的链式操作**？
    - **时刻记住Promise规范**：then方法执行后默认返回的是一个**新的Promise对象**，而不是原来的Promise对象
    - 根据上述Promise规范，其实then方法的链式操作就是**递归实例化Promise对象**，至于参数在多个then链之间的传递则是：**上一个then里面回调方法的返回值作为下一个then里面回调方法的参数**

### 11.JS事件循环、任务队列、宏任务、微任务？
#### （1）同步与异步简介
我们知道，Javascript语言的执行环境是**单线程**（single thread）的。

所谓"单线程"，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。

这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段Javascript代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

为了解决这个问题，Javascript语言将任务的执行模式分成两种：**同步（Synchronous）**和**异步（Asynchronous）**。

**同步模式**就是后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；

**异步模式**则完全不同，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行队列上的后一个任务，而是执行回调函数；后一个任务则是不等前一个任务的回调函数的执行而执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。

#### （2）任务队列
异步操作会将相关回调添加到任务队列中。而不同的异步操作添加到任务队列的时机也不同，如onclick, setTimeout,ajax 处理的方式都不同，这些任务就构成了任务队列。

#### （3）宏任务和微任务
任务队列中的任务，又分为**宏任务**和**微任务**两种：

##### ① 宏任务(macrotask)
script（整体代码：你的全部JS代码，包括“同步代码”和“异步代码”）, setTimeout, setInterval, setImmediate, I/O, UI rendering

宏任务中的任务优先级：setTimeout = setInterval > setImmediate

##### ② 微任务(microtask)
process.nextTick, Promises（这里指浏览器实现的原生 Promise，注意：Promise中的代码是同步的，then里面的才是异步的微任务）, Object.observe, MutationObserver

微任务的任务优先级：process.nextTick > Promise

**注意：**
- 同层级的微任务是优先于宏任务执行的，即不论任务注册的先后顺序，总是执行完所有微任务后，再去执行宏任务，当然，注意这里说的是同层级的微任务和宏任务，因为还可能存在宏任务嵌套微任务的情况
```javascript
Promise.resolve().then(()=>{
  console.log('Promise1')  
  setTimeout(()=>{
    console.log('setTimeout2')
  },0)
})

setTimeout(()=>{
  console.log('setTimeout1')
  Promise.resolve().then(()=>{
    console.log('Promise2')    
  })
},0)

// 最后输出结果是Promise1，setTimeout1，Promise2，setTimeout2
```

- 一个事件循环可以有多个任务队列，队列之间可有不同的优先级，同一队列中的任务按先进先出的顺序执行，但是不保证多个任务队列中的任务优先级，具体实现可能会交叉执行。
```javascript
setTimeout(function(){
    console.log(2);
},0);
 
new Promise(function(resolve){
    console.log(3);
    resolve();
    console.log(4);
}).then(function(){
    console.log(5);
});
 
console.log(6);
 
setTimeout(function(){
    console.log(7);
},0);
 
console.log(8);
// 输出结果是，3 4 6 8 5 2 7
```

#### （4）事件循环
- 在浏览器环境中，首先在 macrotask 的队列（这个队列也被叫做 task queue）中取出第一个任务（即整体代码），执行整体代码中所有的同步任务，同时将异步任务注册到任务队列里面
- 然后再取出 任务队列 中的所有微任务按照优先级执行（先清空process.nextTick队列，再清空promise.then队列）
- 之后再取出 任务队列 中的下一个宏任务执行，**如果这个宏任务子级有微任务，执行完这个宏任务需要将其子层级下的微任务按照优先级执行**
- 以此类推，直到任务队列中的任务都执行完毕

上述的任务执行过程就是事件循环

最后看一个复杂一些的例子：
```javascript
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})

new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

process.nextTick(function() {
    console.log('6');
})

setTimeout(function() {
    console.log('9');
    
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
    process.nextTick(function() {
        console.log('10');
    })
})

// 整段代码，共进行了三次事件循环，完整的输出为1，7，6，8，2，4，3，5，9，11，10，12
```

**第一轮事件循环**流程分析如下：

- 整体script作为第一个宏任务进入主线程，遇到console.log，输出1。
- 遇到setTimeout，其回调函数被分发到宏任务Event Queue中。我们暂且记为setTimeout1。
- 遇到Promise，new Promise直接执行，输出7。then被分发到微任务Event Queue中。我们记为then1。
- 遇到process.nextTick()，其回调函数被分发到微任务Event Queue中。我们记为process1。
- 又遇到了setTimeout，其回调函数被分发到宏任务Event Queue中，我们记为setTimeout2。

| 宏任务Event Queue | 微任务Event Queue |
| ---- | ---- |
| setTimeout1 | then1 |
| setTimeout2 | process1 |

- 上表是第一轮事件循环宏任务结束时各Event Queue的情况，此时已经输出了1和7。

我们发现了process1和then1两个微任务。虽然Promise比nextTick先注册，但是nextTick比Promise优先级更高，所以：

- 先执行process1,输出6。
- 再执行then1，输出8。

好了，第一轮事件循环正式结束，这一轮的结果是输出1，7，6，8。

**第二轮事件循环**从setTimeout1宏任务开始：

- 首先输出2。接下来遇到了process.nextTick()，同样将其分发到微任务Event Queue中，记为process2。
- new Promise立即执行输出4，then也分发到微任务Event Queue中，记为then2

| 宏任务Event Queue | 微任务Event Queue|
| ---- | ---- |
| setTimeout2 | process2 |
| | then2 |

- 执行process2，输出3
- 再执行then2，输出5

第二轮事件循环正式结束，这一轮的结果是输出2，4，3，5。

**第三轮事件循环**从setTimeout2宏任务开始，过程和第二轮事件循环是类似的，这里就不展开了，第三轮输出9，11，10，12。

最后的输出为1，7，6，8，2，4，3，5，9，11，10，12。(请注意，node环境下的事件监听依赖libuv与前端环境不完全相同，输出顺序可能会有误差)


### 12.es6新特性
#### （1）变量声明const 和 let
作用：

1.防止全局变量泄露

2.防止变量提升带来的覆盖问题

#### （2）字符串模板

#### （3）箭头函数
- 普通函数this指向会发生改变
- 箭头函数则指向函数上下文中this所指的对象
#### （4）类的引入
在ES6 中引入了类的概念，使用class即可声明一个类，对于类的继承写起来也比较易懂
#### （5）形参默认值
在ES6中可以直接在函数声明的过程中设置默认参数
```javascript
var link = function(height = 50, color = 'red', url = 'http://azat.co') {
  ...
}
```
#### （6）promise、async/await介绍
es6 中可以使用promise风格书写异步代码，通过.then捕获resolve，.catch捕获reject。
同时es7 推出了async和await，await的出现实现了以同步代码的方式书写异步代码，解决了promise过多产生的then链问题
#### （7）set和map
##### Maps 和 Objects 的区别
- 一个 Object 的键只能是字符串或者 Symbols，但一个 Map 的键可以是**任意数据类型**。
- Map 中的键值是**有序的**（FIFO 原则），而添加到对象中的键则不是。
- Map 的**键值对个数可以从 size 属性获取**，而 Object 的键值对个数只能手动计算。
- Object 都有自己的原型，原型链上的键名有可能和你自己在对象上的设置的键名产生冲突。

##### set
set中**元素不允许重复**

set可用作数组去重
```javascript
var mySet = new Set([1, 2, 3, 4, 4]);
[...mySet]; // [1, 2, 3, 4]
```

#### （8）WeakMap
WeakMap与Map类似，但有几点区别：

1、WeakMap只接受对象作为key，如果设置其他类型的数据作为key，会报错。

2、WeakMap的key所引用的对象都是弱引用，只要对象的其他引用被删除，垃圾回收机制就会释放该对象占用的内存，从而避免内存泄漏。

3、由于WeakMap的成员随时可能被垃圾回收机制回收，成员的数量不稳定，所以没有size属性。

4、没有clear()方法

5、不能遍历

#### （9）for-in 和 for-of
- for in 取 key；for of 取 value
- for of只能用于数组遍历，for in 还可以用于对象属性的遍历

### 13.防抖和节流
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

### 14.回流和重绘
#### （1）回流
当我们对 DOM 的修改引发了 DOM 几何尺寸的变化（比如修改元素的宽、高或隐藏元素等）时，浏览器需要重新计算元素的几何属性（其他元素的几何属性和位置也会因此受到影响），然后再将计算的结果绘制出来。这个过程就是回流（也叫重排）。**浏览器窗口大小发生变化也会引起回流**

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

### 15.关于性能优化的一些思路
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

### 16.JS内存泄漏
#### （1）什么是内存泄漏？
不再用到的内存（变量），没有及时释放，就叫内存泄漏

#### （2）内存泄漏的场景
##### ① 意外的全局变量
##### ② 没有及时清除的定时器
##### ③ 使用不当的闭包
正常来说，闭包并不是内存泄漏，因为这种持有外部函数词法环境本就是闭包的特性。

所以，闭包中返回的函数，它的生命周期不宜过长，方便闭包及时被回收。

##### ④遗漏的DOM元素
DOM元素的生命周期正常是取决于是否挂载在DOM树上，当从DOM树上移除时，就可以被销毁回收了。

但如果某个DOM元素，在js中也持有它的引用时，**那么它的生命周期就由js和是否在DOM树上两者决定了**。移除时，两个地方都需要清理才能回收它。

#### （3）内存泄漏的识别方法
##### ① 浏览器
Chrome 浏览器查看内存占用，按照以下步骤操作。
- 打开开发者工具，选择 Performance 面板
- 点击左上角的录制按钮。
- 在页面上进行各种操作，模拟用户的使用情况。
- 一段时间后，点击对话框的 stop 按钮，面板上就会显示这段时间的内存占用情况。
- 如果内存占用基本平稳，接近水平，就说明不存在内存泄漏。
- 反之，就是内存泄漏了。

##### ② 命令行
命令行可以使用 Node 提供的process.memoryUsage方法。
```javascript
console.log(process.memoryUsage());
// { 
//      rss: 27709440,
//      heapTotal: 5685248,
//      heapUsed: 3449392,
//      external: 8772 
// }
```
process.memoryUsage返回一个对象，包含了 Node 进程的内存占用信息。该对象包含四个字段，单位是字节，含义如下。
- rss（resident set size）：所有内存占用，包括指令区和堆栈。
- heapTotal："堆"占用的内存，包括用到的和没用到的。
- heapUsed：用到的堆的部分。
- external： V8 引擎内部的 C++ 对象占用的内存。

**判断内存泄漏，以heapUsed字段为准。**

### 17.V8引擎垃圾回收机制
#### （1）介绍
V8的垃圾回收策略主要是基于**分代式垃圾回收机制**，其根据对象的存活时间将内存的垃圾回收进行不同的分代，然后对不同的分代采用不同的垃圾回收算法。

**注意**

由于JS是单线程的，所以在V8执行垃圾回收时，程序中的其他各种逻辑都要进入暂停等待阶段，**直到垃圾回收结束后才会再次重新执行JS逻辑**。

#### （2）V8的内存结构
在V8引擎的堆结构组成中，其实除了**新生代**和**老生代**外，还包含其他几个部分，但是垃圾回收的过程主要出现在新生代和老生代，所以对于其他的部分我们没必要做太多的深入：

#### （3）新生代
##### ① 介绍
在V8引擎的内存结构中，新生代主要用于存放存活时间较短的对象。新生代内存是由两个semispace(半空间)构成的，在新生代的垃圾回收过程中主要采用了**Scavenge**算法。
##### ② Scavenge算法
在**Scavenge**算法的具体实现中，主要采用了**Cheney**算法，它将新生代内存一分为二，每一个部分的空间称为**semispace**（半空间），其中处于激活状态的区域我们称为**From**空间，未激活(inactive new space)的区域我们称为**To**空间。这两个空间中，**始终只有一个处于使用状态**，另一个处于闲置状态。Scavenge算法的流程如下：
- 我们的程序中声明的对象首先会被分配到**From**空间；
- 当进行垃圾回收时，如果**From**空间中尚有存活对象，则会被复制到**To**空间进行保存；
- 当复制完成后，**把From空间清空**，**From**空间和**To**空间完成一次角色互换，**To**空间会变为新的**From**空间，原来的**From**空间则变为**To**空间。

**结论**：

- **Scavenge**算法的垃圾回收过程主要就是将存活对象在From空间和To空间之间进行复制，同时完成两个空间之间的角色互换，因此该算法的缺点也比较明显，浪费了一半的内存用于复制。是一种典型的**牺牲空间换取时间**的算法
- 在新生代内存中，大部分对象的生命周期较短，在时间效率上表现可观，所以还是比较适合这种算法。

#### （4）对象晋升
当一个对象在经过多次复制之后依旧存活，那么它会被认为是一个生命周期较长的对象，在下一次进行垃圾回收时，该对象会被直接转移到老生代中，这种对象从新生代转移到老生代的过程我们称之为**晋升**。

对象晋升的条件主要有以下两个，满足其一即可从新生代转移到老生代：

- 对象是否经历过一次Scavenge算法
- To空间的内存占比是否已经超过25%

默认情况下，**我们创建的对象都会分配在From空间中**，当进行垃圾回收时，**在将对象从From空间复制到To空间之前**，会先检查该对象的内存地址来判断是否已经经历过一次**Scavenge**算法，如果地址已经发生变动则会将该对象转移到老生代中；如果对象没有经历过**Scavenge**算法，但是如果此时**To**空间的内存占比已经超过**25%**，则该对象依旧会被转移到老生代。

之所以有25%的内存限制是因为**To**空间在经历过一次**Scavenge**算法后会和**From**空间完成角色互换，会变为**From**空间，后续的内存分配都是在**From**空间中进行的，**如果内存使用过高甚至溢出，则会影响后续对象的分配**，因此超过这个限制之后对象会被直接转移到老生代来进行管理。

#### （5）老生代
##### ① 介绍
在老生代中，因为管理着大量的存活对象，如果依旧使用Scavenge算法的话，很明显会浪费一半的内存，因此已经不再使用**Scavenge**算法，而是采用新的算法**Mark-Sweep(标记清除)**和**Mark-Compact(标记整理)**来进行管理。

##### ② 引用计数法
在早前我们可能听说过一种算法叫做**引用计数**，该算法的原理比较简单，就是**看对象是否还有其他引用指向它**，如果没有指向该对象的引用，则该对象会被视为垃圾并被垃圾回收器回收。

示例:
```javascript
let arr = [1, 2, 3, 4];
arr = null; // [1,2,3,4]这时没有被引用, 会被自动回收
```

**限制：循环引用**

在下面的例子中, 两个对象对象被创建并互相引用, 就造成了循环引用. 它们被调用之后不会离开函数作用域, 所以它们已经没有用了, 可以被回收了. 然而, 引用计数算法考虑到它们互相都有至少一次引用, 所以它们不会被回收。
```javascript
function f() {
  var o1 = {};
  var o2 = {};
  o1.p = o2; // o1 引用 o2
  o2.p = o1; // o2 引用 o1. 这里会形成一个循环引用
}
f();
```
为了避免循环引用导致的内存泄漏问题，截至2012年所有的现代浏览器均放弃了这种算法，转而采用新的**Mark-Sweep(标记清除)**和**Mark-Compact(标记整理)**算法。

##### ③ Mark-Sweep(标记清除)
Mark-Sweep(标记清除)分为**标记**和**清除**两个阶段，在标记阶段会遍历堆中的所有对象，然后标记活着的对象，在清除阶段中，会将死亡的对象进行清除。**Mark-Sweep**算法主要是通过判断某个对象是否可以被访问到，从而知道该对象是否应该被回收，具体步骤如下：

- 垃圾回收器会在内部构建一个**根列表**，用于从根节点出发去寻找那些可以被访问到的变量。比如在JavaScript中，window全局对象可以看成一个根节点。
- 然后，垃圾回收器从所有根节点出发，遍历其可以访问到的子节点，并将其**标记**为活动的，根节点不能到达的地方即为非活动的，将会被视为垃圾。
- 最后，垃圾回收器将会释放所有非活动的内存块，并将其归还给操作系统。

以下几种情况都可以作为根节点：

- 全局对象
- 本地函数的局部变量和参数
- 当前嵌套调用链上的其他函数的变量和参数

**限制：内存碎片问题**

**Mark-Sweep**算法存在一个问题，就是在经历过一次标记清除后，内存空间可能会出现**不连续**的状态，因为我们所清理的对象的内存地址可能不是连续的，所以就会出现**内存碎片**的问题，导致后面如果需要分配一个大对象而空闲内存不足以分配，就会提前触发垃圾回收，而这次垃圾回收其实是没必要的，因为我们确实有很多空闲内存，只不过是不连续的。

##### ④ Mark-Compact(标记整理)
为了解决这种内存碎片的问题，**Mark-Compact**(标记整理)算法被提了出来，该算法主要就是用来解决内存的碎片化问题的，回收过程中将死亡对象清除后，在整理的过程中，会将活动的对象往堆内存的一端进行移动，移动完成后再清理掉边界外的全部内存。

至此就完成了一次老生代垃圾回收的全部过程，我们在前文中说过，由于JS的单线程机制，垃圾回收的过程会阻碍主线程同步任务的执行，待执行完垃圾回收后才会再次恢复执行主任务的逻辑，这种行为被称为**全停顿**(stop-the-world)。在标记阶段同样会阻碍主线程的执行，一般来说，老生代会保存大量存活的对象，如果在标记阶段将整个堆内存遍历一遍，那么势必会造成严重的卡顿。

因此，为了减少垃圾回收带来的停顿时间，V8引擎又引入了Incremental Marking(**增量标记**)的概念，即将原本需要一次性遍历堆内存的操作改为增量标记的方式，先标记堆内存中的一部分对象，然后暂停，将执行权重新交给JS主线程，待主线程任务执行完毕后再从原来暂停标记的地方继续标记，直到标记完整个堆内存。这个理念其实有点像React框架中的Fiber架构，只有在浏览器的空闲时间才会去遍历Fiber Tree执行对应的任务，否则延迟执行，尽可能少地影响主线程的任务，避免应用卡顿，提升应用性能。

得益于**增量标记**的好处，V8引擎后续继续引入了**延迟清理**(lazy sweeping)和**增量式整理**(incremental compaction)，让清理和整理的过程也变成增量式的。同时为了充分利用多核CPU的性能，也将引入并行标记和并行清理，进一步地减少垃圾回收对主线程的影响，为应用提升更多的性能。

### 18.JS设计模式及应用

#### （1）单例模式
##### ① 定义
保证一个类仅有一个实例，并提供一个访问它的全局访问点
##### ② 核心
确保只有一个实例，并提供全局访问
##### ③ 应用场景
适用于全局只存在一个实例的场景，比如页面上只存在一个能做动画的元素，或者系统中只有一个超级管理员。

全局只存在一个实例的方法就是在方法内判断实例存在就直接返回，否则才创建实例

#### （2）策略模式
##### ① 定义
定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。
##### ② 核心
将**算法的使用**和**算法的实现**分离开来。

一个基于策略模式的程序至少由两部分组成：

第一个部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程（**算法的实现**）。

第二个部分是环境类Context，Context接受客户的请求，随后把请求委托给某一个策略类（**算法的使用**）。
##### ③ 应用场景
该模式应用场景很多，绝大多数功能都可以用这种**使用**和**实现**分离的模式来做，比较常见的比如表单的验证，我们把表单的验证规则都写到一起（**算法的实现**），然后在用到的地方根据需求进行调用（**算法的使用**）。

### 19.什么是柯里化？柯里化有什么好处？
#### （1）柯里化的定义
柯里化是一种将使用**多个参数的一个函数**转换成一系列**使用一个参数的函数**的技术。
#### （2）柯里化的好处
##### ① 参数复用
##### ② 利用闭包的特性，减少多次调用函数的重复判断
#### （3）柯里化的实现
**实现思路**

- 由于函数可以链式调用，也就是说**函数调用的返回结果还是函数**，这里用到的方法就是利用**递归和闭包**，通过将多次调用的函数的参数 传到 数组 里面进行缓存
- 下面方法的 \_args 数组就把每次调用方法的参数通过push进行收集，当 \_args.length >= length 的时候，就说明参数收集够了，直接调用 fn 即可

```javascript
function add(a, b, c) {
  return a + b + c;
}

/**
* fn 要柯里化的函数
* args 参数列表（负责收集参数，用来判断是递归调用curry还是直接触发fn）
*/
function curry(fn, args) {
  var length = fn.length; // fn.length 可以取到fn函数的参数个数

  args = args || [];

  return function () {
    var _args = args.slice(); // 浅拷贝

    // arguments 是调用这个 返回函数 的时候传入的参数
    for (var i = 0; i < arguments.length; i++) {
      _args.push(arguments[i]);
    }
    if (_args.length < length) {
      return curry.call(this, fn, _args);
    } else {
      return fn.apply(this, _args);
    }
  };
}

var curryAdd = curry(add);
console.log(curryAdd(1, 3)(2));

// 实现目标：链式调用
// var curryAdd = curry(add)
// curryAdd(1, 2, 3) // 6
// curryAdd(1, 2)(3) // 6
// curryAdd(1)(2, 3) // 6
// curryAdd(1)(2)(3) // 6
```
