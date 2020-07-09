- [START](#start)
- [Application Structure(项目结构)](#application-structure项目结构)
- [Core Concepts(核心概念)](#core-concepts核心概念)
  - [1.State](#1state)
  - [Getter](#getter)
- [通过方法访问](#通过方法访问)
  - [Mutation](#mutation)

### START

```html
<script src="https://unpkg.com/vuex@3.4.0/dist/vuex.js"></script>  <!--CDN引入 -->
```

或

```js
import Vuex from 'vuex' // Vue/cli开发
Vue.use(Vuex);
```

`Vuex`对象的内容如下:

![image-20200709212816608](README.assets/image-20200709212816608.png)

1. Vuex 的状态存储是响应式的。当 Vue 组件从 `store` 中读取状态的时候，若 `store `中的状态发生变化，那么相应的组件也会相应地得到高效更新。

2. 你不能直接改变` store `中的状态。改变 `store` 中的状态的唯一途径就是显式地**提交 (commit)` mutation`**。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。我们通过提交 `mutation `的方式，而非直接改变 `store.state.count`，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。

   由于 `store `中的状态是响应式的，在组件中调用 `store `中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 `methods `中提交` mutation`。

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

store.commit('increment')
console.log(store.state.count) // -> 1
```

为了在 Vue 组件中访问 `this.$store` property，你需要为 Vue 实例提供创建好的 store。Vuex 提供了一个从根组件向所有子组件，以 `store` 选项的方式“注入”该 store 的机制：

```js
new Vue({
    el: '#app',
    store,//=store:store ES6
    methods: {
    	increment() {increment
    	this.$store.commit('increment') //调用this.$store._mutations.increment
    	console.log(this.$store.state.count)
    	}
    },
    computed:{
        count(){
            return this.$store.state.count; //用计算属性返回this.$store.state.count
        }
    }
})
```

`store`对象的内容如下:

![image-20200709212839841](README.assets/image-20200709212839841.png)

------

### Application Structure(项目结构)

Vuex 并不限制你的代码结构。但是，它规定了一些需要遵守的规则：

1. 应用层级的状态应该集中到单个**` store `**对象中。
2. 提交 **`mutation`** 是更改状态的唯一方法，并且这个过程是同步的。
3. 异步逻辑都应该封装到 **`action`** 里面。

只要你遵守以上规则，如何组织代码随你便。如果你的 **`store`** 文件太大，只需将 `action`、`mutation` 和 `getter` 分割到单独的文件。

对于大型应用，我们会希望把 `Vuex` 相关代码分割到模块中。下面是项目结构示例：

```bash
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```

请参考[购物车示例](https://github.com/vuejs/vuex/tree/dev/examples/shopping-cart)。[^Vuex文档]

------

### Core Concepts(核心概念)

#### 1.State

如[ Application Structure](#application-structure项目结构)所示,Vuex 使用**单一状态树**，用一个对象就包含了全部的应用层级状态。至此它便作为一个“唯一数据源 ([SSOT](https://en.wikipedia.org/wiki/Single_source_of_truth))”而存在。这也意味着，每个应用将仅仅包含一个 `store `实例。单一状态树让我们能够直接地定位任一特定的状态片段，在调试的过程中也能轻易地取得整个当前应用状态的快照。单状态树和模块化并不冲突,存储在 Vuex 中的数据和 Vue 实例中的 `data` 遵循相同的规则，例如状态对象必须是纯粹 (plain) 的。**参考：**[Vue#data](https://cn.vuejs.org/v2/api/#data)。

Vuex 通过 `store` 选项，提供了一种机制将状态从根组件“注入”到每一个子组件中（需调用 `Vue.use(Vuex)`）：

```js
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})
```

通过在根实例中注册 `store` 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 `this.$store` 访问到。让我们更新下 `Counter` 的实现：

```js
const Counter = {
	template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

**`mapState` 辅助函数**

当一个组件需要获取多个状态的时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性:

```js
let { mapState } = Vuex;//解构Vuex中的mapState方法 ES6

state: {
  count: 0,
  todos: [
    { id: 1, text: '...', done: true },
    { id: 2, text: '...', done: false }
  ],
}
```

```js
1.
computed: mapState({
  // 箭头函数可使代码更简练
  count: state => state.count,
  // 传字符串参数 'count' 等同于 `state => state.count` countAlias = count
  countAlias: 'count',
  count2: 'count',
  // 为了能够使用 `this` 获取局部状态，必须使用常规函数
  countPlusLocalState(state) {
      return state.count + this.localCount
  }
}) // mapState返回的是一个对象,将该对象赋值给computed

2.
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count','todos'
]),
```

我们如何将它与局部计算属性混合使用呢？通常，我们需要使用一个工具函数将多个对象合并为一个，以使我们可以将最终对象传给 `computed` 属性。但是自从有了[对象展开运算符](https://github.com/tc39/proposal-object-rest-spread)，我们可以极大地简化写法:

```js
data() {
  return {
      localCount: 4
  }
},
computed:{
  countPlusLocalState() {
      return this.localCount;
  },
  ...mapState([
      // 映射 this.count 为 store.state.count
      'count',
      "todos"
  ]),
}
```

使用 Vuex 并不意味着你需要将**所有的**状态放入 Vuex。虽然将所有的状态放到 Vuex 会使状态变化更显式和易调试，但也会使代码变得冗长和不直观。如果有些状态严格属于单个组件，最好还是作为组件的局部状态。你应该根据你的应用开发需要进行权衡和确定。

#### Getter

有时候我们需要从 store 中的 state 中派生出一些状态，例如对列表进行过滤并计数：

```js
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

如果有多个组件需要用到此属性，我们要么复制这个函数，或者抽取到一个共享函数然后在多处导入它——无论哪种方式都不是很理想。

`Vuex` 允许我们在 `store` 中定义“`getter`”（可以认为是 `store` 的计算属性）。就像计算属性一样，`getter` 的返回值会根据它的依赖被**缓存**起来，且只有当它的**依赖值**发生了改变才会被**重新计算**。

`Getter `接受` state` 作为其第一个参数：

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

**通过属性访问**

`Getter `会暴露为 `store.getters` 对象，你可以以属性的形式访问这些值：

```js
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```

`Getter `也可以接受其他`getter` 作为第二个参数：

```js
getters: {
  doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    },
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}
```

```js
store.getters.doneTodosCount // -> 1
```

![image-20200709223331751](README.assets/image-20200709223331751.png)

我们可以很容易地在任何组件中使用它：

```js
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

### 通过方法访问

你也可以通过让 getter 返回一个函数，来实现给 getter 传参。在你对 store 里的数组进行查询时非常有用。

```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

```js
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

```js
getTodoById: (state) => (id) => {
  return state.todos.find(todo => todo.id === id)
}
等价于
getTodoById(state){
  return function(id){
    return state.todos.find(todo => todo.id === id)
  }
}
//store.getters.getTodoById的返回值为
//function(id){
//  return state.todos.find(todo => todo.id === id)
//}
let getTodoById = store.getters.getTodoById
getTodoById(2)
//store.getters.getTodoById(2)
```

**注意**，`getter` 在通过方法访问时，每次都会去进行调用，而不会缓存结果。

**`mapGetters` 辅助函数**

`mapGetters` 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性：

```js
let {mapGetters} Vuex;//解构Vuex中的mapGetters方法

data() {
  return {
      localCount: 4
  }
},
computed:{
 //...
  ...mapGetters(["doneTodos", "doneTodosCount"]),
}

```

如果你想将一个 getter 属性另取一个名字，使用对象形式：

```js
computed:{
 //...
  ...mapGetters({ getID: "getTodoById" }),
}
```

#### Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 `mutation`。Vuex 中的 `mutation` 非常类似于事件：每个 `mutation` 都有一个字符串的 **事件类型 (`type`)** 和 一个 **回调函数 (`handler`)**。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 `state `作为第一个参数[见start](#Start)

**提交载荷（Payload）**

```js
mutations:{
  //...
  incrementN(state, n) {
    console.log(`n的值为:${n}`);
    n += state.count;
    console.log(`n的值为:${n}`);
    console.log(`state.count+n的值:${n}`);
	},
}

store.commit('incrementN', 10)
```

在大多数情况下，载荷应该是一个对象，这样可以包含多个字段并且记录的 mutation 会更易读：

```js
mutations:{
  //...
	incrementPL(state, payLoad) {
    console.log(`state.count的值为:${state.count}`);
    console.log(`payLoad.n的值为:${payLoad.n}`);
    payLoad.n += state.count;
    console.log(`payLoad.n的值:${payLoad.n}`);
	},
}  n

store.commit('inPLcrementPL', {
  n:10
})
```

**对象风格的提交方式**

提交 mutation 的另一种方式是直接使用包含 `type` 属性的对象：

```js
store.commit({
  type: 'incrementPL',
  n: 10
})
```

当使用对象风格的提交方式，整个对象都作为载荷传给 mutation 函数，因此 handler 保持不变.

**Mutation 需遵守 Vue 的响应规则**

既然 Vuex 的 store 中的状态是响应式的，那么当我们变更状态时，监视状态的 Vue 组件也会自动更新。这也意味着 Vuex 中的 mutation 也需要与使用 Vue 一样遵守一些注意事项：

1. 最好提前在你的 store 中初始化好所有所需属性。
2. 当需要在对象上添加新属性时，你应该

- 使用 `Vue.set(obj, 'newProp', 123)`, 或者
- 以新对象替换老对象。例如，利用[对象展开运算符](https://github.com/tc39/proposal-object-rest-spread)我们可以这样写:

- ```js
  state.obj = { ...state.obj, newProp: 123 }
  ```

**使用常量替代 Mutation 事件类型**

使用常量替代 mutation 事件类型在各种 Flux 实现中是很常见的模式。这样可以使 linter 之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的代码合作者对整个 app 包含的 mutation 一目了然:

```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```



```js
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015(ES6)风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

**Mutation 必须是同步函数**

一条重要的原则就是要记住 **mutation 必须是同步函数**。为什么？请参考下面的例子：

```js
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

现在想象，我们正在 debug 一个 app 并且观察 devtool 中的 mutation 日志。每一条 mutation 被记录，devtools 都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中 mutation 中的异步函数中的回调让这不可能完成：因为当 mutation 触发的时候，回调函数还没有被调用，devtools 不知道什么时候回调函数实际上被调用——实质上任何在回调函数中进行的状态的改变都是不可追踪的。

```js
import { mapMutations } from 'vuex'
//let { maptations } = Vuex;

methods:{
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementN' // 将 `this.incrementN(n)` 映射为 `this.$store.commit('incrementN', n)`
    ]),
    ...mapMutations({
      add: 'incrementPL' // 将 `this.add()` 映射为 `this.$store.commit('incrementPL',payLoad)`
    })
  }
}
```



[^Vuex文档]:https://vuex.vuejs.org/zh/guide/structure.html