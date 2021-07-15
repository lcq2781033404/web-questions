### 1.对MVVM模式的理解
MVVM分为Model、View、ViewModel三者。

Model：代表数据模型，数据和业务逻辑都在Model层中定义；
View：代表UI视图，负责数据的展示；
ViewModel：负责监听Model中数据的改变并且控制视图的更新，处理用户交互操作；
       Model和View并无直接关联，而是通过ViewModel来进行联系的，Model和ViewModel之间有着双向数据绑定的联系。因此当Model中的数据改变时会触发View层的刷新，View中由于用户交互操作而改变的数据也会在Model中同步。

这种模式实现了Model和View的数据自动同步，因此开发者只需要专注对数据的维护操作即可，而不需要自己操作dom。

### 2.vue响应式原理介绍一下
Vue 采用观察者模式，通过Object.defineProperty将对象的key转换成getter/setter来追踪变化，就是所谓的**数据劫持**，Vue中需要收集视图依赖了哪些数据，在getter中收集依赖，在setter中触发依赖。先收集依赖，即把用到该数据的地方收集起来，然后等属性发生变化时，把之前收集好的依赖循环触发一遍，这样就完成了数据更新，视图随之发生变化的效果，但是defineProperty的getter/setter只能追踪一个数据是否被修改，**无法追踪新增属性和删除属性**。对于要新增的属性，Vue给出了Vue.set的方法（Vue.set方法调用了defineReactive方法将添加的属性变为响应式的）响应式的添加属性。

所以Vue 3.0的时候，Vue是通过Proxy代理的方式监测属性变化的，解决了defineProperty无法追踪新增属性和删除属性的问题，而且Proxy 的代理是**针对整个对象的，而不是对象的某个属性**，因此不同于 Object.defineProperty 的必须遍历对象每个属性。



### 3.vue生命周期介绍一下
#### 基本概念
vue实例从创建到销毁的过程

#### vue生命周期的作用
vue生命周期中的多个事件钩子，可以让我们在控制整个Vue实例的过程时更容易形成较为清晰的逻辑。

#### vue生命周期有几个阶段
总共分为8个阶段：创建前/后(beforeCreate, created), 载入前/后(beforeMount, mounted), 更新前/后(beforeUpdate, updated), 销毁前/销毁后(beforeDestroy, destroyed)

**beforeCreate**是new Vue()之后触发的第一个钩子，在当前阶段data、methods、computed以及watch上的数据和方法**都不能**被访问。

**created**在实例创建完成后发生，当前阶段已经完成了数据观测，也就是可以使用数据，更改数据，在这里更改数据**不会触发updated**函数。可以做一些初始数据的获取，在当前阶段无法与Dom进行交互，如果非要想，可以通过vm.$nextTick来访问Dom。

**beforeMount**发生在挂载之前，在这之前template模板已导入渲染函数编译。而当前阶段虚拟Dom已经创建完成，即将开始渲染。在此时也可以对数据进行更改，**不会触发updated**。

**mounted**在挂载完成后发生，在当前阶段，真实的Dom挂载完毕，数据完成双向绑定，可以访问到Dom节点，使用$refs属性对Dom进行操作。

**beforeUpdate**发生在更新之前，也就是响应式数据发生更新，虚拟dom重新渲染之前被触发，你可以在当前阶段进行更改数据，不会造成重渲染。

**updated**发生在更新完成之后，当前阶段组件Dom已完成更新。要注意的是**避免在此期间更改数据**，因为这可能会导致无限循环的更新。

**beforeDestroy**发生在实例销毁之前，在当前阶段实例完全可以被使用，我们可以在这时进行善后收尾工作，比如清除计时器，销毁父组件对子组件的重复监听。beforeDestroy(){Bus.$off("saveTheme")}

**destroyed**发生在实例销毁之后，这个时候只剩下了dom空壳。组件已被拆解，数据绑定被卸除，监听被移出，子实例也统统被销毁。

#### 第一次页面加载会触发哪几个钩子
第一次页面加载时会触发 beforeCreate, created, beforeMount, mounted 这几个钩子

#### DOM 渲染在哪个周期中就已经完成
DOM 渲染在 mounted 中就已经完成了。

#### 简单描述每个周期具体适合哪些场景
**beforecreate**: 可以在这加个loading事件，在加载实例时触发 

**created**: 初始化完成时的事件写在这里，如在这结束loading事件，异步请求也适宜在这里调用 

**mounted**: 挂载元素，获取到DOM节点 

**updated**: 如果对数据统一处理，在这里写上相应函数 

**beforeDestroy**: 可以清除定时器，取消bus监听

#### 生命周期的调用顺序
- 组件的调用顺序都是先父后子
- 渲染完成的顺序是先子后父
- 组件的销毁操作是先父后子
- 销毁完成的顺序是先子后父

**加载渲染过程**: 父beforeCreate -> 父created -> 父beforeMount -> 子beforeCreate -> 子created -> 子beforeMount -> 子mounted -> 父mounted

**组件更新过程**: 父beforeUpdate -> 子beforeUpdate -> 子updated -> 父updated

**销毁过程**: 父beforeDestroy -> 子beforeDestroy -> 子destroyed -> 父destroyed

### 4.组件之间如何传值
### 5.vuex介绍一下
是什么：vue框架中状态管理:有五种，分别是 State、 Getter、Mutation 、Action、 Module

使用：新建一个目录store，

场景：单页应用中，组件之间的状态。音乐播放、登录状态、加入购物车
```
vuex的State特性
A、Vuex就是一个仓库，仓库里面放了很多对象。其中state就是数据源存放地，对应于一般Vue对象里面的data
B、state里面存放的数据是响应式的，Vue组件从store中读取数据，若是store中的数据发生改变，依赖这个数据的组件也会发生更新
C、它通过mapState把全局的 state 和 getters 映射到当前组件的 computed 计算属性中

vuex的Getter特性
A、getters 可以对State进行计算操作，它就是Store的计算属性
B、 虽然在组件内也可以做计算属性，但是getters 可以在多组件之间复用
C、 如果一个状态只在一个组件内使用，是可以不用getters

vuex的Mutation特性
改变store中state状态的唯一方法就是提交mutation，就很类似事件。
每个mutation都有一个字符串类型的事件类型和一个回调函数，我们需要改变state的值就要在回调函数中改变。
我们要执行这个回调函数，那么我们需要执行一个相应的调用方法：store.commit。

Action 类似于 mutation，不同在于：Action 提交的是 mutation，而不是直接变更状态；
Action 可以包含任意异步操作，Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，
因此你可以调用 context.commit 提交一个 mutation，
或者通过 context.state 和 context.getters 来获取 state 和 getters。
Action 通过 store.dispatch 方法触发：eg。
store.dispatch('increment')

vuex的module特性
Module其实只是解决了当state中很复杂臃肿的时候，module可以将store分割成模块，
每个模块中拥有自己的state、mutation、action和getter
```
### 6.如何解决首屏加载速度慢的问题
思路： 安装 webpack-bundle-analyzer 插件，分析项目各模块大小，针对大模块进行压缩

（1）路由懒加载

（2）按需引入ui库

（3）开启gzip压缩

（4）关闭sourceMap

### 7.vue-router原理
#### （1）前端路由和后端路由
后端路由：输入url -> 请求发送到服务器 -> 服务器解析请求的路径 -> 拿取对应的页面 -> 返回给浏览器

缺点：对服务器造成大的压力

前端路由：输入url -> js解析地址 -> 找到对应的地址页面 -> 执行对应页面的js -> 看到页面

#### （2）vue-router工作原理
url改变 -> 触发监听事件 -> 改变vue-router里面的current变量 -> vue监视current的监视者 -> 获取到新的组件 -> Render新组件

#### （3）vue-router路由的两种模式

##### ①hash模式
#后面的就是hash的内容

可以通过location.hash拿到

可以通过onhashchange监听hash的改变
##### ②history模式
history即正常路径

可以通过location.pathname拿到

可以用onpopstate监听history的变化

### 8.vue.use干了什么
这是vue.use的源码
```javascript
function initUse (Vue) {
    Vue.use = function (plugin) {
      var installedPlugins = (this._installedPlugins || (this._installedPlugins = []));
      if (installedPlugins.indexOf(plugin) > -1) {
        return this
      }

      // additional parameters
      var args = toArray(arguments, 1);
      // 这里的this是vue的构造函数，使用unshift将其放到第一个，这样插件的install方法里面第一个参数就是vue的构造函数
      args.unshift(this);
      if (typeof plugin.install === 'function') {
        plugin.install.apply(plugin, args);
      } else if (typeof plugin === 'function') {
        plugin.apply(null, args);
      }
      installedPlugins.push(plugin);
      return this
    };
 }
```
从源码可以看出，首先vue.use不会重复注册插件，使用vue.use后，后优先找到插件中的install方法并执行，如果没有找到install方法，则会执行插件本身
