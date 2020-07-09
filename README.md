- [START](#start)
- [Application Structure(项目结构)](#application-structure项目结构)
- [Core Concepts(核心概念)](#core-concepts核心概念)
  - [1.State](#1state)

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

如[ Application Structure](#application-structure项目结构)





[^Vuex文档]:https://vuex.vuejs.org/zh/guide/structure.html