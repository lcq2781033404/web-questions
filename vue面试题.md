### 1.对MVVM模式的理解
MVVM分为Model、View、ViewModel三者。

Model：代表数据模型，数据和业务逻辑都在Model层中定义；
View：代表UI视图，负责数据的展示；
ViewModel：负责监听Model中数据的改变并且控制视图的更新，处理用户交互操作；
       Model和View并无直接关联，而是通过ViewModel来进行联系的，Model和ViewModel之间有着双向数据绑定的联系。因此当Model中的数据改变时会触发View层的刷新，View中由于用户交互操作而改变的数据也会在Model中同步。

这种模式实现了Model和View的数据自动同步，因此开发者只需要专注对数据的维护操作即可，而不需要自己操作dom。

### 2.vue响应式原理介绍一下（重点关注： 数组 和 对象 类型如何劫持）
Vue 采用**观察者模式**，通过Object.defineProperty将对象的key转换成getter/setter来追踪变化，就是所谓的**数据劫持（只会劫持已经存在的属性）**，Vue中需要收集视图依赖了哪些数据，在getter中收集依赖，在setter中触发依赖（先收集依赖，即把用到该数据的地方收集起来，然后等属性发生变化时，把之前收集好的依赖循环触发一遍）。这样就完成了数据更新，视图随之发生变化的效果，多层对象是通过**递归**来实现劫持。

对于**数组**添加响应式的方法是：通过**劫持数组的方法**来对数组方法进行重写，实现添加响应式

但是defineProperty的getter/setter只能追踪一个数据是否被修改，**无法追踪新增属性和删除属性**。对于要新增的属性，Vue给出了Vue.set的方法（Vue.set方法调用了defineReactive方法将添加的属性变为响应式的）响应式的添加属性。

所以Vue 3.0的时候，Vue是通过Proxy代理的方式监测属性变化的，解决了defineProperty无法追踪新增属性和删除属性的问题，而且Proxy 的代理是**针对整个对象的，而不是对象的某个属性**，因此不同于 Object.defineProperty 的必须遍历对象每个属性。

### 2.上一个问题也可以这么问：vue中如何实现修改数据，视图自动更新的？（关键词：收集依赖，Dep，watcher，Object.defineProperty）
vue中有一个Dep类，负责**收集依赖**，在页面 **初次渲染** 的时候，会把 watcher（监听器） 添加到Dep中。data中每有一个属性，都会创建一个 dep对象 与之对应（data属性和dep对象一一对应），然后通过 Object.definePorperty 劫持data中的数据，并转换成 get 和 set 方法来追踪数据变化，**在 get 中收集依赖（收集依赖调用dep.depend方法）**，**在 set 中触发依赖（触发依赖调用dep.notify方法，dep.notify方法实际上就是遍历了dep下面的watcher，执行更新方法）**，对于**数组**来说，是通过在劫持的数组方法里面触发dep.notify方法，来触发依赖，这样就能实现数据变化，视图自动更新的效果。

### 3.为什么Vue组件中的data是一个函数（而不是一个对象）
vue中的组件是可以**复用**的（也就说是可以共享的），但组件中的**data数据是私有的**，为了防止data数据复用，将其定义为函数，**避免组件与组件之间共享一份data数据**。

原因：

（1）在Vue当中，一个组件被创建好之后是可以被复用的，一个组件它可能在很多个地方被复用（用到），不管这个组件被复用到哪个地方或者是被复用了多少次，那么组件中的data数据都应该是相互隔离、相互独立、互不影响的。

（2）假如我们把组件中的data数据以单纯的对象形式表示的话，那么各个组件实例之间就会共享一份data数据，继而就会导致一个组件上的数据发生改变，其他组件上的数据也会跟着改变。

（3）假如我们把组件中的data写个一个函数并把data中的数据以函数返回值的形式定义，那么每复用一次组件就会返回一份新的data，**相当于给每个组件实例创建了一个私有的数据空间，每个组件实例都有自己的作用域并且相互独立**，各自维护各自的数据，互不影响，这样的话，一个组件上的数据发生改变也不会影响到其他组件上的数据。

### 4.nextTick 的原理是什么？（或者：为什么在nextTick的回调函数里面可以获取到最新的DOM？）
因为nextTick采用了**队列机制**，在数据发生变化的时候，vue框架首先将监听数据变化的**watcher**存入队列，然后调用**nextTick**方法将 **引发多次数据变化的操作 合并为 一次视图渲染操作** ，用户自己写的nextTick里面的回调函数也会进入到nextTick的队列中，不过由于**队列先进先出**的机制，vue框架内部调用的nextTick会先执行完毕，然后再执行用户的nextTick，这样当用户定义的nextTick里面的方法执行的时候，vue框架内部的nextTick已经执行完毕了，所以就能获取到最新的DOM。

### 4.vue初次渲染的流程是什么？
初始化数据  ->  模板编译（3种方式）  ->  生成ast语法树  ->  将ast语法树变成render函数  ->  生成虚拟DOM  ->  将虚拟DOM变成真实DOM  ->  将真实DOM挂载到页面上

### 5.vue 模板编译 有哪几种方式？
有三种：
1. template
2. render
3. el

其中，**根节点中el是必须有的**（没有el有$mount也是一样的效果），否则无法渲染，如果template、render和el都存在，按照render、template、el的优先级顺序来渲染

### 6.虚拟DOM和Diff算法介绍一下
#### （1）虚拟DOM含义
虚拟DOM用js对象模拟真实的DOM节点，该对象描述了真实DOM的结构和属性

#### （2）虚拟DOM的作用
减少了对DOM的操作。页面中的数据和状态变化，都通过Vnode对比（Diff算法），只需要在比对完之后更新DOM，不需要频繁操作，减少了页面回流重绘的次数，提高了页面性能；

#### （3）Diff算法
当数据变化时，vue如何来更新视图的？其实很简单，一开始会根据真实DOM生成虚拟DOM，当虚拟DOM某个节点的数据改变后会生成一个新的Vnode，然后VNode和oldVnode对比，把不同的地方修改在真实DOM上，最后再使得oldVnode的值为Vnode。

diff过程就是调用patch函数，比较新老节点，一边比较一边给真实DOM打补丁(patch)；

#### （4）Diff算法的比较过程（oldVnode和Vnode的递归比较过程）
首先需要明白的是：Diff算法只会比较**同层节点**，如果同层都不一样就没有继续比下去的必要了，直接用新节点替换旧节点

##### ① patch
```javascript
// 代码只保留核心部分
function patch (oldVnode, vnode) {
 // some code
 if (sameVnode(oldVnode, vnode)) {
  patchVnode(oldVnode, vnode)
 } else {
  const oEl = oldVnode.el // 当前oldVnode对应的真实元素节点
  let parentEle = api.parentNode(oEl) // 父元素
  createEle(vnode) // 根据Vnode生成新元素
  if (parentEle !== null) {
   api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl)) // 将新元素添加进父元素
   api.removeChild(parentEle, oldVnode.el) // 移除以前的旧元素节点
   oldVnode = null
  }
 }
 // some code 
 return vnode
}

function sameVnode (a, b) {
 return (
 a.key === b.key && // key值
 a.tag === b.tag && // 标签名
 a.isComment === b.isComment && // 是否为注释节点
 // 是否都定义了data，data包含一些具体信息，例如onclick , style
 isDef(a.data) === isDef(b.data) && 
 sameInputType(a, b) // 当标签是<input>的时候，type必须相同
 )
}
```
首先会进入patch函数，如果判断新旧节点是相同的节点（其实就是判断是不是相同的类型，比如都是div，如果连这点都无法满足就没有继续比下去的必要了），就会进入到下一步patchVnode函数；如果不是相同节点，直接用Vnode替换oldVnode

##### ② patchVnode
```javascript
// 代码只保留核心部分
patchVnode (oldVnode, vnode) {
 const el = vnode.el = oldVnode.el
 let i, oldCh = oldVnode.children, ch = vnode.children
 if (oldVnode === vnode) return
 if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
  api.setTextContent(el, vnode.text)
 }else {
  updateEle(el, vnode, oldVnode)
  if (oldCh && ch && oldCh !== ch) {
   updateChildren(el, oldCh, ch)
  }else if (ch){
   createEle(vnode) //create el's children dom
  }else if (oldCh){
   api.removeChildren(el)
  }
 }
}
```
这个函数做了以下事情：

- a. 找到对应的真实dom，称为 el
- b. 判断 Vnode 和 oldVnode 是否指向同一个对象，如果是，那么直接 return ，否则继续往下判断
- c. 如果他们都有文本节点并且不相等，那么将 el 的文本节点设置为 Vnode 的文本节点。
- d. 如果两者都有子节点，则执行 updateChildren 函数比较子节点，**这一步是整个Diff比较的核心部分，放到下一点着重介绍**
- e. 如果 Vnode 有子节点而 oldVnode 没有有，则将 Vnode 的子节点真实化之后添加到 el 
- f. 如果 oldVnode 有子节点而 Vnode 没有，则删除 el 的子节点

##### ③ updateChildren
```javascript
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    const canMove = !removeOnly

    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(newCh)
    }

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } else if (sameVnode(oldStartVnode, newStartVnode)) { // 新前-旧前
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) { // 新后-旧后
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right // 新后-旧前
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left // 新前-旧后
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }
  }
```
这里是整个Diff算法最核心的部分，进到了这个函数，就说明Vnode和oldVnode都有子节点，这个函数就是对比两者子节点并更新DOM的。先说一下这个函数做了什么：

- a. 将 Vnode 的子节点 Vch 和 oldVnode 的子节点 oldCh 提取出来
- b. oldCh 和 vCh 各有两个头尾的变量 StartIdx 和 EndIdx ，它们的2个变量相互比较，一共有4种比较方式，这四种比较方式的先后顺序是：新前旧前、新后旧后、新后旧前、新前旧后，只要有一个匹配上就不会再继续往下匹配了，而且当匹配成功之后，会递归调用patchVnode，并且匹配到的头尾索引会往中间移动（比如新前旧前匹配到了：startIdx++, oldStartIdx++；比如新后旧前匹配到了：endIdx--, oldStartIdx++）。
- c. 不仅如此，如果是**新后旧前**匹配上了，那么**真实dom中旧前对应的节点会移到新后对应的索引上，同时匹配到的索引依旧会向中间移动**
- d. 同理，如果是**新前旧后**匹配上了，那么**真实dom中旧后对应的节点会移到新前对应的索引上，同时匹配到的索引依旧会向中间移动**
- e. 如果四种匹配没有一对是成功的，那么遍历 oldChild ， **新前节点** 挨个和他们匹配，匹配成功就在真实dom中将成功的节点移到最前面，如果依旧没有成功的，那么将 新前 对应的节点 插入到dom中对应的 旧前 位置， 旧前 和 新前 指针向中间移动。
- f. 这个匹配过程的结束有两个条件：
    - f1. oldS > oldE 表示 oldCh 先遍历完，那么就将多余的 vCh 根据index添加到dom中去 
    - f2. S > E 表示vCh先遍历完，那么就在真实dom中将区间为 [oldS, oldE] 的多余节点删掉

##### ④ 所以这里回归到可能会问的一个问题：为什么v-for需要加key？
v-for中的key来给每个节点做一个唯一的标识，diff算法就可以正确的识别此节点，找到正确的位置区插入新的节点；key的主要作用是为了高效的更新虚拟dom

#### （5）总结
关于这个问题，总结起来就是：
1. Diff算法只会比较**同层节点**，一边比较一边给**真实DOM**打补丁
2. 首先调用 **patch** 函数，判断新旧节点是否是相同的节点，如果不是，直接用新节点替换旧节点；如果是，则进入下一步 **patchVnode** 函数
3. 进入 **patchVnode** 函数，依次判断：
    - 新旧节点是否指向同一个对象？如果是，直接返回
    - 新旧节点是否都有文本？如果有且不相同，在**真实DOM中**用新节点文本替换旧节点文本
    - 新旧节点是否都有子节点？如果都有，则进入下一步 **updateChildren** 函数（**Diff算法核心，要着重介绍**）
    - 新节点有子节点，旧节点无子节点？如果满足这种情况，直接在**真实DOM中**添加新节点的子节点
    - 新节点无子节点，旧节点有子节点？如果满足这种情况，删除**真实DOM中**对应的子节点
4. 进入 **updateChildren** 函数，将新节点和旧节点的子节点列表分别**做标记**：新节点子节点的第一个标记为**新前**，最后一个标记为**新后**；旧节点子节点的第一个标记为**旧前**，最后一个标记为**旧后**。
5. Diff算法采用**双指针**的方法，有**四种命中方式**，先后顺序分别为：**新前旧前**，**新后旧后**，**新后旧前**，**新前旧后**，只要有一个匹配上，就不会继续往下匹配了，匹配完成后，会**递归调用patchNode，检查子节点的子节点**，并且匹配到的头尾索引会往中间移动。
6. 如果**四种命中方式**都没有匹配成功，那么**遍历旧节点的子节点**， **新前节点** 挨个和他们匹配，匹配成功就在**真实DOM中**将成功的节点移到**最前面**，如果依旧没有成功的，那么将**新前**对应的节点**插入到DOM中**对应的**旧前**位置， **旧前**和**新前**指针向**中间移动**。
7. 这个匹配过程的结束有两个条件：
    - （1）**旧前**索引 > **旧后**索引，表示 **旧节点的子节点** 先遍历完，那么就将多余的 **新节点子节点** 根据index添加到dom中去 
    - （2）**新前**索引 > **新后**索引，表示 **新节点的子节点** 先遍历完，那么就在真实dom中将区间为 [**旧前**, **旧后**] 的多余节点删掉
  

### 7.为什么vue中最好给每个节点指定一个key，并且key最好不是索引
因为在**diff算法**中，会**使用节点的key来判断两个节点是不是相同的节点**，如果节点有key并且两个节点key相同，就会进行diff的四种命中方式判断，这样可以提升节点更新效率；反之如果节点没有设置key，diff算法就会判断两个节点不是相同的节点，然后直接用新节点去替换旧节点，这样就造成了性能浪费。

而且最好不要使用 索引 作为key值，因为索引值是会随着数据的位置经常变动的，很可能发生节点其实没变，但是新旧节点的索引值不一样了，这样就导致在diff算法中会认为这是两个不一样的节点（但是其实是一样的），然后用新节点去替换旧节点，同样造成了性能的浪费。

### 8.vue生命周期介绍一下
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

### 9.vue生命周期的原理是什么？（或者：vue的生命周期是如何实现的，为什么在mounted中可以获取到DOM？）（关键词：vue.mixin，发布-订阅模式）
vue的生命周期使用了 **发布-订阅** 的模式，首先使用 **vue.mixin** 方法将vue实例的配置项混入，这个配置项就包含了每个生命周期的定义，由于生命周期数量比较多，所以vue使用了 **策略模式** 将所有生命周期方法集中管理，统一注册。

然后在**vue渲染**的过程中，**在各个阶段 触发生命周期方法** ，比如在初始化数据之前会触发beforeCreate，初始化完成之后触发created方法，在挂载DOM之前触发beforeMount，在DOM挂载完成后触发mounted，等等，直到vue的实例销毁为止

### 10.watch和computed有什么区别和联系？（关键词：watcher类，computed缓存）
**相同点**

 watch 和 computed 在vue内部都是通过**watcher类**来实现的，因为vue本身具有**响应式**的特性，watch和computed都是通过watcher类来监听数据的变化。

**区别**

- computed具有**缓存机制**，内部是通过一个变量（dirty）来实现的，只有当这个变量为true的时候才会执行computed里面的方法，当获取computed属性值的时候，将dirty置为false，这样如果computed依赖的值不发生改变，即使重复获取这个值，也不会再调用computed里面的方法，直到computed依赖的值发生改变，这时将dirty置为true，就会获取到最新的computed的值；而watch是没有缓存的，数据变化，会直接触发watche监听
- computed里面不能写异步，watch可以写异步（因为vue会把computed里面写的代码处理成为一个个的**属性**，属性是不能写异步的，而watch里面写的代码会处理成为方法，方法里面可以写异步）

### 11.vue为什么是组件化开发，有什么好处？（关键词：性能可以得到优化，复用方便，好维护）
采用组件化开发的好处就是在页面更新的时候可以组件**局部化更新**，不必整个页面更新，这样更有利于**提高页面性能**（提高性能最关键），同时，对于开发人员来说，采用组件化的开发可以提高组件的复用性，以后维护起来也更方便

### 8.组件之间如何传值
### 9.vuex介绍一下
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
### 10.如何解决首屏加载速度慢的问题
思路： 安装 webpack-bundle-analyzer 插件，分析项目各模块大小，针对大模块进行压缩

（1）路由懒加载

（2）按需引入ui库

（3）开启gzip压缩

（4）关闭sourceMap

### 11.vue-router原理
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
从源码可以看出：
1. 首先vue.use不会重复注册插件，
2. 使用vue.use后，会优先找到插件对象中的install方法并执行，如果没有找到install方法，则会执行插件本身。（这里的插件就是一个方法或者对象）
3. 在插件的install方法（或者插件本身）中，**第一个参数是vue实例**
