### 1. vuex介绍一下
vuex是专门为vue提供的**全局状态管理系统**，用于多个组件中数据共享、数据缓存等。（**无法持久化**、内部核心原理是通过创造一个全局实例 `new Vue`）
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

### 2. vuex 中 state 的响应式是如何实现的? getters 的 缓存机制 是如何实现的?（关键词：代理）
因为vue中data是响应式的，通过把state代理到vue的data上，就可以实现state的响应式。（在访问state里面的属性时，其实访问的是新实例化的vue实例里面的data）

同理，借用 vue computed 属性的缓存机制，将 getters 上的属性代理到 vue 的computed上面，就可以实现vuex中getters的 缓存机制

```javascript
// 源码如下：

export class Store {
  constructor(options) {
    // state
    // this.state = options.state // 不能这样写，这样写state不是响应式的
    this._vm = new Vue({
      data: {
        state: options.state,
      },
    });
  }
  // 获取state的时候调用get，把vue实例上的data代理过去
  get state() {
    return this._vm.state;
  }
}
```

### 3. vuex里面的mapState、mapMutations等辅助函数的实现原理是什么？（关键词：展开语法，本质上就是调用了this.$store.state.xxx）
利用了 展开语法(...) ，mapState和mapGetters是放到 computed 里面的，并且在里面传入一个数组，mapState和mapGetters函数执行时，将传入的数组遍历，并返回一个对象，对象的每一个属性值都是一个函数，然后利用 展开语法 将返回的对象展开成一个个的函数放到computed里面，以mapState为例，每一个函数里面其实就是调用了this.$store.state.xxx

当然，mapMutations和mapActions也是一样的道理，这两个方法返回的对象会被展开放到methods里面。

```javascript
function mapState(stateArr) {
  let obj = {};
  for (let i = 0; i < stateArr.length; i++) {
    let stateName = stateArr[i];
    obj[stateName] = function() {
      return this.$store.state[stateName];
    };
  }
  return obj;
}

export default {
  computed: {
    ...mapState(["xxx", "yyy"]),
  },
};
```

### 4. vuex里面的state数据在页面刷新之后会变成初始值还是保留更改之后的值，实际工作中一般是如何处理的？（vuex的持久化问题）
首先vuex里面的数据在刷新之后是会变回初始值的，至于是否需要在刷新之后保留更改之后的值，这个要看具体的业务场景。如果需要保留更改，一般有两种方法：
1. 对于一些业务数据来说，页面刷新之后 **重新请求接口** ，然后再把值赋给vuex的state即可
2. 使用 **本地存储** ，在设置state值的时候将新值存放在storage里面，页面刷新的时候，将storage里面的值赋给state，这种方法一般用于存放用户的登录信息。

但是mutations里面一般有很多个方法，可能有很多修改state的操作都需要存储到storage里面，如果在每一个mutations里面都添加存放storage的逻辑，会很麻烦，以后也难维护。

为了解决上面的这个问题，可以使用 vuex 的 **plugins** 语法，给vuex添加插件，在插件里面调用 **store.subscribe** 方法，该方法解释如下：

subscribe(handler: Function): Function

订阅 store 的 mutation。**handler 会在每个 mutation 完成后调用**，接收 mutation 和经过 mutation 后的状态作为参数：
```javascript
store.subscribe((mutation, state) => {
  console.log(mutation.type)
  console.log(mutation.payload)
})
// 要停止订阅，调用此方法返回的函数即可停止订阅。
```

下面展示一个利用vuex plugins 持久化保存state数据的例子：

```javascript
const plugin = () => {
  return store => {
    let localData = JSON.parse(localStorage.getItem('stateData'))
    if(localData) {
      store.replaceState(localData) // replaceState 可以更新vuex里面state的值，这里调用store.commit也可以更新状态
    }
    store.subscribe((mutation, state) => {
      // 这里就可以来判断哪些mutations操作需要持久化
      if(mutation.type === 'LOCAL_DATA') {
        // 对于需要持久化的操作，将最新的state放到storage
        localStorage.setItem('stateData', JSON.stringify(state))
      }
    })
  }
}

// 使用
const store = new Vuex.Store({
  plugins: [plugin()], // Vuex 插件就是一个函数，它接收 store 作为唯一参数：
})
```

### 5. vuex里面用到了哪些设计模式？
#### （1）单例模式
全局只有一个Store类

#### （2）发布-订阅模式
mutations和actions声明使用了发布-订阅模式

#### （3）代理模式
state和getters用到了代理模式，代理到了vue实例的data和computed上，具有响应式和缓存机制的特性

### 6. 手写一个基础的vuex

#### utils/index.js
```javascript
// 封装一个遍历对象的方法
export function foreach(obj, cb) {
  Object.keys(obj).forEach((key) => {
    cb(key, obj[key]);
  });
}
```

#### mixin.js
```javascript
export let Vues;

// install 方法的第一个参数是Vue实例
export const install = function(_Vue) {
  Vues = _Vue;
  Vues.mixin({
    beforeCreate() {
      console.log(this.$options.name);
      let options = this.$options;

      if (options.store) {
        // 给根实例添加$store属性
        this.$store = options.store;
      } else {
        // 给其他组件添加$store属性
        this.$store = this.$parent && this.$parent.$store;
      }
    },
  });
};
```

#### store.js
```javascript
import { foreach } from "./utils/index";
import { Vues } from "./mixin";


// 单例模式
export class Store {
  constructor(options) {

    // getters 计算属性
    let getters = options.getters;
    this.getters = {};
    let computed = {};

    foreach(getters, (key, value) => {
      // 构建computed对象，赋值给 新创建的vue实例 的computed属性上
      computed[key] = () => {
        return value(this.state);
      };
      Object.defineProperty(this.getters, key, {
        get: () => {
          // 这里在获取 gettters 中的属性时，实际上是获取的新创建的vue实例里面的computed属性，这样可以实现 getters 的缓存机制
          return this._vm[key];
        },
      });
    });

    // mutations  actions
    let mutations = options.mutations;
    this.mutations = {};
    foreach(mutations, (key, value) => {
      this.mutations[key] = (data) => {
        value(this.state, data);
      };
    });

    let actions = options.actions;
    this.actions = {};
    foreach(actions, (key, value) => {
      this.actions[key] = (data) => {
        value(this, data);
      };
    });

    // 新创建一个vue实例，把vuex的属性 代理 到vue实例上，这样做可以借用vue的一些特性：data的响应式，computed的缓存机制，让vuex也具备相同的特性
    this._vm = new Vues({
      data: {
        state: options.state,
      },
      computed,
    });
  }
  // 获取state的时候调用get，把vue实例上的data代理过去
  get state() {
    return this._vm.state;
  }
  
  // 发布订阅模式，这里订阅commit和dispatch方法，在需要的地方发布
  commit = (name, data) => {
    this.mutations[name](data);
  };

  dispatch = (name, data) => {
    this.actions[name](data);
  };
}

```
