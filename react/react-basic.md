# 1.React App初体验

## 1.1.React的依赖引入

* 开发React必须依赖三个库：
  * `react`：包含react所必须的核心代码
  * `react-dom`：react渲染在不同平台所需要的核心代码
  * `babel`：将jsx转换成React代码的工具
* 为什么要依赖这么多包
  * 这三个包`各司其职`，目的就是`让每一个包单纯做自己的事情`
  * 在React14版本之前是没有react-dom这个概念的，所有的功能都包含在react中，但是React 2013年推出了ReactNative，2017年推出了ReactVR，这写包中都用到了dom，所以将其单独分出来了
  * 默认情况下`开发React其实可以不使用babel`
  * 但是我们需要`自己使用React.createElement来编写源代码`，非常`繁琐`且`可读性差`，所以需要`借助babel`帮助我们`转成React.createElement`

## 1.2.创建Hello World

**使用的是html进行编写，不是脚手架**

* 注意：这里编写react代码，script必须添加type="text/babel",作用是让babel可以解析jsx的语法

```html
<body>
  <div id="root"></div>
  <script type="text/babel">

    const root = ReactDOM.createRoot(document.querySelector('#root'))
    root.render(<h1>你好章三</h1>)
  </script>
</body
```



* ReactDOM.`createRoot`函数：用于创建一个React的根，之后渲染的内容会包含在这个根中
* root.`render`函数：渲染的内容，可以是`HTML元素`，也可以是`组件`



## 1.3.组件化开发

* React中的组件可以是一个`类`，也可以是`函数`,这里先写类，后续在写函数
* 定义一个类，(`类名大写`，`组件名称`也必须`大写`，小写会被认为是HTML元素)，
* 这个类要`继承React.Component`
* 这个类中必须实现一个`render函数`，用来返回内容

```js
<body>
  <div id="root"></div>

  <script type="text/babel">
    class App extends React.Component {
      // 内容 实现一个render函数(固定的方法)
      render() {
        return (
          <div>
            <h2>哈哈哈</h2>
            <button >修改文本</button>
          </div>
        )
      }

    }

    // 将组件渲染到界面上
    const root = ReactDOM.createRoot(document.querySelector('#root'))
    root.render(<App />)

  </script>
</body>
```

### 1.3.1.数据依赖

**组件的数据定义在哪里**

* 在组件中的数据，可以分为`两类`
  * `参与界面更新的数据`：当数据变化时，需要更新组件渲染的内容
  * `不参与界面更新的数据`：当数据变化时，不需要更新组件渲染的内容

**参与界面更新的数据**

* 可以在构造函数的 `this.state` 中存放

```js
<body>
  <div id="root"></div>

  <script type="text/babel">
    class App extends React.Component {
      // 数据
      constructor() {
        super() //继承所以要这个方法

        this.state = {
          message: '你好,张三'
        }
      }

      // 内容 实现一个render函数(固定的方法)
      render() {
        return (
          <div>
            <h2>{this.state.message}</h2>
            <button >修改文本</button>
          </div>
        )
      }

    }

    // 将组件渲染到界面上
    const root = ReactDOM.createRoot(document.querySelector('#root'))
    root.render(<App />)

  </script>
</body>
```

### 1.3.2.事件绑定

* `在类中的事件函数，其实就是类的实例方法`
* 看以下的案例

```js
// 我们知道如果一个函数默认调用的话，this是window。eg：调用全局函数 foo()
// 类中的方法默认开启严格模式
// 所以下面例子中的this是undefined
// 案例一
class A {
  say() {
    console.log(this)
  }
}
const a1 = new A()
foo = a1.say
foo()  // this的值是undefined

// 在看一个案例
// 下面案列中的this也是undefined 因为引用了babel babel在转换代码的时候默认开启严格模式
<script type="text/babel">
  function bar() {
  console.log(this)
}
bar()  // undefined
</script>
```



**现在看一下react中是如何绑定事件的**

* 绑定事件子需要在元素上写。on+方法名即可，方法名首字母大写 eg：onClick

```js
  <script type="text/babel">
    class App extends React.Component {
      // 数据
      constructor() {
        super() //继承所以要这个方法
        this.state = {
          message: '你好,张三'
        }
        // 使用这种方式提前给方法绑定this
        // 那么在调用的时候就可以不用在绑定this了
        // this.handleClick = this.handleClick.bind(this)
      }
      // 方法 组件的方法就是类的实例方法
      handleClick() {
        console.log(this)
        // 通过this.setState方法更改数据，会自动的渲染视图
        // setState方法来自继承的React.Component中
        // setState做了两件事：1.更改this.state中的数据 2.调用render方法
        this.setState({
          message: '你好，李四'
        })
      }
      // 内容 实现一个render函数(固定的方法)
      // 使用babel对代码进行转换的时候 会将handleClick赋值给onClick 其实就是相当于上面的案例一那样，因为并没有给handleClick进行显示赋值，所以调用的时候 this就是undefined，所以要显示的绑定this，绑定的this是render中的this，也就是这个组件实例
      // 且 onClick={this.handleClick} 虽然是this. 但是没有调用，this的绑定是在调用时候生效
      render() {
        return (
          <div>
            <h2>{this.state.message}</h2>
            <button onClick={this.handleClick.bind(this)} >修改文本</button>
						 {/*
            如果将方法提前绑定了this，这里就不用绑定this了
            <button onClick={this.handleClick} >修改文本</button>
            */}
          </div>
        )
      }

    }
    // 将组件渲染到界面上
    const root = ReactDOM.createRoot(document.querySelector('#root'))
    root.render(<App />)

  </script>
```

# 2.JSX语法

## 2.1.认识JSX

```js
<script type="text/babel">
  const element = <div>hhhhh</div>
	const root = ReactDOM.createRoot(document.querySelector('#root'))
  root.render(element)
<script />
```

* 这段element变量的声明右侧赋值的标签语法是什么？
  * 他`不是一段字符串`，所以没有使用引号包裹
  * 它是`JSX语法`，需要被 type="text/babel"包裹这样才能被babel解析
* 什么是JSX
  * JSX是一种`javascript的语法拓展`，也在很多地方称之为`javascript XML`，因为看起来就是一段XML语法
  * 它用于`描述我们的UI界面`，并且可以将HTML融合在javascript中一起使用
  * 它`不同于Vue中的模版语法`，你`不需要专门学习模版语法中的一些指令`(如，v-for、v-if...)

​	

## 2.2.JSX编写规范

* JSX的顶层`只能有一个根元素`，所以很多时候`会在外层包裹一个div元素`
* 为了方便阅读，通常会在jsx的外层`包裹一个小括号`，这样方便阅读，且JSX可以进行换行书写
* JSX中的标签可以是`单标签`，也可以是`双标签`
  * 注意：如果是单标签必须以`/>`结尾

## 2.3.JSX的注释

* 在非UI的地方就是正常的语法注释 
* 在UI的地方的注释` {/*xxx*/}`

```js
render() {
  // 我是非UI地方的注释
    const { message } = this.state
    return (
      <div>
        {/*这里是我写的注释 <h1>{message}</h1>  */}
        {
          // 我又是注释
          //  <h1>{message}</h1>
        }
        <h1>{message}</h1>
      </div>
    )
  }
```

## 2.4.JSX嵌入变量作为子元素

**比如 \<h2>message\</h2>  message就是h2的子元素**



* `情况一`：当变量是Number、String、Array类型时，可以直接显示
* `情况二`：当变量是undefined、null、Boolean类型时，内容为空
  * 如果希望可以显示null\undefined\Boolean，那么需要转弯字符串
* `情况三`：Object类型不能作为子元素
* JSX还可以嵌入表达式
  * 运算表达式
  * 三元表达式
  * 执行一个函数

```js
   class App extends React.Component {
      constructor() {
        super()
        this.state = {
          count: 100,
          message: 'hello world',
          list: ['hh', 'xx', 'nn'],
          a: undefined,
          b: null,
          c: true,
          info: {
            name: 'zs'
          }
        }

      }
      transformList() {
        return this.state.list.join('---')
      }
      // 渲染函数
      render() {
        const { message, list, count, a, b, c, info, age } = this.state
        return (
          <div>
            {/*1. Number/Sting/Array类型的数据会直接显示出来  Array中的数据会被切割 效果类似arr.join('') */}
            <h1>{message}</h1>
            <h1>{count}</h1>
            <h1>{list}</h1>
            {/*2. undefined/null/Boolean类型的值会被忽略，也就是页面上不显示  如果想显示，需要将值转为字符串*/}
            <h1>{a + ''}</h1>
            <h1>{b}</h1>
            <h1>{c}</h1>
            {/*3.Object类型不能作为字元素进行显示，会报错  可以展示对象的内容*/}
            <h1>{info.name}</h1>

						{/*4.可以插入对应的表达式*/}
            <h1>{1 + 2 * 9}</h1>

            {/*5.可以插入三元运算符*/}
            <h1>{age >= 18 ? '成年' : '未成年'}</h1>

						{/*6.可以调用方法，获取结果*/}
            <h1>{this.transformList()}</h1>
						<h1>{this.state.list.join('---')}</h1>
          </div>
        )
      }
    }
```

## 2.5.JSX嵌入变量作为属性

**注意：变量作为属性的时候是支持对象类型的**

```js
    class App extends React.Component {
      constructor() {
        super()
        this.state = {
          title: '哈哈哈',
          imgUrl: 'https://gss0.baidu.com/-fo3dSag_xI4khGko9WTAnF6hhy/zhidao/pic/item/34fae6cd7b899e5121a2bee044a7d933c8950d11.jpg',
          isActive: true,
          styleObj: { color: "red", fontSize: "30px" }
        }

      }
      // 渲染函数
      render() {
        const { title, imgUrl, isActive, styleObj } = this.state

        //  class绑定写法一：字符串拼接
        const className = `aaa bbb ${isActive ? 'active' : ''}`
        //  class绑定写法二：将所有的class放到数组中
        const classList = ['aa', 'bb']
        if (isActive) classList.push('active')
        // class绑定写法三：使用classnames库 后续脚手架的时候补充
        return (
          <div>
            {/*1.基本绑定 */}
            <h1 title={title}>我是H2元素</h1>
            {/* <img src={imgUrl} />*/}

            {/*2.class的绑定， 最好使用className 因为class是js的关键字 有时候babel可能转换错误 */}
            <h1 className={className}>哈哈的</h1>
            <h1 className={classList.join(" ")}>黑黑的</h1>

            {/*3.style的绑定的动态绑定  属性中可以存放对象，但是内容不能*/}
            <h1 style={styleObj}>哈哈</h1>
          </div>
        )
      }
    }
```

## 2.6.JSX中的事件绑定

* 事件的`命名`采用`小驼峰`，而`不是纯小写`
* 需要通过`传入一个{}事件处理函数`，这个函数会在事件发生时被执行

```js
class App React.Component {
  constructor() {
    super()
    this.state = {}
  }
  handleClick() {}
  render() {
    return (
    	<h1 onClick={this.handleClick}></h1>
    )
  }
}
```

### 2.6.1.this绑定问题

* 当调用绑定事件的函数时，该函数内部的this是undefined，所以就无法做相关的操作
* 原因见上面 `1.3.2`

**三种解决方案，推荐使用第三种**

```js
class App extends React.Component {
      // es6的类中推出了 class fields 
      // 直接在类中声明一个变量 这个变量会作为类的实例字段
      a = '123'
      constructor() {
        super()
        this.state = {
          message: 'hello world'
        }

      }
      handleClick1() {
        console.log('btn1', this)
      }
      // 使用class fields绑定this
      handleClick2 = () => {
        console.log('btn2', this)
      }

      // 
      handleClick3() {
        console.log('btn3', this)
      }
      // 渲染函数
      render() {
        const { message } = this.state
        return (
          <div>
            {/*1. this绑定方式一：bind绑定*/}
            <button onClick={this.handleClick1.bind(this)}>按钮1</button>
            {/*2. this绑定方式二：ES6 class fields
              因为this.handleClick2是一个箭头函数，不存在this 当调用的时候会去上层作用域找
              (函数的作用域与定义位置有关和调用位置无关)
              于是就找到了App这个类 但是这个类已经被实例化了 所以this就是当前的实例
            */}
            <button onClick={this.handleClick2}>按钮2</button>

            {/*3. this绑定方式三：直接传入一个箭头函数
              点击按钮会调用箭头函数，进而执行this.handleClick3() 因为箭头函数没有this 
              去上层作用域找，this就是render函数的this
              调用handleClick3  此时handleClick3的this就是render函数的this 也就是当前的实例
            */}
            <button onClick={() => this.handleClick3()}>按钮3</button>
          </div>
        )
      }
    }
```

### 2.6.2.事件传参

**推荐使用箭头函数的传参方式**

```js
 <script type="text/babel">
    // 问题一：为什么不传递参数 event是第一个参数，传递参数event是最后一个参数
    // 下面案列 我们给a函数永久绑定了this 且传递了参数，等到我们调用b的时候也会给他传递一个参数，所以b调用时候传递的参数会在最后
    // 我们在绑定事件的时候传入了自定义的属性 this.btn3.bind(this, 'zs', 18) 当触发事件调用函数的时候 react会调用函数且传入event 因为绑定this的时候我们手动传入了两个参数 所以event就会在最后
    // 当我们在绑定事件的时候没有传递属性，当触发事件调用函数的时候 react会调用函数 传入event，所以event在第一个
    // function a() { }
    // const b = a.bind(this, 'a', 'b')
    // b('event')

    class App extends React.Component {
      constructor() {
        super()
        this.state = {
          message: 'hello world'
        }

      }
      btn1(e) {
        console.log(e)
      }
      btn2(e) {
        console.log(e)
      }

      btn3(name, age, e) {
        console.log(e, name, age)
      }
      btn4(e, name, age) {
        console.log(e, name, age)
      }
      // 渲染函数
      render() {
        const { message } = this.state
        return (
          <div>
            {/*1.传递event参数
              如果使用的是方式一和方式二的事件绑定 react会将event进行二次封装然后传递给函数的第一个参数
              如果使用方式三的事件绑定， 因为调用了箭头函数 所以event会在箭头函数的第一个参数中 我们只需要手动传入即可获取到event
            */}
            <button onClick={this.btn1.bind(this)}>按钮1</button>
            <button onClick={(e) => this.btn2(e)} >按钮2</button>

            {/*2.传递自定义参数 不推荐使用bind的形式传参会有问题一的问题  推荐使用箭头函数的绑定事件和传参方式，这样我们可以明确的控制event的位置*/}
            <button onClick={this.btn3.bind(this, 'zs', 18)}>按钮3</button>
            <button onClick={(e) => this.btn4(e, 'zs', 19)}>按钮4</button>
          </div>
        )
      }
    }

    const root = ReactDOM.createRoot(document.querySelector('#root'))
    root.render(<App />)
  </script>
```

## 2.7.JSX中的条件渲染

**在Vue中，通过v-show、v-if指令来决定哪些要渲染,在React中，`所以的条件都和普通的Javascript代码一致`**



**常见的条件渲染方式**

* 条件判断语句
* 三元运算符
* 逻辑判断

```js
class App extends React.Component {
      constructor() {
        super()
        this.state = {
          isReady: true,
          friend: {
            name: 'zs',
            age: 19
          }
        }

      }
      // 渲染函数
      render() {
        const { isReady, friend } = this.state
        // 1. 条件判断方式一： 使用if进行条件判断
        let showElement = null
        if (isReady) {
          showElement = <h1>我已经准备好了</h1>
        } else {
          showElement = <h1>我目前尚未准备</h1>
        }


        return (
          <div>
            {/*1.条件判断方式一： 使用if进行条件判断*/}
            {showElement}

            {/*2.条件判断方式二： 使用三元运算符判断*/}
            {isReady ? <h1>准备</h1> : <h1>不行</h1>}

            {/*3.条件判断方式三： 使用逻辑与判断*/}
            {/*场景：比如friedn是服务端返回来的，初始化的时候friedn是null */}
            {friend && <h1>{friend.name + '=====' + friend.age}</h1>}
          </div>
        )
      }
    }
```

## 2.8.JSX中的列表渲染

**React中是没有类似Vue中的v-for的语法的，需要通过Javascript的方式组织数据**



**注意列表渲染中要给每个item绑定一个key 作用和vue的key一样**



**常用的展示列表的方式**

* 使用数组的map函数 
* 使用数组的filter方法
* 使用数组的slice方法
* 等等 

```js
 class App extends React.Component {
      constructor() {
        super()
        this.state = {
          books: [
            { title: '我与地坛', price: 39.29 },
            { title: '朝花夕拾', price: 29.29 },
            { title: '边城', price: 32.19 },
          ]
        }

      }
      // 渲染函数
      render() {
        const { books } = this.state
        const filterBooks = books.filter(item => item.price > 30)
        return (
          <div>
            <h1>书籍列表</h1>
            <ul>
              {books.map(item => <li key={item.title}>{item.title}---{item.price}</li>)}
            </ul>
            <h1>便宜书籍列表</h1>
            <ul>
              {filterBooks.map(item => <li key={item.title}>{item.title}---{item.price}</li>)}
            </ul>
          </div>
        )
      }
    }

```

