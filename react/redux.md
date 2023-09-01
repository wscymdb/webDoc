# 1.Redux的核心

## 1.1.State

* 用来存数据的
* 但是数据源并不能直接写进去，而是要通过某种方式存入
* 详情看下面案例

## 1.2.action

**Redux要求通过action来更新数据**

* 所有的数据变化，必须`通过派发(dispatch)action来更新`
* action就是`一个普通的js对象`，用来描述此次更新的type和content
* `必须要有type字段`

```js
const action1 = {type:'add_name',info:{name:'hhh'}}
const action2 = {type:'change_age',age:18}
...
```

## 1.3.reducer

* reducer是一个纯函数
* reducer做的事情就是`将传入的state和action结合起来生成一个新的state`

# 2.Redux的基本使用

## 2.1.创建store

* Redux是不能直接在store中初始化数据的
* 需要借助reducer来完成初始化数据，以及更新数据的

```js
const { createStore } = require('redux')

// 1.初始化数据
const initialState = {
  name: 'zs',
  level: 199,
}

// 2.定义一个reducer函数：纯函数
function reducer(state = initialState, action) {

  return state
}

// 3.创建store 传入reducer
const store = createStore(reducer)

module.exports = store

```

## 2.2.使用store中的数据

* 首先引入创建好的store
* 然后使用`getState()`方法来获取数据

```js
const store = require('./store')

console.log(store.getState())

```

## 2.3.修改store中的数据

* 首先需要引入创建好的store
* 其次通过dispatch方法派发一个action
* 然后在store的reducer中通过判断action的类型 更改state中的数据

```js
const store = require('./store')
console.log(store.getState())
const colorAction = { type: 'change_name', name: '历史' }
store.dispatch(colorAction)
console.log(store.getState())
```



store.js

```js
const { createStore } = require('redux')

// 1.初始化数据
const initialState = {
  name: 'zs',
  level: 199,
}

// 2.定义一个reducer函数：纯函数
//  两个参数
// 参数一state： store中目前保存的state
// 参数二action：本次需要更新的action(dispatch传入的action)
// 返回值：她的返回值会作为store的state
function reducer(state = initialState, action) {
  // 更新数据 返回更新后的state
  // 使用switch 比if判断更加直观
  switch (action.type) {
    case 'change_name':
      return { ...state, name: action.name }
  }

  // 没有更新数据 返回之前的state
  return state
}

const store = createStore(reducer)
module.exports = store

```

## 2.4.订阅store中的数据

* 当数据发生了变化会自动执行订阅的函数

```js
const store = require('./store')

// 数据发生变化 自定执行回调函数
// subscribe返回一个函数 调用就表示取消订阅
const unsubscribe = store.subscribe(() => {
  console.log('订阅数据的变化：', store.getState())
})

store.dispatch({ type: 'change_name', name: '历史' })
// 取消订阅
unsubscribe()
store.dispatch({ type: 'change_name', name: '你好' })
```

## 2.5.动态生成action

* 这是对派发action的一种优化写法
* 比如要修改过10次name 那么需要派发10次action，每次都要手动写一个action 很麻烦
* 所以可以使用函数 动态的生成action

```js
const store = require('./store')

// actionCreator :创建action
// 如果要多次修改name，每次都要写 一个对象 太过于麻烦
// 所以可以创建一个函数动态的生成修改name的action
const changeNameAction = (name) => ({ type: 'change_name', name })

store.dispatch(changeNameAction('里斯'))
store.dispatch(changeNameAction('王武'))
```

# 3.Redux的三大原则

* **单一数据源**
  * 整个应用程序的`state被存储在一棵object tree中`，且`这个object tree只存储在一个store`中
  * Redux并`没有强制让我们不能创建多个Store`，但是那样做并`不利于数据的维护`
  * `单一的数据源`可以热让整个应用程序的`state变得方便维护、跟踪、修改`





* **State是只读的**
  * `唯一修改过State的方法一定是触发action`，`不要试图在其他地方通过任何的方式来修改State`
  * 这样就确保了View或网络请求都`不能直接修改state`，他们`只能通过action来描述自己想要如何修改过state`
  * 这样可以保证所有的修改`都被集中化处理`，且`按照严格的顺序来执行`







* **使用纯函数来执行修改**
  * 通过reducer`将旧的state和action联系在一起`，且`返回一个新的State`
  * 随着`应用程序的复杂度增加`，我们可以`将reducer拆分成多个小的reducers`，`分别操作不同的state tree的一部分`
  * 但是`所有的reducer都应该是纯函数`，不能产生任何的副作用

# 4.React中使用Redux





了解了redux的基本使用，我们就可以在react中使用redux了，

```jsx
import store from '../store'
import { createAddAction, createSubAction } from '../store/actionCreator'
export class Home extends PureComponent {
  constructor() {
    super()
    this.state = {
      // 初始化store中的值
      counter: store.getState().counter,
    }
    this.unsubscribe = null
  }

  componentDidMount() {
    // 订阅store中的变化
    this.unsubscribe = store.subscribe(() => {
      const { counter } = store.getState()
      this.setState({
        counter,
      })
    })
  }

  componentWillUnmount() {
    // 取消订阅
    this.unsubscribe && this.unsubscribe()
  }

  handleClick(num) {
    const action = num > 0 ? createAddAction(num) : createSubAction(num)
    store.dispatch(action)
  }

  render() {
    const { counter } = this.state
    return (
      <div>
        <h2>Home Counter:{counter}</h2>
        <div>
          <button onClick={(e) => this.handleClick(-1)}>-1</button>
          <button onClick={(e) => this.handleClick(3)}>+3</button>
          <button onClick={(e) => this.handleClick(5)}>+5</button>
        </div>
      </div>
    )
  }
}
```

## 4.1.react-redux库

* 上面是原生的使用方式，但是会发现一个问题，每个组件使用store都很繁琐，而且需要在挂载的时候订阅，卸载的时候取消订阅，这些步骤是重复的
* 所以使用`react-redux`三方库来解决这个问题，这个库已经将上述的步骤封装了

**安装**

```cmd
npm i react-redux
```

**使用步骤**

* 首先从库中导入一个`Provider`，且`提供一个store`
  * 因为建议一个项目只用一个store，所以这个store写`在入口文件即可`，这样所有的组件都能使用这个store
* 然后在需要使用stroe的组件中在这个库中`导入connect`即可
  * connect函数本身`是一个高阶函数`，它需要提供两个参数，`返回值是一个高阶组件`，这个高阶组件接受一个组件
  * **参数一**：是一个函数
    * 这个函数有一个参数，是state(store中的state)，返回值是一个对象，表示`要使用的是store中的哪些属性`
    * 这些属性会`被注入到传入的组件中`，通过this.props.xxx获取
  * **参数二**：是一个函数
    * 这个函数有一个参数，是dispatch(用于发射action的)，返回值是一个对象，对象中函数，`表示发射action的函数`
    * 这些函数会被`注入到传入的组件中`，通过this.props.xxx获取



index.js

```jsx
import { Provider } from 'react-redux'
import store from './store'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <Provider store={store}>
    <App />
  </Provider>
)
```



使用store的组件

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'
import { createAddAction, createSubAction } from '../store/actionCreator'

export class About extends PureComponent {
  handleClick(num) {
    if (num > 0) {
      this.props.addCount(num)
    } else {
      this.props.subCount(num)
    }
  }
  render() {
    return (
      <div>
        <h2>About count:{this.props.counter}</h2>
        <div>
          <button onClick={(e) => this.handleClick(+66)}>+66</button>
          <button onClick={(e) => this.handleClick(-66)}>-66</button>
        </div>
      </div>
    )
  }
}

// 想要使用store中state的哪些参数
const mapStateToProps = (state) => ({
  counter: state.counter,
})

// 派发action的函数
const mapDispatchToProps = (dispatch) => ({
  addCount: (num) => dispatch(createAddAction(num)),
  subCount: (num) => dispatch(createSubAction(num)),
})
export default connect(mapStateToProps, mapDispatchToProps)(About)

```

# 5.Redux的异步操作

有时候我们希望在store中共享从服务器拿到的数据，一种方式就是在组件内部获取到数据，然后通过派发action的方式去更新store中的数据，但是这种方式有一个不好的点在于，发送网络请求的代码和组件代码杂糅在一起了，最好的方式就是共享的数据来源于store，那么请求的操作也要放在store中，但是redux中dispatch总是同步的，这就造成了拿不到数据。



解决方案就是，`派发一个函数`，`原本dispatch`是派发一个对象的，是`不支持派发函数的`，这时候就需要使用中间件来对dispatch进行增强 官方推荐的中间件是`redux-thunk`

## 5.2.redux-thunk

**安装**

```
npm i redux-thunk
```

**使用**

* 在redux中可以引入applyMiddleware函数，用于应用中间件
* 传入到createStore的第二个参数
* 之后dispatch就可以派发函数了
* 派发的函数会被自动回调，且传入两个参数，`dispatch和getState`

```js
import { createStore, applyMiddleware } from 'redux'
import reducer from './reducer'
import thunk from 'redux-thunk'

//  applyMiddleware(thunk,middleware1,middleware2,...)
const store = createStore(reducer, applyMiddleware(thunk))

export default store
```

actionCreator.js

```js
export const fetchBannerListAction = () => {
  return function (dispatch, getState) {
    fetch('http://12.07.32.32:8000/home/multidata')
      .then((res) => res.json())
      .then((res) => {
        dispatch(createFetchData(res.data.banner.list))
      })
      .catch((e) => {
        alert(e)
      })
  }
}
```

# 6.combineReducers函数

一个大型项目的开发，可能会有很多人，那么如果将所有的的操作都放在一个store中(不是分成多个store)，那么合并代码的时候肯定会有冲突，这时候就需要模块化的处理，每个模块下都有reducer、初始值等操作，这时候就会涉及一个问题，多个reducer怎么合并呢，这时候就需要用到`combineReducers函数`



`这个函数来自于redux内部`



```js
import { createStore, applyMiddleware, compose, combineReducers } from 'redux'
import counterReducer from './counter'
import homeReducer from './home'

// 合并各个模块的reducer
const reducer = combineReducers({
  counter: counterReducer,
  home: homeReducer,
})
const store = createStore(reducer)
```

# 7.Redux Toolkit

* **Redux Toolkit是官方推荐的编写Redux逻辑的方法**
  * 使用createStore方式编写redux的逻辑过于繁琐和复杂
  * 且代码通常拆分在多个文件中(虽然也可以放在一个文件中，但是代码量过多，不利于管理)
  * Redux Toolkit包`旨在于成为编写Redux逻辑的标准方式`，从而解决上面提到的问题
  * 在很多地方为了方便称呼，也将其称为`"RTK"`





**安装**



**注意：redux toolkit只对redux的编写逻辑进行了简化，但是使用redux的数据还是需要借助`react-redux`库**

```cmd
npm i @reduxjs/toolkit react-redux
```



**核心API**

* **configureStore**：包装createStore以提供`简化的配置项`和`良好的默认值`。他可以自动组合slice reducer(开发中通常会有多个reducer切片 最后在合并)，添加你提供的任何Redux中间件，`redux-thunk默认包含`，且`启用了Redux DevTools Extension`
* **createSlice**：创建一个仓库切片，会自动生成reducer，且带有相应的actions
* **createAsyncThunk**

## 7.1.store的创建

**configureStore用于创建store对象，常见的参数如下**

* `reducer`：合并多个reducer切片
* `middleware`：可以使用参数，传入其他的中间件
* `devTools`：是否配置devTools工具，`默认true`



```js
import { configureStore } from '@reduxjs/toolkit'
// counter reducer切片
import counterSliceReducer from './features/counter'

const store = configureStore({
  reducer: {
    counter: counterSliceReducer,
  },
})

export default store

```

## 7.2.创建切片

开发中我们不可能将所有的代码都写在一个js文件中，肯定要分模块的，reduxtoolkit提供了`createSlice `方法,可以对不同的模块创建一个切片



**主要参数**

* `name`：用户标记slice的名字
  * 在之后的redux-devtool中会显示对应的名字
* `initialState`：初始化值
  * 第一次初始化的值
* `reducers`：相当于之前的reducer函数
  * 对象类型，且可添加多个函数
  * 每个函数类似于redux原来reducer中的一个case语句
  * 函数的参数
    * 参数一：state
    * 参数二：action
* **createSlice返回值是一个对象，包含所有的actions**

```js
import { createSlice } from '@reduxjs/toolkit'

const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    count: 888,
  },
  reducers: {
    addNumber(state, { payload }) {
      // 值都在payload中
      // 这里也简化了 不需要返回一个新对象了 内部帮我们做了
      state.count = state.count + payload
      // return { ...state, count: payload + state.count }
    },
    subNumber(state, action) {},
  },
})

// 外界需要使用dispatch派发reducers中的事件才能完成调用，不能直接调用reducers中的函数 否则没有state和action参数
export const { addNumber, subNumber } = counterSlice.actions
export default counterSlice.reducer
```

## 7.3.执行异步操作

* 通过createStore方式使用redux，是通过redux-thunk中间件来完成异步操作的
* Redux Toolkit默认已经集成了Thunk相关的功能
* 如果我们想要创建一个异步的action，需要用到createAsyncThunk方法
* 这个action 并没有在切片里 而是单独创建的

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

// 创建一个异步的action
// 参数一：名字 devtool可以看到
// 参数二：函数
export const fetchBannersAction = createAsyncThunk('fetchbanners', async () => {
  let res = await fetch('http://123.207.32.32:8000/home/multidata')
  res = await res.json()
  return res.data
})

```

**那么我们怎么将获取到的内容替换到state中呢**

* 当createAsyncThunk创建出来的action被dispatch时(被调用时)，会存在三种状态
  * pending：action被触发，但是没有最终结果
  * fulfilled：获取到最终结果
  * rejected：执行过程中有错误或者抛出了异常
* 然后我们可以在slice中的`extraReducers`中监听当前action的每个状态

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

// 创建一个异步的action
export const fetchBannersAction = createAsyncThunk('fetchbanners', async () => {
  let res = await fetch('http://123.207.32.32:8000/home/multidata')
  res = await res.json()
  return res.data
})

const homeSlice = createSlice({
  name: 'home',
  initialState: {
    banners: [],
  },
  reducers: {
    fetchBanners(state, { payload }) {
      console.log(payload)
      // state.banners = payload
    },
  },
  extraReducers: {
    [fetchBannersAction.pending]() {
      console.log('fetchBannersAction pending')
    },
    [fetchBannersAction.fulfilled](state, { payload }) {
      const banner = payload.banner.list
      state.banners = banner
    },
    [fetchBannersAction.rejected]() {
      console.log('fetchBannersAction rejected')
    },
  },
})

export const { fetchBanners } = homeSlice.actions

export default homeSlice.reducer
```

**extraReducers的链式写法**

如果不想用计算属性名的写法，可以使用链式调用

**上面的那种写法 官方说RTK2.0会被废弃，建议使用这种链式写法**

```js
const homeSlice = createSlice({
  name: 'home',
  initialState: {
    banners: [],
  },
  // extraReducers的链式写法
  extraReducers: (builder) => {
    builder.addCase(fetchBannersAction.fulfilled, (state, { payload }) => {
      state.banners = payload.banner.list
    })
  },
})
```

**其他写法**

如果觉得还要监听很麻烦，那么可以直接在异步操作中dispatch一个action然后更改，createAsyncThunk接受两个参数，第一个参数时用户dispatch时候传入的参数(可以不传)，第二个参数就是store对象，我们可以拿到store，通过store.dispatch进行派发 然后在slice的reducer中更改state

```js
// 创建一个异步的action
export const fetchBannersAction = createAsyncThunk(
  'fetchbanners',
  async (info, store) => {
    let res = await fetch('http://123.207.32.32:8000/home/multidata')
    res = await res.json()
    // 取出数据
    store.dispatch(fetchBanners(res.data.banner.list))
    // return res.data
  }
)

const homeSlice = createSlice({
  name: 'home',
  initialState: {
    banners: [],
  },
  reducers: {
    fetchBanners(state, { payload }) {
      console.log(payload)
      state.banners = payload
    },
  },

})
```



# 8.middleware

* redux可以使用中间件，减少我们重复的操作
  * 比如在每次派发action的时候我们需要打印当前派发的action
  * 那么所有的页面组件都有派发的时候 我们要重复很多遍
  * 这时候只需要在创建组件的时候添加一个中间件即可
* 中间件的原理就是劫持api，然后添加一些自己的功能

**log中间件**

```js
const store = createStore(reducer)

// 添加一个log的中间件
// 这种改了原本方法的叫做 monkey patch  猴补丁
function log(store) {
  const next = store.dispatch
  function dispatchLog(action) {
    console.log('派发之前的action', action)

    next(action)

    console.log('派发之后的结果', store.getState())
  }

  store.dispatch = dispatchLog
}

log(store)
```



  
