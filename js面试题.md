### 1.js数据类型都有哪些
基本数据类型：Undefined, Null, Boolean, Number, String, Symbol(es6)
引用数据类型：Object, Array, Function
**主要区别**
- 声明变量时不同的内存分配：前者由于占据的空间大小固定且较小，会被存储在**栈**当中，也就是变量访问的位置；后者则存储在**堆**当中，变量访问的其实是一个指针，它指向存储对象的内存地址。
- 也正是因为内存分配不同，在复制变量时也不一样。前者复制后2个变量是独立的，因为是把值拷贝了一份；后者则是复制了一个指针，2个变量指向的值是该指针所指向的内容，一旦一方修改，另一方也会受到影响。
**注意**
这里有个规定，原始数据类型的值，也称原始值是无法被修改的。即只有Object类型的数据才可被修改。

### 2.typeof运算符和instanceof运算符以及isPrototypeOf()方法的区别

### 2.请描述一下 cookies sessionStorage和localstorage区别
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

### 3.js异步操作介绍一下
### 4.es6新特性

