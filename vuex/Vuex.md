## vuex
### 为什么使用vuex：
- 在我们构建一些小型应用的时候，组件之间的通信，状态管理较为简单，父子组件之间的通信可以使用props来传递:
1. 如果子组件需要向父组件传递数据，
可以通过$on将父组件的事件绑定到子组件

2. 在子组件中通过$emit来触发$on绑定的父组件事件

- 非父子组件通信也使用以上方式，如下：

```
var bus = new Vue()

// 触发组件 A 中的事件
bus.$emit('id-selected', 1)

// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {  
  // ...
})
```
- 但是当你的项目比较大的时候，会出现多个视图组件依赖同一个状态，来自不同视图的行为需要变更同一个状态，props传递的链路过长时，万一数据发生了错误，定位问题将是一件很麻烦的事。遇到以上情况时候，你就应该考虑使用Vuex了。
- Vuex一个专为Vue.js应用程序开发的状态管理模式，使用单一组件树来管理共用状态，无论哪个位置的状态需要改变，都会获取状态并发出行为。
- 简单来说就是：能把组件的==共享状态==抽取出来，当做一个==全局==单例模式进行管理。这样不管你在何处改变状态，都会通知使用该状态的组件做出相应修改


### Vuex组成：
Vuex分成四个部分：

1. State：单一状态树
2. Getters：状态获取
3. Mutations：触发同步事件
4. Actions：提交mutation，可以包含异步操作
>  Vuex的数据总是“单向流动”，用户访问页面并触发action，action提交mutation事件，mutation事件更改state状态
state状态改变后更新页面(vue comptents)

#### （一）State

之所以叫单一状态树，就是因为用一个对象包含了全部的应用层级状态。在Vue组件中如果想要获取Vuex的状态，都需要从state中获取。最简单的方式就是在计算属性computed中返回state的某个状态。

- Demo
```
// 创建一个 Counter 组件
const Counter = {  
  template: `
{{ count }}
`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
如果你想在每个子组件中都调用state，可以在根组件中注册store选项，vuex就会提供了一种机制将状态从根组件『注入』到每一个子组件中（需调用 Vue.use(Vuex)）。

const app = new Vue({  
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store
});

/*改为：*/
/*子组件*/
const Counter = {  
  template: `
{{ count }}
`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```
- mapState辅助函数

在Vuex 2中有mapState辅助函数，当一个组件需要获取多个状态时，可以直接在函数中声明。

```
import { mapState } from 'vuex'

export default {  
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
当计算属性与state子节点相同时，可以这么写（我习惯这么写）：

computed: mapState([  
  // 映射 this.count 为 store.state.count
  'count'
])
```

#### （二）Getters
当state中的某些状态在各个组件中都被频繁使用，如果在每个组件中都声明一次，将会变得非常繁琐。因此便有了getters，可以把它看做Vuex的计算属性。
- getters接受两个参数：state与getters，我们可以在store中定义getters：

```
const store = new Vuex.Store({  
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)    //{ id: 1, text: '...', done: true }
    }
  }
})

store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
可以在组件中轻松使用：

computed: {  
  doneTodos () {
    return this.$store.getters.doneTodos
  }
}
```
- mapGetters 辅助函数

同样Vuex 2中提供了一个mapGetters辅助函数用于有多个状态需要获取的情况

```
import { mapGetters } from 'vuex'

export default {  
  computed: {
  // 使用对象展开运算符将 getters 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
    ])
  }
}

如果需要重命名，可以这样写：
mapGetters({  
  // 映射 this.doneCount 为 store.getters.doneTodosCount
  doneCount: 'doneTodosCount'
})
```

#### （三）Mutations
- 更改 Vuex的store中的状态的==唯一==方法是==提交== mutation。
- mutation不能直接调用，而要通过相应的 type 调用相应的store.commit方法：

> store.commit('increment')  
- mutation接受两个参数state和payload(载荷)。

- 通过执行回调函数修改state的状态，可以向store.commit传入额外的参数payload，payload可以是一个对象，这样可以包含多个字段并且记录的 mutation 会更易读。


```
mutations: {  
  increment (state, payload) {
    state.count += payload.amount
  }
}

store.commit('increment', {  
  amount: 10
})
```
> 遵守Vue的响应规则
Vuex中的store是响应式的，因此当要在对象上添加新属性时，应该使用Vue.set()方法来监听，否则state的状态无法实现自动更新。
> 
>  Vue.set(obj, 'newProp', 123)  
为了避免这样的情况出现，最好在store中提前初始化好所有需要使用的属性。

```
this.$store.commit(xxxx , {aaa: 'aaa', bbb: 'bbb'})

const mutations = {
    [xxxx] (state, {aaa, bbb}) {
        console.log(aaa, bbb)
    }
}
```

- mutation必须是==同步==函数

```
mutations: {  
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```
- mapMutations
```
// 在组件中提交Mutations
import { mapMutations } from 'vuex'

export default {  
  methods: {
    ...mapMutations([
      'increment' // 映射 this.increment() 为 this.$store.commit('increment')
    ]),
    ...mapMutations({
      add: 'increment' // 映射 this.add() 为 this.$store.commit('increment')
    })
  }
}
```
#### (四) Actions
Action相对于mutation，有以下不同：

- Action 提交的是mutation，而不是直接变更状态。

```
const store = new Vuex.Store({  
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```
- Action 可以包含任意==异步==操作
```
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```
axios（异步）相关代码写在actions中，在.vue文件中来dispatch触发actions。
- Action 函数接受一个与 store 实例具有相同方法和属性的context对象。
- action的参数也可以是：
```
({commit,state},params)
```
- 除了使用context.commit提交mutations，也可以使用context.getter和context.state来获取getter和state。有一点需要注意点是，context并不是state实例本身。

- 在使用commit时，常使用参数结构的方式来简化代码：

```
actions: {  
  increment ({ commit }) {
    commit('increment')
  }
}
```
- 分发Action
```
store.dispatch('increment') 
//dispatch最多就只接受两个参数，type和payload
```
为什么要使用action而不是直接分发mutation？
> 因为mutation必须执行同步函数，而在Action中可以执行异步函数。与mutation类似，Actions支持同样的载荷(payload)方式和对象方式进行分发：

```
// 以载荷形式分发
store.dispatch('incrementAsync', {  
  amount: 10
})
```

```
// 以对象形式分发
store.dispatch({  
  type: 'incrementAsync',
  amount: 10
})
在组件中可以使用以下方式分发Action

import { mapActions } from 'vuex'

export default {  
  // ...
  methods: {
    ...mapActions([
      'increment' // 映射 this.increment() 为 this.$store.dispatch('increment')
    ]),
    ...mapActions({
      add: 'increment' // 映射 this.add() 为 this.$store.dispatch('increment')
    })
  }
}
```
#### （五）Module
就是模块化，因为随着后面的业务逻辑的增多，把Vuex分模块的开发会使得代码更加简洁清晰明了。如果Module的名字叫search，每次在子组件中都需要将：this.$store.state.data改为this.$store.state.search.data

#### 我刚学的时候看到的入门点博客：
1. 入门：
> https://yeaseonzhang.github.io/2017/03/16/Vuex-%E9%80%9A%E4%BF%97%E7%89%88/
> https://segmentfault.com/a/1190000008861913
2. 模块化：
> https://segmentfault.com/a/1190000007667542
> https://segmentfault.com/a/1190000012083258