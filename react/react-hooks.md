# 1.介绍

## 1.1.为什么需要Hook

* `Hook`是React `16.8`的新增特性，它可以让我们`不编写类组件`的情况下`使用state以及其它的React特性`(eg:生命周期)
* 换句话来说`Hook弥补了函数组件的缺陷`，比如没有声明周期，不能保存状态(更改数据无法刷新页面)等问题
* 所以，Hook出现之前，通常会使用类组件，类组件的优缺点看下面

## 1.2.类组件的优点

* class组件可以`定义自己的stat`e，用来保存组件自己内部的状态
* class组件有`自己的生命周期`
* class组件可以在`状态改变时`只会`执行render函数`以及我们希望重新调用的生命周期函数
  * 函数式组件在重新渲染时，整个函数都会被执行

## 1.3.类组件的缺陷

* 随着业务的增加，`类组件的逻辑会变的复杂`，难以理解
* `this指向问题`，this自古以来就是js的难题
* `组件复用状态很难`，需要使用高阶组件来实现复用，有时候逻辑很复杂，往往是高阶组件嵌套高阶组件，难以理解

## 1.4.Hook的使用场景

* Hook的出现`基本可以替代之前所有使用class组件的地方`
* 如果是一个`旧的项目`，并不需要`直接将所有的代码重构为Hooks`，因为它`完全向下兼容`，可以渐进式的来使用它
* Hook`只能在函数组件中使用`，不能在类组件，或函数组件之外的地方使用

# 2.Hooks

## 2.1.Hook使用规则

* 只能在`函数组件`的最`外层调`用Hook。`不能`在`循环`、`条件判断`语句中调用

```jsx
import { memo, useState } from 'react'

function CounterH() {
  console.log(1)
  const [counter, setCounter] = useState(20)

  // 只能在函数组件最外层调用 下面的调用不在最外层了 不能这样写
  // if (true) {
  //   const [counter, setCounter] = useState(20)
  // }
}
```



## 2.2.useState

**State Hook的API就是useState**



* useState来自react，需要从react中导入，他是一个hook

**参数**

* 接受一个参数作为`初始化值`，如果不设置初始化值为undefined

**返回值**

* 返回一个`数组`，包含两个元素
* `元素一`：当前的状态值(第一次调用会使用初始化值)
* `元素二`：设置状态值的函数



**FAQ**

* **为什么叫useState而不叫createState**
  * create 可能不是很准确，因为`state只在组件首次渲染的时候被创建`
  * 在`下一次重新渲染时`，`useState返回给我们当前的state`
  * 这也就是Hook的名字总是以use开开头的一个原因



**结果**

当使用元素二改变状态后，会使用最新的状态重新渲染

```jsx
import { memo, useState } from 'react'

function CounterH() {
  console.log(1)
  const [counter, setCounter] = useState(20)

  return (
    <div>
      <h2>Hook 当前计数:{counter}</h2>
      <button onClick={(e) => setCounter(counter + 1)}>+1</button>
      <button onClick={(e) => setCounter(counter - 1)}>-1</button>
    </div>
  )
}
```



## 2.3.useEffect

**useEffect可以模拟class中的生命周期，但是比生命周期更加强大**

* **useEffect的解析**：
  * 通过useEffect的Hook，可以`告诉React需要在渲染后执行某些操作`
  * useEffect要求`传入一个回调函数`，在React`执行更新DOM操作后`，`就会回调这个函数`
  * 默认情况下，`无论是第一次渲染之后`，还是每次更新之后，都会`执行这个回调函数`
* **使用场景**
  * 比如发送网络请求
  * 比如订阅事件
  * ...

### 2.3.1.需要清除的Effect

上面提到可以在useEffect中订阅事件，那么在哪里进行取消订阅呢？



**useEffect中可以返回一个函数**

* 这是`effect可选的清除机制`。`每个effect都可以返回一个清除函数`
* 如此`可以将添加和衣橱订阅的逻辑放在一起`，他们都属于effect的一部分



* **React会在组件更新和卸载的时候 `先执行清除操作(useEffect返回的回调)` ，`再执行effect中的内容`**

### 2.3.2.使用多个Effect

* 使用Hook的其中一个目的就是解决class中生命周期经常将很多的逻辑放在一起的问题
  * 比如网络请求，事件监听，手动修改DOM，这些往往都会放在componentDidMount中





* **使用Effect Hook 可以将他们分离到不同的useEffect中**





* **Hook允许我们按照代码的用途分离他们**，而不是像生命周期函数那样：
  * React将`按照effect声明的顺序依次执行组件中的每一个effect`

### 2.3.3.代码示例

```react
const App = memo(() => {
  const [counter, setCounter] = useState(10)
  // 使用一个useEffect
  useEffect(() => {
    // 当组件渲染完毕后会执行这个回调
    document.title = counter
    console.log('添加订阅')

    // 如果返回一个回调 那么每次更新和卸载的时候都会调用这个回调
    // 可以在这个清除函数中做一些取消订阅/取消监听的操作
    return () => {
      console.log('取消订阅')
    }
  })
  // 再使用一个useEffect
  useEffect(() => {
    console.log('执行网络请求')
    return () => {
      console.log('操作dom')
    }
  })

  return (
    <div>
      <h2>当前计数：{counter}</h2>
      <button onClick={(e) => setCounter(counter + 1)}>+1</button>
    </div>
  )
})

```

### 2.3.4.Effect性能优化



上面说到每次渲染都会执行effect，比如我在当前effect中执行的是从服务器获取数据的操作，那么每次执行渲染操作，都会发送一次请求，这样显然是不合理的



所以，如果某些代码只希望执行一次可以按照下面的解决方案



**注意**：不管依赖谁 在组件第一次加载的时候，useEffect中的内容都会执行一次，否则没有意义了



**解决**

* useEffect`实际上有两个参数`
* 参数一：`执行的回调函数`
* 参数二：是一个数组，表示该`useEffect在哪些state发生变化时才重新执行`
  * 说白了就是当前useEffect依赖谁，类似于computed，只有依赖变化了才执行

```react
const App = memo(() => {
  const [counter, setCounter] = useState(10)
  const [message, setMessage] = useState('你好')

  // 只要页面渲染都会执行
  useEffect(() => {
    console.log('修改title')
  })

  // 依赖message 当message 变化才执行
  // 参数二是一个数组，可以依赖多个 eg:[message,counter]
  useEffect(() => {
    console.log('操作DOM')
  }, [message])

  // 不依赖任何，只有当组件加载的时候执行一次
  useEffect(() => {
    console.log('发送网络请求，获取服务器的数据')

    // 因为该useEffect不依赖任何，所以清除的effect也只会在当前组件被卸载时候执行
    return () => {
      console.log('组件被卸载时执行一次')
    }
  }, [])

  return (
    <div>
      <h2>当前计数：{counter}</h2>
      <button onClick={(e) => setCounter(counter + 1)}>+1</button>
      <h2>当前msg：{message}</h2>
      <button onClick={(e) => setMessage('我很好')}>change msg</button>
    </div>
  )
})
```

## 2.4.useContext

之前的学习中，在函数组件中使用context是很不方便的，可以使用useContext Hook来简化操作



**注意**： 当前组件依赖的Context数据发生变化时，该组件也会重新渲染

```react
import React, { memo, useContext } from 'react'
import { ThemeContext, UserContext } from './context'

const App = memo(() => {
  //如果 UserContext ThemeContext 传入的数据发生变化 那么当前组件也会重新渲染
  const user = useContext(UserContext)
  const theme = useContext(ThemeContext)
  return (
    <div>
      <h2>展示userContext的数据:{user.name}</h2>
      <h2>展示ThemeContext的数据:{theme.color}</h2>
    </div>
  )
})

```

## 2.5.useReducer(了解)

* **这个不是redux的替代品**
* useReducer`仅仅是useState的一种替代方案`
  * 某些场景下，`如果state的处理逻辑比较复杂`，可以通过useReducer来对其进行拆分
  * 比如现在点击按钮，既要让counter+1，又要改变message，如果使用useState那么就需要两个，一个是操作counter的useState，一个是操作message的useState
  * 如果操作的state更多，那么就需要更多的useState
  * 可以使用userReducer，只需要一个即可，因为操作的都会放到一个state中(就像redux的state一样)
* 开发中不推荐这样，如果遇到这样的需求，建议使用redux

```react
import React, { memo, useReducer } from 'react'

function reducer(state, action) {
  switch (action.type) {
    case 'add':
      return {
        ...state,
        counter: state.counter + action.num,
        message: '我是add',
      }
    default:
      return {
        ...state,
        counter: state.counter - action.num,
        message: '我是sub',
      }
  }
}

const App = memo(() => {
  const [state, dispatch] = useReducer(reducer, { counter: 1 })

  return (
    <div>
      <h1>
        counter:{state.counter},msg:{state.message}
      </h1>
      <button onClick={(e) => dispatch({ type: 'add', num: 1 })}>+</button>
      <button onClick={(e) => dispatch({ type: 'sub', num: 1 })}>-</button>
    </div>
  )
})

export default App
```

## 2.6.useCallback

* useCallback`实际目的是为了进行性能的优化`
* useCallback`会返回一个函数的memoized(记忆的)值（返回一个有记忆的回调函数）`
* 在`依赖不变的情况下，多次定义的时候，返回值是相同`的





**性能优化**

* 当需要`将一个函数传递给子组件时`，最好`使用useCallback进行优化`，将优化后的函数，传递给子组件

```react
// 解析
// 看下面案例

// 当点击加一操作之后，count进行了更新 重新执行了app组件完成重新渲染 这时<YMIncrement>子组件接收了increment函数，所以也会重新渲染一次，这都是正常的 因为app组件重新渲染了 所以increment函数又被重新定义了一次 子组件接收到新的参数所以也会重新渲染

// 当点击change message按钮之后 改变了message  此时子组件也会被重新渲染 道理和上面一样 因为message发生了变化 所以app重新渲染 increment函数重新定义 子组件拿到新的参数又会被重新渲染

// 但是这样是有损性能的 子组件依赖的是count相关的 并不是message相关的 所以当message变化不应该去影响这个子组件  这个案例中没有那么复杂 所以性能看不出有啥损耗 如果真是开发中有100个子组件都需要使用这个函数那么这100个子组件都会渲染，就会影响很大了

// 使用useCallback对increment进行优化，他依赖count，只有count变化导致的重新渲染才会使得increment的回调foo重新定义，否则就会使用之前定义的，这时因为increment没有被重新定义，所以子组件不会重新渲染

const YMIncrement = memo((props) => {
  console.log('YMIncrement渲染了')
  return (
    <div>
      <button onClick={props.increment}>YMIncrement+1</button>
    </div>
  )
})

const App = memo(() => {
  const [count, setCount] = useState(0)
  const [message, setMessage] = useState('hello')

  // 未被优化的函数
  // const increment = () => {
  //   setCount(count + 1)
  // }

  // 使用useCallback优化的函数
  const increment = useCallback(
    function foo() {
      console.log(count)
      setCount(count + 1)
    },
    [count]
  )

  const changeMessage = () => {
    setMessage(Math.random())
  }

  return (
    <div>
      <h2>count:{count}</h2>
      <button onClick={(e) => increment()}>+1</button>
      <YMIncrement increment={increment} />
      <h2>message:{message}</h2>
      <button onClick={(e) => changeMessage()}>change message</button>
    </div>
  )
})


```

**必包陷阱问题**

```react

// 必包陷阱问题
// 现在如果我们将代码写成下面这样，increment表示不依赖任何，即，即使页面重新渲染 increment中的foo也不会被重新定义(解决了一个函数不会被重复定义问题？？)  但是会有问题 +1的时候count变成1之后就不会累加了
// 原因是 因为不依赖任何 所以 传入的回调foo就不会被再次定义 当点击+1 app渲染 但是foo不会被重新定义   虽然这次更新后count已经变成了1 但是foo内部的count是0 他引用的依旧是第一次app的count，0，所以就是点击很多次+1 foo内部的count依然是0  所以页面上count永远是0+1 =1


 const increment = useCallback(function foo() {
   console.log(count)  // 永远是0
   setCount(count + 1)
 }, [])


// 所以使用useCallback的时候添加一个依赖 当依赖发生变化 useCallback的回调会被重新定义
// 所以说useCallback并不是让函数只定义一次这个浅薄且有误的理解
```

**进一步优化**

* 上面的优化方案 如果添加依赖，当依赖发生变化时，回调函数还是会被重新定义，如果不添加依赖，虽然回调函数不会被多次定义，但是会存在必包陷阱的问题
* 可以使用useRef解决这个问题
  * useRef在组件多次渲染的时候，返回的是同一个值

```react
  // 虽然可以使用添加依赖的方式解决 必包陷阱问题
  // 但是当改变count foo还是会多次定义
  // 使用 useRef解决
  // useRef，在组件多次渲染时，返回的是同一个值
  const countRef = useRef()
  countRef.current = count
  const increment = useCallback(function foo() {
    console.log(countRef.current)
    setCount(countRef.current + 1)
  }, [])
```



**总结**

* 使用useCallback和不使用useCallback`在函数定义的使用时不会带来性能的优化`(可以使用useRef配合来完成优化)
* 使用useCallback和不使用useCallback定义的函数`在传递给子组件可以带来性能的优化`
* 通常使用useCallback的目的是不希望子组件进行多次渲染，并不是为了函数进行缓存

## 2.7.useMemo

* useMemo实际的目的也是为了性能优化
* useMemo`返回的也是一个memoized(记忆的)值`
* 在`依赖不变的情况下，多次定义的时候返回的值是相同的`
* 和useCallback的区别是 `useCallback返回的是一个记忆的回调函数`，`useMemo返回的是一个记忆的值`

**案例一**

```react
// 案例中 当点击计数器操作 count发生变化，组件被重新渲染，组件内部的所有东西都会被重新执行一遍，那么calcTotal这个函数也被重新执行
// 但是calcTotal函数与count没有半毛钱关系，所以没必要执行
// 使用useMemo 只有当依赖发生了变化时 才会重新返回一次结果

const calcTotal = (num) => {
  console.log('calcTotl重新执行')
  return num * 2
}

const App = memo(() => {
  const [count, setCount] = useState(0)

  // const calcNum = calcTotal(20)
  // 不依赖任何，当组件重新渲染 calcTotal函数也不会被重新执行 括号内可以写依赖的内容,空表示不依赖任何
  const calcNum = useMemo(() => calcTotal(20), [])

  function handleClick() {
    setCount(count + 1)
  }

  return (
    <div>
      <h2>total:{calcNum}</h2>
      <button onClick={(e) => handleClick()}>count:{count}</button>
    </div>
  )
})
```

**案例二**

```react
// 案例中 当点击计数器操作，count发生变化，组件被重新渲染 info又被重新生成了一次 是一个新对象，子组件中接受到新的值 所以重新渲染
// 其实子组件是不用重新渲染的 避免性能浪费
// 使用useMemo 只有当依赖发生了变化时 才会重新返回一次结果

const YmUser = memo((props) => {
  console.log('子组件渲染')
  return <h2>use:{props.info.name}</h2>
})

const App = memo(() => {
  const [count, setCount] = useState(0)

  function handleClick() {
    setCount(count + 1)
  }

  // 使用useMemo对子组件渲染进行优化
  // const info = { name: 'zs' }
  const info = useMemo(() => ({ name: 'zs' }), [])

  return (
    <div>
      <button onClick={(e) => handleClick()}>count:{count}</button>
      <YmUser info={info} />
    </div>
  )
})
```

## 2.8.useRef

* `useRef返回一个ref对象，`
* 返回的ref对象在`组件的整个生命周期是保持不变的`
  * 不然当前组件重新渲染几次，这个ref对象始终是同一个

**常用的用法**

* 获取DOM元素、或者组件(但是需要class组件)
* 保存一个数据，解决必包陷阱问题，可参考useCallback案例

```react
import React, { memo, useRef } from 'react'

const App = memo(() => {
  const titleRef = useRef()

  return (
    <div>
      <h2 ref={titleRef}>Hello 拿捏</h2>
      <button onClick={(e) => console.log(titleRef.current)}>查看dom</button>
    </div>
  )
})

export default App

```

## 2.9.useImperativeHandle

**不常用**

* 通过useImperativeHandle可以只暴露固定的操作

```react
// 如下案例场景是， 父组件想要操作子组件的input框，父组件拿到子组件input元素后可以做任何操作，但是子组件却希望父组件只能做获取焦点事件
// 这时候就需要借助useImperativeHandle，useImperativeHandle返回的是一个对象，父组件操作的实际是这个返回的对象，父组件只能操作该对象中有的方法

// 子组件
const YMIpt = memo(
  forwardRef((props, ref) => {
    const iptRef = useRef()

    useImperativeHandle(ref, () => {
      return {
        focus() {
          // 操作自己的input
          iptRef.current.focus()
        },
      }
    })

    return (
      <div>
        <input ref={iptRef} type="text" />
      </div>
    )
  })
)
// 父组件
const App = memo(() => {
  const iptRef = useRef()

  function handleClick() {
    iptRef.current.focus()
    // 无效的操作 子组件没提供该方法
    iptRef.current.value = ''
  }
  return (
    <div>
      <YMIpt ref={iptRef} />
      <button onClick={handleClick}>操作DOM</button>
    </div>
  )
})
```

## 2.10.useLayoutEffect

* useLayoutEffect和useEffect很相似，只有一点区别
  * `useEffect`会在`渲染内容更新到DOM`上`之后`执行，不会阻塞DOM的更新
  * `useLayoutEffect`会在`渲染的内容更新到DOM`上`之前`执行，会阻塞DOM的更新
* 官方给的建议是：`useLayoutEffect` 可能会影响性能。尽可能使用 [`useEffect`](https://zh-hans.react.dev/reference/react/useEffect)。
* 用法：**useLayoutEffect(setup, dependencies?)**

```react
// 下面案例 点击按钮设置count的值为0
// 如果在useEffect中进行判断  因为useEffect是在屏幕出现内容之后执行，所以会先设置成0，然后才设置成判断的值，会有一个闪烁的效果
// 如果在useLayoutEffect中判断， 因为useLayoutEffect是在屏幕出现内容之前执行，所以页面不会显示0，也不会有闪烁的效果
const App = memo(() => {
  const [count, setCount] = useState(99)
  // 当内容显示到屏幕上之后执行，会阻塞DOM
  // useLayoutEffect(() => {
  //   console.log('useLayoutEffect：我第二执行')
  //   if (count === 0) {
  //     setCount(Math.random() + 99)
  //   }
  // })

  // 当内容显现到屏幕上之后才执行 不会阻塞DOM
  useEffect(() => {
    console.log('useEffect：我第三执行')
    if (count === 0) {
      setCount(Math.random() + 99)
    }
  })

  console.log('app render:我第一执行')

  return (
    <div>
      <h1>count:{count}</h1>
      <button onClick={(e) => setCount(0)}>click</button>
    </div>
  )
})
```

## 2.11.自定义Hook

* 自定义Hook本质上只是一种函数代码逻辑的抽取，严格意义上来说，他并不算React的特性
* 案例在另一个代码库中

## 2.12.redux中的Hook

* 在之前的redux开发中，为了让组件和redux结合起来，使用了react-redux中的conne
  * 但是这种方式 必须使用高阶函数结合返回的高阶组件
  * 且必须编写：mapStateToProps和mapStateToDispatch映射函数 比较麻烦
* 在Redux7.1开始，`提供了Hook的方式`简便了使用stroe数据的写法（只是换了使用store的方式 创建store还是和之前一样）
* 使用`useSelector`和`useDispatch`两个hook来完成`读取和操作`redux中数据
* 还可以使用useStore来获取整个store（不推荐）





**useSelector 和  useDispatch**

* `useSelector`的作用是将sate映射到组件中，有两个参数
  * ·**参数一**：将state映射到需要的数据中
    * `需要传入一个回调`，回调函数的`参数`是stroe的`state`
  * **参数二**：可以进行比较来决定是否组件重新渲染
* `useDispatch`作用就是返回一个dispatch函数，用来派发事件

```react
import React, { memo } from 'react'
import { shallowEqual, useDispatch, useSelector } from 'react-redux'
import { addNumberAction } from './store/features/counter'

// useSelector第二个参数的作用
// 案例中 在APP和Home组件中 都是用了store中的数据  App中使用的是counter Home中使用的是message
// 现在 在App中派发一个事件去修改counter  这时候Home组件重新渲染了 这是有损性能的 因为Home中压根没用到counter
// 原因是因为 useSelector Hook 在不传入第二个参数的情况下 默认会监听store中的所有数据，如果有数据变化了就会使得组件重新渲染
// 解决 只需要在使用useSelector的时候传入第二个参数即可  这个参数可以是函数然后在里面判断时候要渲染 这个不常用     还可以使用react-redux 提供的shallowEqual函数
// shallowEqual函数会做一个浅层的比较 从而判断是否要重新渲染组件
// 添加完第二个参数后 就不会发生案例的情况了


const App = memo(() => {
   // 1.使用useSelector将redux中store的数据映射到组件内部
  const { count } = useSelector(
    (state) => ({
      count: state.counter.counter,
    }),
    shallowEqual
  )
  // 2.获取dispatch
  const dispathc = useDispatch()

  function handleClick(num) {
    // 3.派发dispatch
    dispathc(addNumberAction(num))
  }
  console.log('App render')
  return (
    <div>
      <h1>count:{count}</h1>
      <button onClick={(e) => handleClick(1)}>+1</button>
      <Home />
    </div>
  )
})



const Home = memo(() => {
  console.log('Home render')
  const { message } = useSelector(
    (state) => ({
      message: state.counter.message,
    }),
    shallowEqual
  )
  return (
    <div>
      <h2>Home:{message}</h2>
    </div>
  )
})

export default App

```

## 2.13.useId

学完ssr之后再补充

## 2.14.useTransition

* 官网解释：返回一个状态值表示过渡任务的等待状态，以及一个启动该过渡任务的函数
* 个人理解：**实际上就是告诉react对于某部分任务的更新优先级较低，可以稍后进行更新**



**只有当数据量很大的时候适合使用这个 像下面案例中那样**



**返回值**

* 返回一个数组 
* 第一项是一个boolean 表示：`过渡的状态`
  * 默认状态是false。如果启动了过渡函数 该状态是true。
* 第二项是一个函数 表示`启动该过渡任务`
  * 这个函数接受一个回调

```react
import React, { memo, useState, useTransition } from 'react'
import { users } from './namesArray'
import './style.css'

// useTransition作用
// 案例中 有1万条数据用于展示在页面中 当我们输入时会触发input事件，会从1万条数据中匹配输入的内容
// 会看到这么一个现象 当我们输入的时候，按下键盘，此时界面上会停顿约1s之后才会将内容显示在输入框中  原因是当输入的时候会触发input事件 就会枚举这1万条数据 这就会导致页面出现上述现象 这会让用户感到很不舒服
// 正确做法应该是 先显示输入的内容 在执行input事件  所以可以借助于useTransition 当输入的时候 告诉react  input事件中setTransition包裹的内容优先级较低 稍后进行更新

const App = memo(() => {
  const [infos, setInfos] = useState(users)
  const [pending, setTransition] = useTransition()

  function handleInput(e) {
    setTransition(() => {
      const target = e.target.value
      const filterNames = users.filter((item) => item.username.includes(target))
      setInfos(filterNames)
    })
  }

  return (
    <div>
      <h1>names</h1>

      <div className="search">
        <input
          type="text"
          placeholder="请输入要搜索的名称🔍"
          onInput={(e) => handleInput(e)}
        />
        {/* <button>search</button> */}
      </div>
      {pending ? (
        <h1>loading...</h1>
      ) : (
        <ul>
          {infos.map((item) => (
            <li key={item.userId}>
              <div className="name">{item.username}</div>
              <div className="info">
                {/* <img src={item.avatar} alt="" /> */}
                <span>
                  <i>email:</i> {item.email}
                </span>
              </div>
            </li>
          ))}
        </ul>
      )}
    </div>
  )
})

export default App


```

## 2.15.useDeferredValue

* 官方解释：**useDeferredValue接受一个值，并且返回该值的新副本，该副本将推迟到更紧急地更新之后**
* 其实useDeferredValue效果和useTransition是一样的，`只不过useDeferredValue需要接受一个值且返回的是该值的副本，react在渲染的时候发现是useDeferredValue的副本之后 就会推迟更新`
* 所以如果想要做loading效果选择使用useTransition

```react
import React, { memo, useDeferredValue, useState } from 'react'
import { users } from './namesArray'
import './style.css'
// useDeferredValue作用
// 作用和useTransition作用一样 只不过返回的是传入值的副本
// 案例中 有1万条数据用于展示在页面中 当我们输入时会触发input事件，会从1万条数据中匹配输入的内容
// 会看到这么一个现象 当我们输入的时候，按下键盘，此时界面上会停顿约1s之后才会将内容显示在输入框中  原因是当输入的时候会触发input事件 就会枚举这1万条数据 这就会导致页面出现上述现象 这会让用户感到很不舒服
// 正确做法应该是 先显示输入的内容 在执行input事件  所以可以借助于useDeferredValue 当react发现更新的是useDeferredValue产生的副本时候，就会延迟更新该副本做造成的渲染
const App = memo(() => {
  const [infos, setInfos] = useState(users)
  const deferredValue = useDeferredValue(infos)

  function handleInput(e) {
    const target = e.target.value
    const filterNames = users.filter((item) => item.username.includes(target))
    setInfos(filterNames)
  }

  return (
    <div>
      <h1>names</h1>

      <div className="search">
        <input
          type="text"
          placeholder="请输入要搜索的名称🔍"
          onInput={(e) => handleInput(e)}
        />
        {/* <button>search</button> */}
      </div>

      <ul>
        {deferredValue.map((item) => (
          <li key={item.userId}>
            <div className="name">{item.username}</div>
            <div className="info">
              {/* <img src={item.avatar} alt="" /> */}
              <span>
                <i>email:</i> {item.email}
              </span>
            </div>
          </li>
        ))}
      </ul>
    </div>
  )
})

export default App
```

