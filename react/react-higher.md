# 1.React的组件化开发

**组件化思想的应用**

* 有了组件化的思想，开发中要充分的利用它
* 尽可能的将页面拆分成一个个小的、可复用的组件，方便代码的组织和管理，且扩展性也更强

**React组件更加灵活和多样，按照不同的方式可以分成很多类别的组件**

* 根据组件的`定义方式`，可以分为：`函数组件Function Component`和`类组件Class Component`
* 根据组件`内部是否有状态`需要维护，可分为：`无状态组Stateless Component`件和`有状态组件Stateful Component`
* 根据组件的`不同职责`可分为：`展示组件Presentational Component`和`容器组件Container Component`

**这些概念会有很多重叠，但是他们最主要是关注数据逻辑和UI展示分离**

* 函数组件、无状态组件、展示组件主要关注UI展示
* 类组件、有状态组件、容器组件主要关注数据逻辑

**还有很多组件概念：如异步组件、高阶组件后续补充**



## 1.1.类组件

* 类组件的要求
  * 组件的名称是`大写字符开头`（无论是类组件还是函数组件）
  * 类组件需要`继承自React.Component`
  * 类组件`必须要实现render函数`
  * constructor是可选的，通常在constructor中初始化一些数据

## 1.2. render函数的返回值

**当render被调用时，它会检查this.props和this.state的变化，并且返回以下类型之一**

* React元素
  * 通过JSX编写的代码会被翻写成React.createElement，这就是React元素
* 数组或fragments
* Portals
* 字符串或数字类型：在DOM中会被渲染为文本节点
* boolean或null或undefined：什么都不渲染

```js
class App extends React.Component {
  constructor() {
    super()
    this.state = {
      message: 'App Component',
    }
  }

  render() {
    return (
      <div>
        {/* 1.可以返回React元素 通过JSX编写的代码会被翻写成React.createElement，这就是React元素 */}
        <h1>{this.state.message}</h1>
        {/* 2.可以返回一个数组，数组里面可以是React元素或字符串，会将数组中的元素遍历然后展示 */}
        {[<h1 key={'1232'}>我是h1元素</h1>, '哈哈哈']}
        {/* 3.返回一个字符串/数字 */}
        sss{123}
        {/* 4.返回 */}
      </div>
    )
  }
}
```

## 1.3.函数组件

**函数组件返回值和类组件的render返回内容是一样的**



**函数组件的特点(后续hooks会弥补这些特点)**

* `没有生命周期`，但是会被挂载更新
* `this关键字不能指向组件实例`（因为没有组件实例）
* `没有内部状态`

```js
function App() {
  return (
    <div>
      <h1>App Functional Component</h1>
    </div>
  )
}
```



# 2.React组件生命周期

**这里说的React生命周期，主要是类的生命周期，因为函数没有生命周期(后续可以通过hooks来模拟生命周期)**



**React将生命周期分为三个阶段**

* 装载阶段(Mount)，组件第一次在DOM树中被渲染的过程
* 更新阶段(Update)，组件状态发生变化，重新更新渲染的过程
* 卸载阶段(Unmount)，组件从DOM树中被移除的过程



## 2.1.React常用的生命周期



**React有如下生命周期函数**

* `constructor`: 类的constructor
  * constructor通常只做两件事：1.通过this.state赋值对象来初始化内部的state，2.为事件绑定this
* `render`: 类的render函数

* `componentDidMount`：组件已经挂载到DOM上时，会调用
  * 依赖DOM的操作可以在这里进行
  * 在此处可以发送网络请求(官方建议)
  * 可以在此处添加一些订阅，记得在`componentWillUnmount`中取消订阅
* `componentDidUpdate`：组件已经发生了更新时，会调用
* `componentWillUnmount`：组件即将被移除时，会调用



### 2.1.1**装载阶段**

* 挂载阶段会依次执行以下方法
  1. `constructor`
  2. `render`
  3. `componentDidMount`

**说明**

这个阶段会先执行组件的constructor，因为组件实际上是一个类(函数的这里先不说)，我们调用的时候其实就是相当于new，那么第一步肯定是执行constructor，之后会执行render函数,将我们编写的JSX元素渲染成真实的DOM元素，然后就会执行componentDidMount函数，表示真实DOM已经被渲染



### 2.1.2更新阶段

* 更新阶段会依次执行以下方法
  1. `render`
  2. `componentDidUpdate`

**说明**

当更调用setState方法改数据之后，这时候会重新调用render方法，重新渲染，之后会调componentDidUpdate方法表示已经更新完毕

### 2.1.3卸载阶段

* 卸载阶段会依次执行以下方法
  1. `componentWillUnmount`

**说明**

- 卸载阶段只有一个生命周期函数 ，componentWillUnmount，该函数会在组件卸载及销毁前执行

## 2.2.React不常用的生命周期

![](https://cdn.staticaly.com/gh/wscymdb/picx-images-hosting@master/algorithm/Snipaste_2023-08-17_16-12-36.41qrpe0ieg60.webp)



* getDerivedStateFromProps:state的值在任何时候都依赖于props时使用，该方法返回一个对象来更新state
* `getSnapshotBeforeUpdate`：在React更新DOM之前回调的一个函数，可以获取DOM更新的一些信息（比如滚动位置）
* `shouldComponentUpdate`：更新的时候是否重新渲染



# 3.React组件间通信



## 3.1.父传子

* 在父组件中通过`属性=值`的方式向子组件传递参数
* 在子组件中通过`this.props`接收参数

父组件

```js
 class main extends Component {
  constructor() {
    super()
    this.state = {
      baaners: ['广告一', '广告二', '广告三'],
    }
  }
  render() {
    const { baaners } = this.state
    return (
      <div>
        <MainBanner baaners={baaners} />
      </div>
    )
  }
}
```

子组件

```js
 class MainBanner extends Component {
   // 父组件传入的参数会在constructor的第一个参数拿到
  // 如果没有自己的数据要写可以直接不写constructor
  // 父组件传入的数据会自动绑定到实例的props上
  // constructor(props) {
  // super(props)  // 这步操作相当于 this.props=props
  //   this.state = {}
  // }

  render() {
    console.log(this.props)
    const { baaners } = this.props
    return (
      <div>
        <h1>广告类别</h1>
        <ul>
          {baaners.map((item) => (
            <li key={item}>{item}</li>
          ))}
        </ul>
      </div>
    )
  }
}

```





## 3.2.参数类型限制(propTypes)

[官网示例](https://zh-hans.legacy.reactjs.org/docs/typechecking-with-proptypes.html)



* 对于传递给子组件的数据，有时候希望进行验证
  * 如果项目中继承了Flow或TypeScript，那么直接就可以进行类型验证
  * 如果没有，可以通过prop-types库来进行参数验证
* 从React v15.5开始 React.PropTypes已移入另一个包中：`prop-types库`



子组件：

```js
import React, { Component } from 'react'
import PropTypes from 'prop-types'

export class MainBanner extends Component {
  // 如果没有自己的数据要写可以直接 不行constructor
  // 父组件传入的数据会自动绑定到实例的props上
  // constructor(props) {
  //   super(props)
  //   this.state = {}
  // }

  // props的默认值 从 ES2022 开始，你也可以在 React 类组件中将 defaultProps 声明为静态属性
  // static defaultProps = {
  //   title: '默认',
  // }
  render() {
    console.log(this.props)
    const { baaners, title } = this.props
    return (
      <div>
        <h1>{title}</h1>
        <ul>
          {baaners.map((item, i) => (
            <li key={i}>{item.value}</li>
          ))}
        </ul>
      </div>
    )
  }
}
// 对传入的props进行验证
MainBanner.propTypes = {
  // 表示banners的值是array类型，且值是必要的
  baaners: PropTypes.array.isRequired,
}
// props书写默认值
MainBanner.defaultProps = {
  title: '我是默认值',
}

export default MainBanner
```

## 3.3.子传父

**在Vue中可以通过自定义事件中去进行子传父的操作，但是React中没有自定义事件，React中通过在父组件中传递一个回调函数，然后子组件调用这个回调完成子传父**



父组件

```js
export class App extends Component {
  constructor() {
    super()
    this.state = {
      count: 100,
    }
  }
  addCount(v) {
    console.log(v)
    this.setState({
      count: this.state.count + v,
    })
  }
  render() {
    const { count } = this.state
    return (
      <div>
        <h1>{count}</h1>
        <AddCounter addCount={(v) => this.addCount(v)} />
      </div>
    )
  }
}
```

子组件

```js
export class AddCounter extends Component {
  addCount(v) {
    console.log(v)
    const { addCount } = this.props
    addCount(v)
  }
  render() {
    return (
      <div>
        <button onClick={() => this.addCount(1)}>+1</button>
        <button onClick={() => this.addCount(3)}>+3</button>
        <button onClick={() => this.addCount(8)}>+8</button>
      </div>
    )
  }
}

```

# 4.React组件插槽用法

**React中是没有插槽的概念的，因为React压根不需要插槽，他很灵活，可以通过很多方法实现这种效果**



## 4.1.通过Children方式实现插槽

* 如果一个子组件是双标签，且里面有内容，那么就会作为children
* 在子组件的`this.props.children`中可以获取到
* 如果只有一个元素那么`this.props.children`就是当前元素
* 如果有多个`this.props.children`就是一个数组，里面存放元素

父组件

```js
export class App extends Component {
  render() {
    return (
      <div>
        <NavBar>
          <button>按钮</button>
          <h1>我是标题</h1>
          <i>斜体文字</i>
        </NavBar>
      </div>
    )
  }
}
```

子组件

```js
export class index extends Component {
  render() {
    console.log(this.props.children)
    const { children } = this.props
    return (
      <div className="bar">
        <div className="left">{children[0]}</div>
        <div className="center">{children[1]}</div>
        <div className="right">{children[2]}</div>
      </div>
    )
  }
}

// 限制children的个数
NavBar.propTypes = {
  // 限制只允许传入一个内容
  // children:propTypes.element
  // 限制传入的内容要是多个
  children:propTypes.array
}
```

## 4.2.通过props方式实现插槽

**这种方式通过父传子的方式 给子组件传入值**



父组件

```js
export class App extends Component {
  render() {
    return (
      <div>
        <NavBar2
          leftSlot={<button>按钮2</button>}
          centerSlot="哈哈哈"
          rightSlot={123}
        />
      </div>
    )
  }
}
```

子组件

```js
export class NavBar extends Component {
  render() {
    const { leftSlot, centerSlot, rightSlot } = this.props
    return (
      <div className="bar">
        <div className="left">{leftSlot}</div>
        <div className="center">{centerSlot}</div>
        <div className="right">{rightSlot}</div>
      </div>
    )
  }
}
```

## 4.3.作用域插槽的实现

**父组件传入的属性是一个函数，然后子组件调用这个函数进行参数的传递，从而实现作用域插槽**



父组件

```js
export class App extends Component {
  constructor() {
    super()
    this.state = {
      bars: ['流行', '新款', '精选'],
      currentIndex: 0,
    }
  }
  changeBar(i) {
    this.setState({
      currentIndex: i,
    })
  }
  getType(item) {
    if (item === '流行') {
      return <button>{item}</button>
    } else {
      return <span>{item}</span>
    }
  }
  render() {
    const { bars, currentIndex } = this.state
    return (
      <div className="app">
        <BarHeader
          bars={bars}
          currentIndex={currentIndex}
 				// 这里实现了作用域插槽
          // itemType={(item) => <button>{item}</button>}
          itemType={(item) => this.getType(item)}
          
          changeBar={(i) => this.changeBar(i)}
        />
        <BarItem content={bars[currentIndex]} />
      </div>
    )
  }
}
```

子组件

```js
export class BarHeader extends Component {
  barClick(i) {
    this.props.changeBar(i)
  }
  render() {
    const { bars, currentIndex, itemType } = this.props
    return (
      <div className="bar">
        {bars.map((item, i) => (
          <div
            className={`${currentIndex === i ? 'active' : ''} bar-item`}
            key={item}
            onClick={() => this.barClick(i)}
          >
            {/* {item} */}
            {itemType(item)}
          </div>
        ))}
      </div>
    )
  }
}
```





# 5.React非父子的通信

* 在开发中，比较常见的数据传递方式是通过props属性自上而下的传递(父传子，子传孙...)
* 但是如果层级很深，比如父组件想传递数据给曾孙，但是这些数据也会被父和孙给获取到(他们用不到)，麻烦且冗余
* 所以React提供了一个API：`Context`，可以解决上述问题



**补充知识点**

React提供了向v-bind={...}的语法，方便我们将每个属性都传入到子组件中

```js
const info = {name:'zs',age:18}

<Home {...info} />
```



## 5.1.Context

**使用步骤**

1. 创建一个context对象
2. 在父组件中引入这个context，然后用context.provider包裹要共享数据的子组件，且在context.provider中传入共享的数据
3. 子组件中 通过contextType指定要获取的是哪个context
4. 就可以在子组件实例中使用共享数据了

context.js

```js
import React from 'react'
// 1.创建context
export const DemoCnotext = React.createContext()
```

父组件

```js
import { DemoCnotext } from './context/demo-context'
import Home from './Home'
export class App extends Component {
  render() {
    return (
      <div>
        <h1>App</h1>
      {/* 2.包裹要共享数据的组件 传递共享数据 */}
        <DemoCnotext.Provider value={{ name: 'zs', age: 18 }}>
          <Home />
        </DemoCnotext.Provider>
      </div>
    )
  }
}
```

子组件

```js
import { DemoCnotext } from './context/demo-context'

export class Home extends Component {
  render() {
    // 4.使用共享数据
    console.log(this.context)
    return <div>Home</div>
  }
}
// 3. 指定使用哪个context
Home.contextType = DemoCnotext

export default Home

```



## 5.2.函数组件中使用Context

**函数组件中使用和类组件中有些不同，但是父组件中共享的方式是一样的**

子组件

```js
import React from 'react'
import { DemoCnotext } from './context/demo-context'

export default function Fnc() {
  return (
    <div>
      <h1>函数组件</h1>
     {/* 内部必须是一个回调函数 参数就是共享的数据 */}
      <DemoCnotext.Consumer>
        {(val) => <h2>获取共享数据:{val.name}</h2>}
      </DemoCnotext.Consumer>
    </div>
  )
}
```

## 5.3.使用多个context

`不常用了解即可`

* 父组件可以进行Context的互相嵌套
* 子组件通过xxxContext.Consumer中获取

父组件

```js
import { DemoCnotext } from './context/demo-context'
import { UserContext } from './context/user-context'
export class App extends Component {
  render() {
    return (
      <div>
        <h1>App</h1>
        {/* 2.包裹要共享数据的组件 传递共享数据 */}
        {/* 如果想要使用多个context 可以包裹context */}
        <UserContext.Provider value={{ nickName: '就发顺丰' }}>
          <DemoCnotext.Provider value={{ name: 'zs', age: 18 }}>
            <Home />
            <Fnc />
          </DemoCnotext.Provider>
        </UserContext.Provider>
      </div>
    )
  }
}
```

子组件

```js
import { DemoCnotext } from './context/demo-context'
import { UserContext } from './context/user-context'

export class Home extends Component {
  render() {
    // 4.使用共享数据
    console.log(this.context)
    return (
      <div>
        <h1>Home</h1>
        <UserContext.Consumer>
          {(val) => <h2>{val.nickName}</h2>}
        </UserContext.Consumer>
      </div>
    )
  }
}
// 3. 指定使用哪个context
Home.contextType = DemoCnotext
```



## 5.4.Context的默认值

**当子组件没有被Provider包裹，那么在子组件中获取context的结果是undefined，所以可以设置一个默认值**



context.js

```js
// 1.创建context
export const DemoCnotext = React.createContext({ name: '我是默认值' })
```

父组件

```js
import { DemoCnotext } from './context/demo-context'

export class App extends Component {
  render() {
    return (
      <div>
        <h1>App</h1>
          <DemoCnotext.Provider value={{ name: 'zs', age: 18 }}>
          </DemoCnotext.Provider>
					<Home />
      </div>
    )
  }
}
```



## 5.6.Context相关的API

* React.createContext
  * 创建一个需要共享的Context对象，可以`传递一个参数作为默认值`
  * 如果一个组件订阅了Context，那么这个组件就会从离自身最近的那个匹配的Provider中读取当前的context
  * 如果组件在顶层查找过程中没有找到对应的Provider，那么就会使用默认值
* Context.Provider
  * 每个Context对象会返回一个Provider React组件，它允许消费组件订阅context的变化
  * Provider`接收一个value属性`，传递给消费组件
  * 一个Provider可以和多个消费组件有对应关系
  * 多个Provider也`可以嵌套使用`，内层会覆盖外层的数据
  * 当Provider的value值发生变化时，`它内部的所有消费组件都会重新渲染`
* Class.contextType
  * 挂载在class上的contextType属性会被重新赋值一个由React.createContext()创建的Context对象
  * 这能让你`使用this.context`来消费最近的Context上的那个值
  * 可以在任`何生命周期中访问到它`，包括`render函数中`
* Context.Consumer
  * 这里，React组件也可以订阅到context变更，这能让你在`函式组件中完成订阅context`
  * 这里需要`函数作为子元素`这种做法
  * 这个函数接收当前的context值，返回一个React节点

## 5.7.event-bus

还可以使用Event-bus来完成组件的通信，不过要找三方库，或自己写

使用方法同vue 小程序一样 这里不在赘述



# 6.setState的使用详解

## 6.1.为什么使用setState

Vue中不需要setState，因为Vue中对数据进行了劫持，但是React中没有，数据发生了变化，React是不知道的，所以只有通过setState告诉React



## 6.2.setSate的不同用法 

下面案例只用于演示setState的用法，不是完整代码



**基本用法**

```js
 changeText() {
    // 1.setState基本用法
     this.setState({
       message: '你好，世界',
     })
  }
```

**传入一个回调**

```js
 changeText() {
    // 2.setState用法二 传入回调函数
    // 好处一 可以在回调函数中编写对应的逻辑，这样内聚性会更强
    // 好处二 该回调中会传入两个参数 分别是当前实例的state 和 props  当然也可以通过this.state和this.props获取，只不过回调中可以直接获取不需要this
   // 因为setState是异步更新 但是这个函数里面的state是更新过后的值 也就是新值 如果this.state那么是尚未更新的值
     this.setState((state, props) => {
       return {
         message: '你好，世界',
       }
     })
  }
```

**获取更改后的数据**

```js
 changeText() {
    // 3.setState默认是异步调用
    // 如果希望在数据更新完毕后拿到更新后的值可以在setState函数的第二个参数传入一个回调，当数据更新完毕后会调用这个回调
    this.setState({ message: '你好世界' }, () =>
      console.log(this.state.message)
    )
   // 这里获取的值 还是更改之前的值
    console.log(this.state.message)
  }
```

## 6.3.setState是异步调用

* setState设计为异步，可以`显著的提升性能`
  * 如果每次调用set State都进行一次更新，那么意味着render函数会被频繁调用，页面重新渲染，这样的效率是很低的
  * 最好的办法应该`是获取到多个更新，之后进行批量更新`
* 如果同步更新了state，但是还没有执行render函数，`那么state和props不能保持同步`
  * state和props不能保持一致性，会在开发中产生很多的问题
  * 比如 父组件给子组件传入message参数 这时候在父组件中同步更新了message，但是render函数还没执行，这时候子组件的message还是之前的值 但是父组件已经是更改过的值了
  * 

### 6.3.1.setState非异步情况

**React18之前有些情况，是同步的，React18之后任何情况都是异步的**

[官方说明](https://react.dev/blog/2022/03/29/react-v18#whats-new-in-react-18)



验证一: 在setTimeout中更新

```js
changeText() {
	setTimeout(() => {
    this.setState({name:'zs'})
  },0)  
}
```

验证二：原生DOM事件

```js
componentDidMount() {
  const btnEl = document.getElementById('btn')
  btnEl.addEventlistener('click',() => {
    this.setState({name:'zs'})
  })
}
```

### 6.3.2.如何实现同步更新

这种效果类似Vue中的`nextTick`函数，

**需要从react-dom包中引入flushSync并使用**

```js
import { flushSync } from 'react-dom'

changeText1() {
  flushSync(() => {
    this.setState({ message: '你好世界' })
  })
  console.log(this.state.message)
}
```

# 7.React性能优化SCU

## 7.1.keys的优化

* 方式一：在最后位置插入数据
  * 这种情况，有无key意义不大
* 方式二，在前面插入数据，或中间插入数据
  * 这种情况，在没有key的情况下左右的列表内容都要修改
* 当子元素拥有key时，React使用key来匹配原有树上的子元素以及最新树上的子元素
  * 比如在a元素和c元素中间插入b元素，那么只会将b元素进行位移，并不会修改b元素

**key的注意事项**

* key应该是唯一的
* key不要使用随机数(随机数会在下一次render时，会重新生成一个新的数字)
* 使用index作为key，对性能是没有优化的(比如插入一个元素，那么后面的index就会变化的，那么新旧DOM对比的时候key是不一样的)

## 7.2.控制render函数被调用

* 有以下场景
  * App组件中声明了message变量，且引入了Home组件
  * 现在点击按钮修改了message，这时候会触发render函数，Home的render函数也被重新执行了
  * 但是我们并没有修改Home中的数据，这样是浪费性能的
* 如何控制render方法是否被调用
  * 通过`shouldComponentUpdate钩子`即可

## 7.3.shouldComponentUpdate

* **React给我们提供了一个生命周期方法`shouldComponentUpdate`(很多时候，`简称SCU`),这个方法接受参数，且需要有返回值**



* 该方法有两个参数
  * 参数一：`nextProps` 修改之后，最新的props属性
  * 参数二：`nextState` 修改之后，最新的state属性



* 该方法`返回值`是一个`boolean`类型
  * 返回值为true， 表示需要调用render方法
  * 返回值为false，表示不需要调用render方法
  * 默认返回值是true， 也就是只要state发生改变，就会调用render方法



**我们可以通过判断state中的值是否变化，判断props的值是否发生变化来决定要不要执行render**

```js
export class App extends Component {
  constructor() {
    super()
    this.state = {
      message: 'hello world',
      count: 0,
    }
  }

  changeText() {
    this.setState({
      message: 'hello world1',
    })
  }
  changeCount() {
    this.setState({ count: this.state.count + 1 })
  }

  render() {
    console.log('App render')
    const { message, count } = this.state
    return (
      <div>
        <h2>{message}</h2>
        <button onClick={() => this.changeText()}>修改文本</button>
        <button onClick={() => this.changeCount()}>{count}</button>
        <Home count={count} />
      </div>
    )
  }

  shouldComponentUpdate(nextProps, nextState) {
    console.log(nextState)
    if (
      nextState.message !== this.state.message ||
      nextState.count !== this.state.count
    )
      return true
    return false
  }
}
```



## 7.4.PureComponent

* 上面例子中所有判断都自己来写，如果所有的类，都需要手动来实现shouldComponentUpdate，那么工作量是非常大的
* 所以React提供了一个类`PureComponent`
* 我们只需用将继承的类由`Component`转换为`PureComponent`即可



```js
import React, { PureComponent } from 'react'

export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      message: 'hello world',
    }
  }

  render() {
    console.log('App render')
    const { message } = this.state
    return (
      <div>
        <h2>{message}</h2>
      </div>
    )
  }

}
```



## 7.5.memo

PureCompoent适用于类组件，但是不适用函数组件，所以要使用`memo`高阶函数

```js
import { memo } from 'react'

const Fnc = memo(function () {
  console.log('Fnc')
  return <h1>Fnc</h1>
})

export default Fnc

```

## 7.6.数据不可变的力量

* 开发中如果要对`引用类型数据`进行操作，`不能直接对原数据进行修改`，需要对原数据进行`浅拷贝`，然后更改
* 这样的目的是为了PureComponent对更新的数据做比较的时候 可以触发render函数

```js
export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      friends: [
        { name: 'zs', age: 19 },
        { name: 'ls', age: 21 },
      ],
    }
  }

  addClick() {
    // 这里不能直接对friends进行更改，而是要先浅拷贝一层然后在操作
    // 不能直接 this.state.friends.push({ name: '王武', age: 25 })
    // 因为继承的是PureComponent 当数据发生变化的时候会和原来的值做比较
    // 直接push那么等于原来的值还是一样没有变化 所以不会触发render函数
    const newFriends = [...this.state.friends]
    newFriends.push({ name: '王武', age: 25 })
    this.setState({
      friends: newFriends,
    })
  }
  render() {
    const { friends } = this.state
    console.log(friends)
    return (
      <div>
        <h1>好友列表</h1>
        <ul>
          {friends.map((item, i) => (
            <li key={i}>{item.name}</li>
          ))}
        </ul>
        <button onClick={() => this.addClick()}>添加好友</button>
      </div>
    )
  }
}
```

# 8.获取DOM方式refs

**获取DOM的方式有三种,开发中推荐使用`方式二`**



**方式一：ref字符串**

这种方式和Vue2中一样

```js
export default class App extends PureComponent {
  getDOMH2() {
     // 通过ref获取 不推荐使用了
     const el = this.refs.ddm
      console.log(el)
  }

  render() {
    return (
      <div>
        {/* 1.ref方式获取 */}
        <h2 ref="ddm">我是h2</h2>
        <button onClick={() => this.getDOMH2()}>获取DOM</button>
      </div>
    )
  }
}
```





**方式二：createRef**

这种方式和Vue3很像

```js
import React, { PureComponent, createRef } from 'react'
export default class App extends PureComponent {
  constructor() {
    super()
    this.titleRef = createRef()
  }
  getDOMH2() {

    // 通过createRef获取
     console.log(this.titleRef.current)
  }

  render() {
    return (
      <div>
        <h2 ref={this.titleRef}>我是</h2>
        <button onClick={() => this.getDOMH2()}>获取DOM</button>
      </div>
    )
  }
}
```



**方式三：ref传入函数**

```js
export default class App extends PureComponent {
  constructor() {
    super()
    this.titleEl = null
  }
  getDOMH2() {
    console.log(this.titleEl)
  }

  render() {
    return (
      <div>
        <h2 ref={(el) => (this.titleEl = el)}>我是大哥</h2>
        <button onClick={() => this.getDOMH2()}>获取DOM</button>
      </div>
    )
  }
}
```



## 8.1.Ref的转发

* `ref不能作用于函数式组件`
  * 因为`函数式组件没有实例`，所以不能获取到对应的组件对象
* 但是`可以获取函数组件中某个DOM元素`
  * 使用`forwardRef`高阶函数



父组件

```js
export default class App extends PureComponent {
  constructor() {
    super()
    this.fnRef = createRef()
  }
  getInstanceFn() {
    console.log(this.fnRef.current)
  }
  render() {
    return (
      <div>
        <Fnc ref={this.fnRef} />
        <button onClick={() => this.getInstanceFn()}>获取函数组件DOM</button>
      </div>
    )
  }
}
```

子组件

```js
import { forwardRef } from 'react'

const Fnc = forwardRef(function (prop, ref) {
  // 第一个参数是父组件传入的prop 
  // 第二个参数就是父组件传入的ref
  return (
    <div>
      <h1 ref={ref}>Fnc</h1>
    </div>
  )
})

export default Fnc

```



# 9.受控和非受控组件

## 9.1.受控组件

* 如果一个表单元素，`绑定了属性对应的值`，那么用户用户是无法输入的,`这时候这个表单元素就是受控组件`
  * eg: type="text"对应的属性是value 事件是onchange
* 这时候需要给这个表单元素绑定对应的事件，在这个事件函数中更改表单元素对应的属性值
* `换句话来说受控组件是由React来维护状态的`

```js
export default class Example1 extends PureComponent {
  constructor() {
    super()
    this.state = {
      username: 'zs',
    }
  }

  handleUsernameChange(e) {
    this.setState({
      username: e.target.value,
    })
  }
  render() {
    const { username } = this.state
    return (
      <div>
         {/* 此时的input元素就是受控组件 */}
        <input
              type="text"
              value={username}
              onChange={(e) => this.handleUsernameChange(e)}
            />
      </div>
    )
  }
}
```



## 9.2.非受控组件

* React推荐`大多数情况下`使用`受控组件`来处理表单数据
* 如果非要使用非受控组件中的数据，我们需要使用`ref来从DOM节点`中获取表单数据
* **在非受控组件中通常使用defaultValue或defaultChecked来设置默认值**
* 非受控组件就是在操作原生DOM

```js
export default class Example4 extends PureComponent {
  constructor() {
    super()
    this.state = {
      intr: 'hhh',
    }
    this.iptRef = createRef()
  }

  handleClick(e) {
    // 获取非受控组件的值
    console.log(this.iptRef.current.value)
  }
  componentDidMount() {
    // 在这个生命周期中设置事件监听
    this.iptRef.current.addEventListener('input', () => {
      console.log(1)
    })
  }
  render() {
    return (
      <div>
        <h3>非受控组件</h3>
        <input type="text" ref={this.iptRef} defaultValue={this.state.intr} />
        <button onClick={(e) => this.handleClick(e)}>click me </button>
      </div>
    )
  }
}

```



# 10.React的高阶组件

**定义**

* 高阶组件的英文是`Higher-Order Component`，简称为`HOC`
* 官方定义：高阶组件是`参数为组件`，`返回值`是一个新的`组件函数`

**判断**

* 首先，高阶组件`本身不是一个组件，而是一个函数`
* 其次，这个函数的`参数是一个组件，返回值也是一个组件`

## 10.1.示例

**高阶组件的调用类似这样**

```js
const EnhanceComponent = higherOrderComponent(DComponent)
```



**高阶函数的编写过程类似这样**

```js
function higherOrderComponent(Cpn) {
  class NewComp extends PureComponent {
    render() {
      return <Cpn />
    }
  }
    NewComp.displayName = 'hhh' // devtool工具中可以区分名字
    return NewComp
}
```

* 在ES6中，类表达式中类名是可以省略的
* 组件的名称都可以通过displayName来修改过

## 10.2.高阶组件的缺陷

* HOC需要在`原组件上进行包裹或者嵌套，如果大量使用HOC`，将`会产生非常多的嵌套`，这`让调试变得非常困难`
* HOC可`以劫持props，在不遵守约定的情况下也可能造成冲突`

**这些问题会被Hooks解决 后续会写**





# 11.portals和fragment

## 11.1.portals

**和Vue3的teleport类似**

* 某些情况下我们希望渲染的内容独立于父组件，甚至独立当前挂载的DOM元素中(默认都是挂载到#root上)
* 这时候可以使用Portal

**参数**

* 第一个参数：`任何可渲染的React子元素`，例如：一个元素，字符串或fragment
* 参数二：`DOM元素`

```js
import { createPortal } from 'react-dom'
import Modal from './Modal'

export default class App extends PureComponent {
  render() {
    return (
      <div>
        <h1>protal的使用</h1>
        {createPortal(<h2>我是h2</h2>, document.querySelector('#cym'))}

        {createPortal(
          <Modal>
            <h1>我是标题</h1>
            <p>我是内容</p>
          </Modal>,
          document.querySelector('#modal')
        )}
      </div>
    )
  }
}
```







## 11.2.fragment

**这个和Vue的template类似，不会被渲染成实际的元素**



* 语法糖写法:`<></>`
* 如下情况不能使用语法糖，`要给Fragment绑定key的时候`

```js
import React, { Fragment, PureComponent } from 'react'

export default class App extends PureComponent {
  render() {
    return (
      <Fragment>
        <h2>woshi hhh</h2>
        {/* 这中写法是<Fragment>的语法糖 */}
        <>
          <h3>woshi</h3>
        </>
      </Fragment>
    )
  }
}

```



# 12.StrictMode严格模式

* **StrictMode是一个用来突出显示应用程序中潜在问题的工具**
  * 与Fragment一样，StrictMode不会渲染任何可见的UI
  * 严格模式检查仅在开发模式下运行，他们不会影响生产构建(开发模式下会警告，生产模式不会)
* **可以为任意部分开启严格模式**
* **开启严格模式后，组件的声明周期会执行两次，是刻意执行的，为了检查是否存在副作用，在生产模式中不会执行两次**



**严格模式检查的内容**

* 识别不安全的生命周期(有些生命周期已经不推荐使用)
* 使用过时的 ref API
* 检查意外的副作用
  * 开启严格模式的组件的生命周期会在`开发环境下执行两次`，目的是查看编写的逻辑代码被调用多次，是否会产生一些副作用
  * `生产模式下就不会被调用两次`
* ...等等



# 13.React的过渡动画

* react中没有为我们提供过渡动画的组件，需要使用三方库完成，官方推荐的是`react-transition-group`这个库

## 13.1.react-transtion-group介绍

* 我们可以通过原生的CSS来实现过渡动画，但是React社区提供了过渡动画的库
* React曾为开发者提供过动画插件react-addons-css-transtion-group，后由社区维护，形成了现在的react-transition-group
* 这个库可以帮助我们方便的实现组件的入场和离场动画

**安装**

```cmd
npm i react-transition-group --save
```

## 13.2.react-transition-group主要组件

**Transition**

* 该组件是一个和平台无关的组件(不一定要结合CSS)
* 在前端开发中，我们一般是结合CSS来完成样式，所以比较常用的是CSSTransition

**CSSTransition**

* 在前端开发中，通常使用CSSTransition,适用于`元素的显示和隐藏`

**SwitchTransition**

* 两个组件显示和隐藏时，使用该组件

**TransitionGroup**

* 将多个动画组件包裹在其中，一般用于列表中元素的动画

## 13.3.CSSTransition

* CSSTransition是基于Transition组件构建的，适用于`元素的显示和隐藏`
* CSSTransition执行过程中，有三个状态:appear(首次渲染)、enter(进入)、exit(推出)

**三种状态对应的CSS类**



`x表示的是自定义的类名 eg：<CSSTransition className="d"></CSSTransition> 那么x就要换成d-appear`



* `开始状态`，对应的类: x-appear、x-enter、x-exit
* `执行动画`，对应的类：x-appear-active、x-enter-active、x-exit-active
* `执行结束`：对应的类：x-appear-done、x-enter-done、x-exit-done

### 13.3.1.CSSTransition常见的属性

**in**：

* 触发`进入或退出的状态`，传入一个`boolean`类型的值
* 如果添加了`unountOnExit={true}`，那么组件会在执行退出动画结束后被移除掉
* `当in为true时`。触发进入状态，会添加-enter、-enter-active的class开始执行动画，当动画执行结束后，会移除这两个class，且添加一个-enter-done的class
* `当in为false时`。触发退出状态，会添加-exit、-exit-active的class开始执行动画，当动画执行结束后，会移除这两个class，且添加-enter-done的class

**className**

* 动画的class名称
* 决定了在编写css时，对应的class名称

**timeout**

* 过渡动画时间，这个时间最好和类中过渡时间保持一致
* 如果不保持一致，执行动画的时间还是按照类中的时间，但是移除类的时间会按照timeout的时间来移除
* 比如类中设置 transition：all 1s linear  属性\<CSSTransition timeout={2000}> 那么组件执行动画的时间就是1s，但是-enter、-enter-active 这些类的移除时间就是 2s

**unmountOnExit**

* 执行退出动画完毕后卸载组件

### 13.3.2.CSSTransition生命周期函数

CSSTransition提供了对应的钩子函数，可以在动画执行过程中，完成一些js 代码的操作

* `onEnter`：在进入动画之前被触发
* `onEntering`：在应用进入动画时被触发
* `onEntered`：在应用进入动画结束后被触发
* `onEnter`：在退出动画之前被触发
* `onEntering`：在应用退出动画时被触发
* `onEntered`：在应用退出动画结束后被触发

### 13.3.3.示例代码

css文件

```css
/* 首次渲染时的动画 */
.r-appear {
  transform: translateX(-550px);
}
.r-appear-active {
  transform: translate(0px);
  transition: all 1s linear;
}

/* 进入动画 */
.r-enter {
  opacity: 0;
}

.r-enter-active {
  opacity: 1;
  transition: all 1s linear;
}

/* 离开动画 */
.r-exit {
  opacity: 1;
}
.r-exit-active {
  opacity: 0;
  transition: all 1s linear;
}
```

js文件

```jsx
export default class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      isShow: true,
    }
  }
  render() {
    const { isShow } = this.state
    return (
      <div>
        <CSSTransition
          in={isShow}
          timeout={1000}
          classNames="r"
          unmountOnExit={true}
          appear
          onEnter={(e) => console.log('开始进入动画')}
          onEntering={(e) => console.log('开始执行动画')}
          onEntered={(e) => console.log('结束进入动画')}
          onExit={(e) => console.log('开始退出动画')}
          onExiting={(e) => console.log('退出执行动画')}
          onExited={(e) => console.log('结束退出动画')}
        >
          <h1>云想衣裳花想容，春风拂剑露华浓</h1>
        </CSSTransition>

        <button onClick={(e) => this.setState({ isShow: !isShow })}>
          切换
        </button>
      </div>
    )
  }
}
```

## 13.4.SwitchTransition

**SwitchTransition可以完成来年更高组件之间的切换的炫酷动画**

* 比如有一个按钮需要在on和off之间切换，希望看到on先从左侧退出，off在从右侧进入
* 这种动画在Vue中被称之为vue transition modes
* react-transition-goup中使用SwitchTransition组件来实现这种动画

**SwitchTransition中主要有一个属性，mode，值如下**

* in-out:表示新组件先进入，旧组件再移出
* out-in：表示旧组件先移除，新组件再进入

**使用**

* SwitchTransition组件里`要包裹CSSTransition或Transition组件`，不能直接包裹想要切换的元素
* SwitchTransition里面的CSSTransition或Transition组件`不在向以前那样接受in`属性来判断元素是何状态，而是用`key属性`，如果想要切换只需要保证key不同即可



css文件

```css
.r-enter {
  transform: translateX(100px);
  opacity: 0;
}
.r-enter-active {
  transform: translateX(0);
  opacity: 1;
  transition: all 1s ease;
}

.r-exit {
  opacity: 1;
}
.r-exit-active {
  transform: translateX(-100px);
  opacity: 0;
  transition: all 1s ease;
}
```

js文件

```jsx

export default class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      isShow: true,
    }
  }
  render() {
    const { isShow } = this.state
    return (
      <div>
        {/* 示例一 两个组件的切换 */}
        <SwitchTransition mode="out-in">
           {/* 只要key发生了变化即可 无论key是什么值 */}
          <CSSTransition classNames="r" timeout={1000} key={isShow ? 1 : 2}>
            {isShow ? <Home /> : <Page />}
          </CSSTransition>
        </SwitchTransition>
        {/* 示例二 文字的切换 */}
        <SwitchTransition mode="out-in">
          <CSSTransition classNames="r" timeout={1000} key={isShow ? 1 : 2}>
            <button onClick={(e) => this.setState({ isShow: !isShow })}>
              {isShow ? '退出' : '登录'}
            </button>
          </CSSTransition>
        </SwitchTransition>
      </div>
    )
  }
}
```

## 13.5.TransitionGroup

* 专门用来给列表做动画的

**注意**

`TransitionGroup 可以直接代替包裹元素` 比如下面的li是被ul包裹 ，这时候可以 直接使用TransitionGroup替换ul 但是默认情况下TransitionGroup渲染出来是div 所以可以使用component属性指定渲染出的元素内容 

css

```css
.r-enter {
  transform: translateX(-150px);
  opacity: 0;
}
.r-enter-active {
  transform: translateX(0);
  opacity: 1;
  transition: all 1s ease;
}


.r-exit {
  opacity: 1;
}
.r-exit-active {
  transform: translateX(150px);
  opacity: 0;
  transition: all 1s ease;
}
```



js

```jsx
export default class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      books: [
        {
          id: 1,
          title: '三国演义',
          author: '罗贯中',
        },
        {
          id: 2,
          title: '水浒传',
          author: '施耐庵',
        },
        {
          id: 3,
          title: '红楼梦',
          author: '曹雪芹',
        },
        {
          id: 4,
          title: '西游记',
          author: '吴承恩',
        },
      ],
      id: 5,
    }
  }
  hanldeAdd() {
    const newBooks = [...this.state.books]
    newBooks.push({
      id: this.state.id,
      title: '你查查',
      author: 'r',
    })

    this.setState({
      books: newBooks,
      id: this.state.id + 1,
    })
    // console.log(this.state.books)
  }

  handleRemove(i) {
    const newBooks = [...this.state.books]
    newBooks.splice(i, 1)
    this.setState({
      books: newBooks,
    })
  }
  render() {
    const { books } = this.state
    return (
      <div>
       {/*TransitionGroup 可以直接代替包裹元素 比如下面的li是被ul包裹 ，这时候可以 直接使用TransitionGroup替换ul 但是默认情况下TransitionGroup渲染出来是div 所以可以使用component属性指定渲染出的元素内容  */}
          <TransitionGroup>
            {books.map((item, i) => (
              <CSSTransition key={item.id} timeout={1000} classNames="r">
                <li>
                  {item.title}
                  <button onClick={(e) => this.handleRemove(i)}>X</button>{' '}
                </li>
              </CSSTransition>
            ))}
          </TransitionGroup>
 
        <button onClick={(e) => this.hanldeAdd()}>添加</button>
      </div>
    )
  }
}
```

# 14.CSS相关

## 14.1.内联样式

* **内联样式是官方推荐的一种css写法**
  * style接受一个采用`小驼峰命名`属性的`js对象`，`不是CSS字符串`
  * 且可以引用state中的状态来设置相关的样式
* **内联样式的优点**
  * 样式之间不会发生冲突
  * 可以动态获取当前的state中的状态
* **内联样式的缺点** 
  * 写法上都需要使用驼峰标识
  * 某些样式没有智能提示
  * 大量的样式会导致代码混乱
  * 某些样式无法编写(比如伪类、伪元素)

```jsx
export default class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      fontSize: 10,
      objStyle: {
        color: 'green',
        backgroundColor: 'yellow',
      },
    }
  }
  render() {
    return (
      <div>
        <button
          onClick={() => this.setState({ fontSize: this.state.fontSize + 2 })}
        >
          增加fontsize
        </button>
        <h1 style={{ color: 'red', fontSize: `${this.state.fontSize}px ` }}>
          我是哈哈哈
        </h1>
        <h2 style={this.state.objStyle}>我是奔波儿霸</h2>
      </div>
    )
  }
}  
```

## 14.2.普通的CSS

* 普通的CSS我们通常会编写到一个独立的文件，之后再引入到组件中
* **但是这种会出现一个问题，普通的css会作用于全局，相同的的样式会被层叠掉**
* 比如：A组件中有一个.c的class，B组件中也有.c的class  在A组件中引入样式，但是B组件的.c也会被污染，因为这种普通的css是作用域全局的

## 14.3.CSS Modules

* css modules并不是React特有的方案，而是所有使用了`webpack环境`的框架都能用

  * 如果想要在项目中使用它，需要在`webpack.config.js`中设置`moduls:true`

* React脚手架已经内置了css modules的配置

  * 我们只需要将`.css/.less/.scss等样式文件`，修改为`.module.css/.module.less/.moudle.scss`

* css modules确实解决了局部作用域的问题，这也是很多人喜欢在React中喜欢的一种方案

  

**缺陷**

* 所有的`className都必须使用对象`的形式来编写
* 不方便`动态的修改某些样式`，依然吸引使用内联样式



**使用**

* 将文件名改为.module.css
* 引入的方式是变量方式



App.module.css

```css
.title-1 {
  color: red;
}
```



```jsx
import appSyle from './App.module.css'

export class App extends PureComponent {
  render() {
    return (
      <div>
        <div className={appSyle['title-1']}>我是标题</div>
      </div>
    )
  }
}
```

## 14.4.CSS in JS

* 'CSS-in-JS'是指`一种模式`，其中CSS由JS生成
* 注意次功能并不是React的一部分，而是`由第三方库提供`
* CSS-in-JS通过JS来为CSS赋予一些能力，包括类似于CSS预处理器一样的`嵌套、函数定义、逻辑复用、动态修改等`
* 虽然CSS预处理器也具备某些能力，但是获取动态状态依然是一个不好处理的点

**目前比较流行的CSS-in-JS库**

* **`styled-components`**
* emotion

### 14.4.1.styled-components

**安装**

```cmd
npm i styled-components
```

#### 14.4.1.1**基本使用**

* styled-components的本质是`通过函数的调用`，最终`创建出一个组件`
  * 这个组件会被自动添加上一个不重复的class
  * styled-components会给该class添加相关的样式
* 它支持类似CSS预处理器一样的样式嵌套等功能



`样式的js文件`

```js
import styled from 'styled-components'

// styled.div 如果想让渲染的是div那么就styled.div  如果想让渲染的是一个ul那么就是styled.ul
// styled.div div其实就是一个函数  div``这种写法是模版字符串的函数调用
export const AppWrapper = styled.div`
  .section {
    background-color: #ff0;
    .title {
      color: red;
      &:hover {
        background-color: green;
      }
    }
  }
`

```

`App组件`

```jsx
import React, { PureComponent } from 'react'

import { AppWrapper } from './style'
export class App extends PureComponent {
  render() {
    return (
      // 样式中返回的是一个div 所以AppWrapper就是一个div 直接替换原本div的包裹即可
      <AppWrapper>
        <div className="section">
          <div className="title">我是标题</div>
          <p className="content">我是内容</p>
        </div>
      </AppWrapper>
    )
  }
}
```

#### 14.4.1.2.动态传入样式





**方式一：使用props和attrs（不常用）**

* styled-components支持从 外部传入属性，还可以设置默认值



样式js文件

```js
// 在调用函数的时候可以通过attrs设置一些变量
export const FooterWrapper = styled.div.attrs({
  sage: '12323',
})`
  /* 外界传入的值是在 props上  如果需要获取 需要写一个回调，styled-components会自动调用这个回调 且将props作为第一个参数传入 */
  .title {
    color: ${(props) => props.color};
    /* 因为props事在函数中 所以返回的时候判断外界有没有传入值 没有的话可以设置默认值 */
    font-size: ${(props) => props.size || '52'}px;
  }
`
```

App组件

```jsx
import { AppWrapper, FooterWrapper } from './style'
export class App extends PureComponent {
  render() {
    return (
      // 样式中返回的是一个div 所以AppWrapper就是一个div 直接替换原本div的包裹即可
      <AppWrapper>
        <div className="section">
          <div className="title">我是标题</div>
          <p className="content">我是内容</p>
        </div>
        {/* 给样式传入参数 */}
        <FooterWrapper color="blue" size="18">
          <div className="title">免责声明</div>
        </FooterWrapper>
      </AppWrapper>
    )
  }
}
```

**方式二：（常用）**

* 定一个js文件，这个文件中存放样式变量，然后引入到css-in-js的文件中使用，当某一天如果要变化样式，直接修改存放变量的js就行
* 比如现在要定义颜色 可以在一个js文件中保存颜色的变量，在css-in-js文件中使用，当某天要变化颜色了 直接改变量就行



`存放变量的js`

```js
export const primaryColor = '#af0'
export const secondColor = '#f00'

```

`css-in-js文件`

```js
import styled from 'styled-components'
import { primaryColor, secondColor } from './style/variable'

export const AppWrapper = styled.div`
  .section {
    background-color: ${primaryColor};
    .title {
      color: red;
      &:hover {
        background-color: ${secondColor};
      }
    }
  }
`
```



#### 14.4.1.3.styled高级特性

**支持样式的继承**

```js
const YMButton = styled.button`
padding:8px
`
const YMWarnButton = styled(YMButton)`
background:'#faf'
`
```

**设置主题色**

需要引入ThemeProvider组件 使用方法和Contex类似 只不过要用theme传递值

```jsx
// 引入组件
import { ThemeProvider } from 'styled-components'
export class App extends PureComponent {
  render() {
    return (
      // 样式中返回的是一个div 所以AppWrapper就是一个div 直接替换原本div的包裹即可
      <AppWrapper>
       
        {/* 通过ThemeProvider共享变量 */}
        <ThemeProvider theme={{ color: 'red' }}>
          <Home />
        </ThemeProvider>
      </AppWrapper>
    )
  }
}
```

Home -> style.js

直接通过props.theme获取即可

```js
import styled from 'styled-components'

export const HomeWrapper = styled.div`
  .title {
    color: ${(props) => props.theme.color};
  }
`
```

## 14.5.classnames库

* 这个库可以让我们像Vue那用动态的添加class
* 当然不用这个库我们也能做的，但是会做很多的判断，是代码冗余



**安装**

```cmd
npm i classnames
```

**使用**

```jsx
import classnames from 'classnames'

export class App extends PureComponent {
  render() {
    return (
      <div>
        hhh
        <div className={classnames('aaa', { bbb: true }, { ccc: false })}></div>
      </div>
    )
  }
}
```

**示例**

[更多示例](https://github.com/JedWatson/classnames)

```js
classNames('foo', 'bar'); // => 'foo bar'
classNames('foo', { bar: true }); // => 'foo bar'
classNames({ 'foo-bar': true }); // => 'foo-bar'
classNames({ 'foo-bar': false }); // => ''
classNames({ foo: true }, { bar: true }); // => 'foo bar'
classNames({ foo: true, bar: true }); // => 'foo bar'

// lots of arguments of various types
classNames('foo', { bar: true, duck: false }, 'baz', { quux: true }); // => 'foo bar baz quux'

// other falsy values are just ignored
classNames(null, false, 'bar', undefined, 0, 1, { baz: null }, ''); // => 'bar 1'
```

