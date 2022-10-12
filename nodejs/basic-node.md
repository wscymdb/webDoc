# Node.js

# 1.定义

**官方定义**

* Node.js是一个基于``V8 JavaScript引擎``的``JavaScript``运行时环境

**自己理解**

* Node.js基于V8引擎来执行JavaScript的代码，但是不仅仅只有V8引擎
  * V8可以``嵌入任何C++应用程序中``，无论是``chrome``还是``node``，事实上都是嵌入了V8引擎来执行JavaScript代码
  * 但是只是嵌入，比如chrome中还有`别的程序来执行解析、渲染、浏览器的API`等
  * 那么node中也有别的程序来执行`别的操作，比如文件系统的读写、网络的IO`等

# 2.特殊的全局对象

* 这些全局对象实际上是模块中得变量，只是`每个模块都有`，`看起来像是全局变量`
* 在命令行交互中是不可以使用的
* 包括
  * `__dirname、__filename、eports、module、require()`

# 3.模块化开发

## 3.1.定义

* 

## 3.2.模块化历史

* ES6之前JavaScript是没有模块化得概念的
* `直到ES6(2015)`才推出了自己的`模块化方案(ESModule)`
* 在此之前，为了让JavaScript支持模块化，涌出很多不同的模块化规范:`AMD、CMD、ComminJS`

## 3.3.CommonJS

* `CommonJS是一个规范`，最初提出来是在浏览器以外的地方使用，并且当时名命为`ServerJS`，后来为了体现他的广泛性，修改为`CommonJS`，平时也会简称`CJS`
  * `Node是`CommonJS在`服务器端`一个具有代表性的实现
  * `webpack打包工具`具备对CommonJS的`支持和转换`
* Node中`对CommonJS进行了支持和实现`，让我们在开发node的过程中可以方便的进行模块化开发
  * 在Node中每一个js文件都是一个单独的模块
  * 这个模块中包含CommonJS规范的核心变量，`exports、module.exports、require`
* 模块化的核心是`导出和导入`，Node中对其进行了实现
  * `exports`和`module.exports`可以负责对模块中的内容进行`导出`
  * `require`函数可以`导入`其他模块中的内容

### 3.3.1.exports导出

* exports是一个对象，可以在对象中添加属性，添加的属性会被导出

* **导出导入的原理**

  * require(url)对象的时候，其实是通过url找到文件
  * 然后拿到文件的exports对象，将其赋值给接收的变量
  * 那么其实导出和导入的文件里面的exports``对象都是同一个``
  * 简单来说，导出和导入的对象其实就是``赋值引用``的关系

  ```javascript
  // a.js
  const a = {
    name: "jack",
  };
  setTimeout(() => {
    exports.a.name = "sdfsd";
  }, 2000);
  
  ```

  ```javascript
  //b.js
  const res = require("./a");
  console.log(res);
  setTimeout(() => {
    console.log(res);
  }, 3000);
  
  // 运行b.js文件
  // 可以看到在定时器结束后名字也变了
  // { name: 'jack' }
  // { name: 'sdfsd' }
  
  ```

  

### 3.3.2.module.exports导出

* CommonJS中是没有`module.exports`的概念的
  * 为了符合CommonJS的规范，所以设计了一个exports对象
  * 默认情况下module.exports指向exports对象
* 但是为了实现模块的导出，Node中`使用的是Module的类`，每一个模块都是Module的一个实例，也就是module
* 所以在Node中真正`用于导出`的其实根本不是exports，而`是module.exports`
  * 所以module.exports的`优先级高于`exports
* 因为module才是导出的正真实现者

### 3.3.3.require

* require是一个函数，可以引入一个文件(模块)中的导出对象

**require的查找规则**

导入格式如下：

​	**require(X)**

* **情况一**：X是一个Node的`核心模块`，如http、path

  * `直接返回`核心模块，并`停止查找`

* **情况二**：X是以`./`或`../`或`/`(根目录)开头的

  * `第一步`：将X当作一个文件在对应的目录下查找
    * 如果`有后缀名`，按照后缀名的格式查找文件
    * 如果`没有后缀名`，会按照如下顺序
      1. 直接查找文件X
      2. 查找X.js文件
      3. 查找X.json文件
      4. 查找X.node文件
  * `第二步`：灭有找到对应的文件，将X作为一个目录
    * 查找目录下的index文件，按照如下顺序
      1. 查找X/index.js文件
      2. 查找X/index.json文件
      3. 查找X/index.node文件
  * 如果`都没有找到，报错`

* **情况三**：直接是X，并且X不是核心模块

  * 会在当前文件夹的`node_modules文件夹`下查找X文件

    * 查找的顺序和上面一样

    如果当前文件夹没有node_modules，会`一级一级的往上级查找`这个node_modules文件夹，直到查找`到根目录`，没有就报错

### 3.3.4.Node模块加载解析过程

* **结论一：模块在被第一次引入时，模块中的js代码`会被运行一次`**
* **结论二：模块在被多次引入时，`会缓存`，最终`只加载(运行)一次`**
  * 因为每个模块对象module都有一个属性，`loaded`
  * 为false表示还没加载，为true表示已经加载
* **结论三：如果有循环引用，加载顺序时`按照深度优先算法`来执行的**
  * 有以下的文件引用关系
  * main.js -> a.js -> c.js -> d.js    main.js ->b.js -> e.js   b.js -> c.js
  * 这种引用关系是一种数据结构：`图结构`
    * 图结构在遍历过程中有`深度优先`搜索和`广度优先`搜索
  * 执行顺序是：main.js - a.js - c.js - d.js - e.js - b.js
    * 因为c.js已经被加载过一次，所以b引用的时候就不会被加载了

### 3.3.5.CommonJS的缺点

* CommonJS加载模块都`是同步的`
  * 同步意味着`只有等待对应的模块加载完毕，当前模块中的内容才能被运行`
  * 这个在服务器不会有什么问题，因为`服务器加载js文件都是本地文件`，加载`速度非常快`

* 浏览器中，通常是不使用CommonJS规范
  * 浏览器加载js文件需要先从服务器将文件下载下来，之后在加载运行
  * 那么采用同步的就意味着后续的js代码都无法正常运行，即使是一些简单的DOM操作

## 3.4.ES Module

* 是EcmaScript推出的模块化规范
* 使用`import`（导入）和`export`（导出）关键字
* 它采用编译期的静态分析，并且加入了动态引入的方式
* 采用ES Module将`自动采用严格模式`：use strict

### 3.4.1.导出导入的使用语法

* export {标识符1，标识符2，... }
  * {} `不是对象` 是export的特殊语法，所以里面的值也不是对象的增强写法
* import {标识符1，标识符2，... } from  xxx.xxx
  * {} `不是对象`，是import的特殊语法，用来设置接收的导出对象的
  * from后面的路径`必须加后缀名`

**注意**

* 当打开对应的html文件时，如果html中有使用模块化的代码，那么必须开启一个服务来打开

```javascript
// foo.js
const name = "zs";
const age = 18;

function sayHello() {
  console.log("hello");
}

export { name, age, sayHello };
```

```javascript
// main.js
import { name, age, sayHello } from "./foo.js";
console.log(name);
console.log(age);
sayHello();
```

### 3.4.2.export关键字

* 导出的三种方式（常见）

* ```javascript
  // 方式一
  const name = 'zs'
  export {name}
  
  // 方式二 导出时给标识符起一个别名
  const name = 'zs'
  export {name as fname}
  
  // 方式三 定义时直接导出，这中方式不可以起别名
  export const name = 'zs'
  export class Foo {}
  ```

### 3.4.3.import关键字

* 导入的三种方式（常见）

* ```javascript
  // 方式一：
  import {name} from './a.js'
  
  //方式二：起别名
  import {name as fname} from './a.js'
  
  // 方式三：导入时给整个模块起别名
  import * as fname from './a.js'
  console.log(fname.name)
  ```

### 3.4.4.import和export结合使用

有index.js文件,是其他文件的入口文件

* 默认情况

```javascript
import { formatCount } from "./formatCount.js";
import { formatDate } from "./formatDate.js";

export { formatCount, formatDate };
```

* 情况一

```javascript
export { formatCount } from "./formatCount.js";
export { formatDate } from "./formatDate.js";

//等价于
import { formatCount } from "./formatCount.js";
import { formatDate } from "./formatDate.js";

export { formatCount, formatDate };
```

* 情况二

```javascript
// 这种方式时将文件内的内容全部引入，然后再导出
export * from "./formatCount.js";
export * from "./formatDate.js";
```

### 3.4.5.default(默认导出)

* 默认`导出`可以`不需要指定名字`
* 在导入时`不需要使用{}`,并且可以`自己指定名字`
* **注意**：在一个模块中，`只能有一个`默认导出

```javascript
// a.js(导出文件)
const name = 'zs'
export default name

// b.js(导入文件)
import hh from './a.js'
```

### 3.4.6.import函数

* 通过import加载一个模块，是不可以将其放到逻辑代码中的

  ```javascript
  if(true) {
      import a from './a.js'
  }
  // 代码报错
  ```

  * 因为`ES Module再被JS引擎解析`时，就`必须知道他的依赖关系`
  * 由于`这时候js代码没有任何的运行`
  * 甚至`拼接路径的写法也是错误`的，因为必须到运行时才能确定path的值

* 但是再某种情况下，`希望动态的加载某一个模块`

* 这时候可以使用`import函数`来动态加载

  * import函数`返回一个promise`，可以通过then方法获取结果

* ```javascript
  import('./a.js').then(res => {
      console.log(res)
  })
  ```

**import.meta**

* import.meta是一个给JavaScript模块暴露特定上下文的元数据属性的对象
  * 它包含了这个模块的信息，比如模块的URL
  * ES11中新增的特性

### 3.4.7.ES Module的解析流程

* 分为三个阶段
* 阶段一：构建(Construction)，根据地址查找js文件，并且下载，将其解析成模块记录(Module Record)
* 阶段二：实例化(Instantiation)，对模块记录进行实例化，并且分配内存空间，解析模块的导入和导出语句，把模块指向对应的内存地址
* 阶段三：运行(Evaluation)，运行代码，计算值，并且将值填充到内存地址中

# 4.包管理工具

## 4.1.npm的配置文件

* **package.json文件**

**常见的创建方式**

* **方式一**：`手动添加`
  * npm init 
    * 需要手动填写信息
  * nmp init -y 
    * 使用默认的信息
* **方式二**：`通过脚手架创建项目`
  * 脚手架会自动生成package.json，并且配置相关的信息

### 4.1.1package.json常见的属性

**基本属性**

* `name`：项目的名称，（必填）
* `version`：当前项目的版本号，（必填）
* `description`：描述信息，大多时候作为项目的基本描述
* `author`：作者相关信息(发布时用到)
* `license`：开源协议(发布时用到)

**private属性**

* `private`属性记录当前的项目`是否是私有`的
  * 当值为`true`，npm是`不能发布他的`，为了防止私有项目或模块发布出去的方式

**main属性**

* `main`：设置`程序的入口`
  * 当引入模块的时候，只写文件夹，默认情况下会去这个文件夹下找index相关的文件
  * 使用`main属性来配置入口`，那么就会找 main配置的相关文件

**scripts属性**

* `scripts`：用于配置一些脚本命令，以键值对的形式存在

* 配置后可以通过`npm run 命令的key`来执行这个命令

* `npm start` 和 `npm run start`的区别

  * 他们是等价的
  * 对于常用的start、test、stop、restart可以`省略掉run`，直接通过npm start等方式运行

* ```json
  {
      scripts: {
          "start": 'node ./main.js',
          'build': 'webpack ...'
      }
  }
  ```

**dependencies属性**

* `dependencies`：指定无论开发环境还是生产环境都需要依赖的包

**devDenpendencies属性**

* `devDenpendencies`: 开发环境需要的依赖包
* npm install --save-dev

**peerDependencies**

* `peerDependencies`：表示`对等依赖`，也就是`一个依赖包，它必须是以另一个宿主包为前提的`
  * 比如element-plus依赖vue3的，那么element-plus的packages.json中就有`peerDependencies：vue3`,这样的字段，当使用element-plus的时候，如果当前项目没有vue就会提示用户这个包是依赖vue的

**engines属性**

* `engines`：用于`指定Node和NPM的版本号`
* 在安装的过程中，会先检查对应的引擎版本，如果`不符合就会报错`
* 事实上也可以指定所在的操作系统“os”:['linux'],只是很少用到

**browserslist属性**

* 用于配置打包后的JavaScript浏览器的兼容情况
* 也可以单独在一个.browserslistrc文件中进行配置

### 4.1.2.依赖的版本管理

* 安装依赖的时候会出现:`^2.0.3`或`~2.0.3`
* npm的包通常会遵从`semver`版本规范
* semver版本规范是X.Y.Z
  * **X：主版本号(major)**
    * 当你做了不兼容的API修改(可能不兼容之前的版本，比如vue3不兼容vue2的eventbus)
  * **Y：次版本号(minor)**
    * 当你做了向下兼容的功能性新增(新功能增加，但是兼容之前的版本)
  * **Z：修订号(patch)**
    * 当你做了向下兼容的问题修正(没有新功能，修复了之前版本的bug)
* `^`和`~`的区别
  * `x.y.z`：表示一个明确的版本号
  * `^x.y.z`：表示x保持不变，y和z永远安装最新的版本
    * 下次`nmp install`的时候如果有更新，就会使用新的版本
  * `~x.y.z`：表示x和y保持不变的，z永远安装最新的版本
    * 下次`nmp install`的时候如果有更新，就会使用新的版本

### 4.1.3.npm install原理

![](https://img-blog.csdnimg.cn/4a2d459ea4f340578d84b6d86bb75adc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Y2W6I-c55qE5bCP55m9,size_20,color_FFFFFF,t_70,g_se,x_16)

### 4.1.4.npm其他命令

* 卸载某个依赖包
  * npm uninstall package
  * npm uninstall package --save-dev
  * npm uninstall package -D
* 强制重新build
  * npm rebuild
* 清除缓存
  * npm cache clean
* 查看缓存的 位置
  * npm config get cache

## 4.2.yarn