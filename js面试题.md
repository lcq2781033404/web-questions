### 1.js数据类型都有哪些
基本数据类型：Undefined, Null, Boolean, Number, String
复杂数据类型：Object, Array, Function
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

