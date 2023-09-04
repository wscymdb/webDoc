# 1.安装

**这里只针对web端的安装和使用**

```cmd
npm i react-router-dom
```

# 2.基本使用

react-router中提供了两种创建路由的组件BrowserRouter和HashRouter，

想要对哪些组件开启路由，就需要将其包裹，其和其后代组件就可以使用路由了,一般会写在render时



**BrowserRouter和HashRouter**

* `BrowserRouter`使用`history`模式
* `HashRouter`使用`hash`模式

```jsx
import { HashRouter } from 'react-router-dom'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <HashRouter>
    <App />
  </HashRouter>
)
```

## 2.1.路由映射配置

* **Routes**：包裹所有的Route，在其中匹配一个路由
  * Router5x使用的是Switch组件
* **Route**：Route用于路径的匹配
  * `path`：用于设置匹配到的路径
  * `element`：设置匹配到路径后，渲染的组件
    * Router5.x使用的是component属性
  * exact：精确匹配
    * 比如路径一匹配的是/ 路径二匹配的是/home 那么使用路径一的时候不加该属性可能会匹配到路径二
    * Router6.x不在支持该属性，不需要这个了

```jsx
export class App extends PureComponent {
  render() {
    return (
      <div className="app">
        <div className="header">
         header
        </div>
        <div className="main">
          {/* 创建路由映射关系 */}
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/home" element={<Home />} />
            <Route path="/about" element={<About />} />
          </Routes>
        </div>
        <div className="footer">footer</div>
      </div>
    )
  }
}
```

## 2.2.路由配置和跳转

* **Link和NavLink**
  * 通常路径的跳转使用Link组件，最终会被渲染成a元素
  * NavLink是在Link基础上增加了一些样式属性
  * `to属性`：Link中最重要的属性，用于设置跳转到的路径

```jsx
export class App extends PureComponent {
  render() {
    return (
      <div className="app">
        <div className="header">
          <nav>
            <Link to="/home"> to home</Link>
            <Link to="/about"> to about</Link>
          </nav>
        </div>
      </div>
    )
  }
}

```

## 2.3.Navigate重定向

* **Navigate**用于路由的重定向，当这个组件出现时，就会执行跳转对应的to路径中
  * Router5.x使用的是Redirect



**案例一：/ 重定向**

```jsx
export class App extends PureComponent {
  render() {
    return (
      <div className="app">
        <div className="header">
          <nav>
            <Link to="/home"> to home</Link>
            <Link to="/about"> to about</Link>
          </nav>
        </div>
        <div className="main">
          {/* 创建路由映射关系 */}
          <Routes>
            <Route path="/" element={<Navigate to="/home" />} />
            <Route path="/home" element={<Home />} />
          </Routes>
        </div>
        <div className="footer">footer</div>
      </div>
    )
  }
}
```



**案例二**

当isLogin为true跳转页面

```js
export class Login extends PureComponent {
  constructor() {
    super()
    this.state = {
      isLogin: false,
    }
  }
  login() {
    this.setState({
      isLogin: true,
    })
  }
  render() {
    const { isLogin } = this.state
    return (
      <div>
        <h1>login</h1>
        {isLogin ? (
          <Navigate to="/home" />
        ) : (
          <button onClick={(e) => this.login()}>登录</button>
        )}
      </div>
    )
  }
}
```

**案例三：匹配notFound页面**

```jsx
export class App extends PureComponent {
  render() {
    return (
      <div className="app">
        <div className="header">
          <nav>
            <Link to="/home"> to home</Link>
            <Link to="/about"> to about</Link>
          </nav>
        </div>
        <div className="main">
          {/* 创建路由映射关系 */}
          <Routes>
            <Route path="/" element={<Navigate to="/home" />} />
            <Route path="/home" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/login" element={<Login />} />
            {/* *表示通配符 上面的路径都匹配不到的时候会执行这个路由  */}
            <Route path="*" element={<NotFound />} />
          </Routes>
        </div>
        <div className="footer">footer</div>
      </div>
    )
  }
}

```

## 2.4.路由嵌套

* `第一步`，父级路由包裹子路由
  * Router5.x 子路由直接写在父路由对应的页面的位置
* `第二步`，在父路由对应的组件中使用`<Outlet />` 进行占位，表示子组件要渲染的位置



App.jsx

```jsx
export class App extends PureComponent {
  render() {
    return (
      <div className="app">
        <div className="main">
          {/* 创建路由映射关系 */}
          <Routes>
            <Route path="/" element={<Navigate to="/home" />} />
            {/* 路由嵌套 */}
            <Route path="/home" element={<Home />}>
              {/* 跳转到home 自动重定向到/home/reco */}
              <Route path="/home" element={<Navigate to="/home/reco" />} />
              <Route path="/home/reco" element={<HomeRecommand />} />
              <Route path="/home/rank" element={<HomeRanking />} />
            </Route>
          </Routes>
        </div>
      </div>
    )
  }
}

```

Home.jsx

```jsx
export class Home extends PureComponent {
  render() {
    return (
      <div>
        <h1>Home</h1>
        <nav>
          <Link to="/home/reco">to recommand</Link>
          <Link to="/home/rank">to ranking</Link>
        </nav>
        {/* 占位组件符 表示子路由加载的组件放在这里 */}
        <Outlet />
      </div>
    )
  }
}
```

## 2.5.JS跳转

* **在Router6.x版本之后，代码类的API都迁移到了hooks的写法**
  * 如果想要使用js 跳转，需要`通过useNavigate的hook`获取到navigate对象进行操作
  * 但是这个只能在函数组件中调用，不能在类组件中调用
  * 如果想要`在类组件中`使用，需要使用`高阶组件`对原组件进行增强



**原理**

封装的高阶组件返回的是函数组件，这个函数组件中返回外界传入的类组件，然后给这个类组件添加上js API，所以就可以使用js跳转了



封装的高阶组件

```js
import { useNavigate, useParams } from 'react-router-dom'

function withRouter(WrapperComponent) {
  // naviga
  return function (props) {
    // 获取跳转对象
    const navigate = useNavigate()
    // 获取动态传参的参数
    const params = useParams()
    const router = { navigate, params }
    // 给类组件添加router对象
    return <WrapperComponent {...props} router={router} />
  }
}

export default withRouter

```

## 2.6.路由传参

### 2.6.1.动态参数



**传递**

```jsx
<Route path="/detail/:id" element={<Detail />} />
```



**获取**

* 使用`useParams`hook获取

* Router6.x使用的是hook，`类组件中不能直接使用`
* 所以需要使用`高阶组件`进行增强

```jsx
import {  useParams } from 'react-router-dom'

function withRouter(WrapperComponent) {
  
  return function (props) {
    // 获取动态传参的参数
    const params = useParams()
    const router = { params }
    // 给类组件添加router对象
    return <WrapperComponent {...props} router={router} />
  }
}

export default withRouter
```

### 2.6.2.查询字符串

* 使用`useLocation`和`useSearchParams`两个hook获取
* Router6.x使用的是hook，`类组件中不能直接使用`
* 所以需要使用`高阶组件`进行增强



```jsx
import {
  useLocation,
  useSearchParams,
} from 'react-router-dom'

function withRouter(WrapperComponent) {
    // 获取查询字符串  /detail?name=a&age=123
    // 有两种方式获取
    // 方式一 useLocation 这种获取的search是原始形式没有
    const location = useLocation()
    // 方式二 useSearchParams  返回的是URLSearchParams类 需要通过get方法才能获取
    // 这里将其转换为普通的对象 可以直接获取
    const [searchParams] = useSearchParams()
    const query = Object.fromEntries(searchParams)

    const router = { location, query }
    // 给类组件添加router对象
    return <WrapperComponent {...props} router={router} />
  }
}

export default withRouter
```

## 2.7.配置式路由

此方式是为了让react的路由配置和vue一样，配置在一个单独的文件中，不用全部写在组件里面

使用`useRoutes` hook进行设置





使用配置路由

```jsx
import React, { PureComponent, memo } from 'react'
import { Link, useRoutes } from 'react-router-dom'

import routes from './router'

const App = memo(function () {
  return (
    <div className="app">
     	{/* 配置路由 */}
      <div className="main">{useRoutes(routes)}</div>
      <div className="footer">footer</div>
    </div>
  )
})
export default App

```







配置路由文件

```jsx
import Home from '../pages/Home'
import NotFound from '../pages/NotFound'
import HomeRecommand from '../pages/HomeRecommand'
import { Navigate } from 'react-router-dom'

const routes = [
  {
    path: '/',
    element: <Navigate to="/home" />,
  },
  {
    path: '/home',
    element: <Home />,
    children: [
      {
        path: '/home',
        element: <Navigate to="/home/reco" />,
      },
      {
        path: '/home/reco',
        element: <HomeRecommand />,
      },
    ],
  },
 
  {
    path: '*',
    element: <NotFound />,
  },
]

export default routes

```

## 2.8.路由懒加载

使用`React.lazy` 改方法接受一个promise，使用该方法加载的组件，打包的时候会被单独打包



需要配合`<Suspense>`组件，因为页面初始化的时候懒加载的包还没有下载下来，所以会报错，这时候使用`<Suspense>`暂时代替



入口文件

```jsx
import ReactDOM from 'react-dom/client'
import App from './App'
import { HashRouter } from 'react-router-dom'
import { Suspense } from 'react'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <HashRouter>
    {/* 懒加载组件 */}
    <Suspense fallback={<h1>loding...</h1>}>
      <App />
    </Suspense>
  </HashRouter>
)
```



路由文件

```jsx
import Login from '../pages/Login'
import React from 'react'

// 懒加载
const About = React.lazy(() => import('../pages/About'))

const routes = [
  {
    path: '/login',
    element: <Login />,
  },
  {
    path: '/about',
    element: <About />,
  }
]

export default routes

```

