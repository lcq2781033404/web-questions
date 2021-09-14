### 1. vue-router 有几种模式，原理是什么？
#### （1）hash模式
- url路径会出现“#”号字符
- hash值不包括在Http请求中，它是交由前端路由处理，所以改变hash值时不会刷新页面，也不会向服务器发送请求，所以这也是单页面应用的必备。
- hash值的改变会触发hashchange事件
- 当我们进行刷新操作，或者直接输入浏览器地址时，hash路由会加载到地址栏对应的页面，而history路由一般就404报错了（刷新是网络请求，没有后端准备时会报错）。
#### （2）history模式
- history运用了浏览器的历史记录栈，之前有back,forward,go方法，之后在HTML5中新增了pushState（）和replaceState（）方法，它们提供了对历史记录进行修改的功能，不过在进行修改时，虽然改变了当前的URL，但是浏览器不会马上向后端发送请求。
- history的这种模式需要后台配置支持：**要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面**。比如：当我们进行项目的主页的时候，一切正常，可以访问，但是当我们刷新页面或者直接访问路径的时候就会返回404，那是因为在history模式下，只是动态的通过js操作window.history来改变浏览器地址栏里的路径，并没有发起http请求，但是当我直接在浏览器里输入这个地址的时候，就一定要对服务器发起http请求，**因为vue-router设置的路径不是真实存在的路径**，这个目标在服务器上不存在，就会返回404。
- 所以，当vue-router使用history模式，部署时要注意：**服务器的404页面需要重定向到index.html**
- hash路由支持低版本的浏览器，而history路由是HTML5新增的API。

### 2. vue-router如何响应路由参数的变化？
有两种方法：
#### （1）使用watch监听 $route 对象
```javascript
watch: {
  $route(to, from) {
    // 对路由变化作出响应...
  }
}
```

#### （2）使用 2.2 中引入的 beforeRouteUpdate 导航守卫
使用beforeRouteUpdate别忘了调用next方法
```javascript
beforeRouteUpdate(to, from, next) {
  // react to route changes...
  // don't forget to call next()
}
```

### 3. 如何将路由与组件解耦？（或者：在组件里面直接使用this.$route.params会使组件与路由耦合度变高，如何解耦？）
在组件中使用 $route 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性。

可以在routes配置项中使用 props 属性将组件和路由解耦：
```javascript
// 如果 props 被设置为 true，route.params 将会被设置为组件属性。
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { 
      path: '/user/:id', 
      component: User, 
      props: true 
    }
  ]
})
```

### 4. vue-router传参方式有哪些，有什么区别？
#### （1）query方式传参
通过query方式传参，参数会以?key=value的形式显示在url中，并且刷新页面参数还在
```javascript
this.$router.push({
  name: 'user',
  query: {
    id: '123',
    name: 'user'
  }
})
```
在跳转过去的页面通过 this.route.query 来获取id和name，但是为了解耦，一般使用props来传参
```javascript
// router 配置
const router = new VueRouter({
  routes: [
    {
      path: '/user',
      name: 'user',
      component: User,
      props: route => ({ id: route.query.id, name: route.query.name })
    }
  ]
})

// User组件
const User = {
  props: ['id', 'name'],
  template: '<div>User {{ id }} {{ name }}</div>'
}
```

#### （2）params方式传参
通过params方式传参，**可能会出现刷新页面参数丢失的情况**，这是因为在配置routes的时候，没有在path上配置占位符，就会导致参数没有显示到url上面
```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id', // 这里配置的要和你传递的参数名保持一致
      name: 'user',
      component: User,
      props: true // props 被设置为 true，route.params 将会被设置为组件属性
    }
  ]
})
```
经过上面的 /:id 的这种配置，跳转的时候会把id拼接到url的后面（/123），这样在刷新页面的时候参数也不会丢失。

为了解耦，User组件依旧通过 props 获取路由参数

### 5. vue-router的导航守卫有哪些钩子函数？导航解析流程是什么？
路由导航守卫的钩子函数分为三个维度：全局的、单个路由独享的、组件级的。
### 全局守卫
#### （1）全局前置守卫(router.beforeEach)
你可以使用 router.beforeEach 注册一个全局前置守卫：
```javascript
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
  // 确保 next 函数在任何给定的导航守卫中都被严格调用一次！！！
})
```

#### （2）全局解析守卫(router.beforeResolve)
在 2.5.0+ 你可以用 router.beforeResolve 注册一个全局守卫。这和 router.beforeEach 类似，区别是在**导航被确认**之前，同时在**所有组件内守卫和异步路由组件被解析**之后，解析守卫就被调用。

#### （3）全局后置钩子(router.afterEach)
你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 next 函数也**不会改变导航本身**
```javascript
router.afterEach((to, from) => {
  // ...
})
```
### 路由独享守卫
可以在路由配置上直接定义 beforeEnter 守卫：
```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

### 组件内的守卫
#### （1）beforeRouteEnter
```javascript
const Foo = {
  template: `...`,
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  }
}
```
beforeRouteEnter 守卫**不能 访问 this**，因为守卫在导航确认前被调用，因此即将登场的新组件还没被创建。

不过，你可以通过传一个回调给 next来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数。
```javascript
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
    // beforeRouteEnter 是支持给 next 传递回调的 唯一 守卫。对于 beforeRouteUpdate 和 beforeRouteLeave 来说，this 已经可用了，所以不支持传递回调，因为没有必要了。
  })
}
```

#### （2）beforeRouteUpdate (2.2 新增)
```javascript
const Foo = {
  template: `...`,
  beforeRouteUpdate(to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  }
}
```
beforeRouteUpdate这个钩子函数可以用来**监听路由参数变化**

#### （3）beforeRouteLeave
```javascript
const Foo = {
  template: `...`,
  beforeRouteLeave(to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```
beforeRouteLeave通常用来禁止用户在还未保存修改前突然离开。该导航可以通过 next(false) 来取消。
```javascript
beforeRouteLeave (to, from, next) {
  const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
  if (answer) {
    next()
  } else {
    next(false)
  }
}
```

### 完整的导航解析流程如下：
1. 导航被触发。
2. 在失活的组件里调用 **beforeRouteLeave** 守卫。
3. 调用全局的 **beforeEach** 守卫。
4. 在重用的组件里调用 **beforeRouteUpdate** 守卫 (2.2+)。
5. 在**路由配置**里调用 **beforeEnter**。
6. 解析**异步路由组件**。
7. 在被激活的组件里调用 **beforeRouteEnter**。
8. 调用全局的 **beforeResolve** 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 **afterEach** 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的**回调函数**，创建好的组件实例会作为回调函数的参数传入。
