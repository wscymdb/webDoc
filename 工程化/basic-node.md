# node

主要的内容是和前端工程化相关的

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
* 在此之前，为了让JavaScript支持模块化，涌出很多不同的模块化规范:`AMD、CMD、CommonJS`

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
  * 默认情况下module.exports指向exports对象(exports = module.exports)
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
  * `第二步`：没有找到对应的文件，将X作为一个目录
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
  * main引入a.js和b.js
  * a.js引入c.js  c.js引入d.js.  d.js引入e.js
  * b.js引入c.js和e.js
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

## 4.2.npx工具

* npx是npm5.2之后自带的一个命令

  * npx的作用非常多，但是比较常见的是使`用它来调用某个模块的指令`

* 举例（以webpack为例）

  * 在终端输入命令，默认会在当前的目录下查找可执行文件，但是不会去当前目录的子目录中查找
  * 当我们在当前文件夹下输入 webpack 它会先去当前目录下查找，如果没有，再去全局查找，如果没有就会报错
  * 怎么解决呢(`局部查找方式一`)？node_modules文件夹下有一个.bin文件夹里面存放的有所有包的可执行文件，我们进入到这个.bin文件夹下打开终端然后输入webpack即可执行局部的webpack
  * `npx的作用就是优先去当前文件夹的node_modules的.bin文件下查找`
    * 当 输入npx webpack，会先去当前目录的node_modules的.bin文件夹下查找webpack可执行文件

* 如果在package.json中的scripts中是不用写npx的，因为这个里面默认就会去.bin文件夹下查找（`局部查找方式二`）

  ```json
  {
      "scripts": {
          "build": "webpack"
      }   
  }
  ```

  

## 4.3.pnpm

### 4.3.1.原来包管理工具的痛点

* 像npm、yarn、cnpm这些包管理工具都一个最大的缺点
* 如果你电脑有3个项目，每个项目都要axios、vue、webpack等等这些三方包
* 那么每一个项目都会下载一份的，如果你有10个100个项目呢，
* 那么`电脑的内存会随着项目的增加而减少`
* `pnpm解决了这一痛点`

### 4.3.2.定义

* pnpm：`performant npm`的缩写
* 速度快、节省磁盘空间的软件包管理器

### 4.3.3.硬链接和软链接

**知识储备**

* 存放在磁盘的数据是`物理数据我们无法去操作`
* 这时候要用到操作系统去操作，操作系统会将磁盘的数据抽象成一个文件的形式让我们去访问操作

**维基百科定义**

* **硬链接(hard link)**
  * 是`电脑文件系统中多个文件平等的共享同一个存储单元`
  * 删除一个文件名字后，还可以用其他名字继续访问该文件
* **符号链接(软链接soft link)：**
  * 是`一类特殊的文件`
  * 包`含一条以绝对路径或相对路径的形式指向其他文件或者目录的引用`

**举例**

* **硬链接**
  * 有一个mp4文件存放在物理磁盘中，操作系统在F盘有个a.mp4,这个a.mp4指向物理磁盘的那个MP4文件，那么这中指向关系就是`硬链接`
  * 在C盘中有一个b.mp4也指向物理磁盘的MP4文件，这种指向也是`硬链接`
  * 所以即使删除b.mp4文件，a.mp4也依然能够访问磁盘的MP4文件
* **软链接**
  * 在桌面创建了a.mp4的快捷方式，那么这个快捷方式存放的是a.mp4的相对绝对路径
  * 这个快捷方式和a.mp4这种指向就是`软链接`
  * `软链接的文件是不可编辑的`
* **演示**(`widnow 的cmd进行操作`)
  * **文件拷贝**：会在硬盘中复制一根新的文件数据
    * copy  原文件  新文件
    * copy a.js  a_copy.js
  * **文件的硬链接**
    * `mklink /H  新文件  原文件`
    * mklink /H  a_hardlink.js  a.js
  * **文件的软链接**
    * `mklink  新文件  原文件`
    * mklink  a_soft.js  a.js

### 4.3.4.pnpm原理

* 使用`pnpm安装依赖`，依赖包将被`存放在一个统一的位置`，因此
  * 如果多个项目对`同一个依赖的版本相同`，那么磁盘上`只有一份这个依赖包`
  * 如果多个项目对`同一个依赖是不同版本`，那么版本之间`不同的文件就会被存储起来`
  * `使用pnpm安装的依赖，所有文件都保存在硬盘上的统一位置`
    * 当安装软件包时，其包含的所有文件`都会硬链接到此位置`，为不会占用额外的磁盘空间
    * 这可以让项目之间方便的共享相同版本的依赖包

* **举了栗子**
  * pnpm add axios
  * 在当前文件的node_modules可以看到axios文件夹和.pnpm文件夹
  * axios文件夹是一个软链接，他的地址就是.pnpm里面的axios文件夹(这个文件夹是硬链接，指向磁盘上的数据)

### 4.3.5.pnpm常见命令

* pnpm install
* pnpm add \<pkg>
* pnpm remove \<pkg>
* pnpm \<cmd>
* **pnpm store path**
  * 查找当前电脑pnpm的仓库在哪里
* **pnpm store prune**
  * 从store中`删除当前未被引入的包`来释放store的空间

# 5.Node内置模块

## 5.1path模块

* path模块用于`对路径和文件进行处理`
* 在Mac OS、Linux和window上的路径分隔符是不一样的
* path会根据不同操作系统返回分隔符

### 5.1.1.常见的PAI

**从路径中获取信息**

* dirname：获取文件夹的父文件夹
* basename：获取文件名
* extname：获取文件拓展名

**路径拼接**

* path.join
* 如果希望将多个路径进行拼接，但是不同 的操作系统可能使用的是不同的分隔符，可以使用path.join

**拼接绝对路径**

* path.resolve
* 该方法会把一个路径或路径片段的序列(传入的路径参数)解析成一个绝对路径
* 给定的路径序列是从右往左被处理的，后面每个path被依次解析，`直到构造完成一个绝对路径`，就停止解析
* 如果在处理完所有给定path之后，还没有生产绝对路径，则使用当前工作目录
* 生成的路径被规范化并删除尾部斜杠，零长度的path将被忽略
* 如果没有path传入(无参数),path.resolve()将返回当前工作目录的绝对路径

```javascript
const path1 = './abc'
const path2 = './cba/nbc'

const res1 = path.join(path2,'../text')
// cba\text

// / 表示根目录
const res2 = path.resolve(path1,'/aaa',path2)
// C:\aaa\cba\nbc


```



# 6.webpack_basic

## 6.1.定义

* webpack是一个静态的模块化打包工具，为现代的JavaScript应用程序
* 官方解释
  * webpack is a `static module bundler` for `modern` JavaScript applications
* `打包bundler`：webpack可以进行打包，所以是一个打包工具
* `静态的static`：webpack最终可以将代码打包成静态资源
* `模块化module`：webpack默认支持各种模块化开发
* `现代的modern`：现在的开发模式才需要打包

## 6.2.安装

* webpack的安装目前分为：`webpack、webpack-cli`
* **两者的关系**
  * `执行webpack命令`，会执行node_modules下的.bin`目录下的webpack`
  * webpack在执行时是依赖webpack-cli的，如果没有安装就会报错
  * 而`webpack-cli中代码执行时`，才是`真正利用webpack`进行编译和打包的过程
  * 所以在安装webpack时，需要同时安装webpack-cli（第三方框架事实上没有用webpack-cli的，用的类似于自己的vue-cervice-cli的东西）

## 6.3.常见指令

**指定配置文件**

* 默认情况下webpack的配置文件是`webpack.config.js`
* 如果想`换成其他的`，就这终端输入`webpack --config 自定义名称`
  * webpack --config a.js

## 6.4.loader

### 6.4.1.loader配置方式

* `module.rules`中允许我们配置多个loader

**module.rules配置**

* rules属性对应的值是一个`数组`: **[Rule]**
* 数组中存放的是一个个Rule，`Rule是一个对象`，对象中可以设置多个属性
  * `test属性`：用于对resource(资源)进行匹配，通常会设置成`正则表达式`
  * `use属性`：对应的值是一个数组[UseEntry]
    * UseEntry是一个对象，可以通过对对象的属性来设置一些其他属性
      * loader:必须有一个loader属性，对应的是一个字符串
      * options：可先的属性，值是一个字符串或对象，会被传入到loader中
      * query:目前已经使用options来代替
    * userEntry是一个字符串：是loader的一种简写
  * `loader属性`：如果只有一个loader时候，可以使用这个属性简写

```javascript
const path = require("path");
module.exports = {
    entry: "./src/main.js",
    output: {
        filename: "bundle.js",
        path: path.resolve(__dirname, "./dist"),
    },
    module: {
        rules: [
            {
                // 告诉webpack匹配什么样的文件
                test: /\.css$/,
                // 详细写法
                // use: [{ loader: "style-loader" }, { loader: "css-loader" }],
                // 简写一：只有一个loader的情况
                // loader: "css-loader",
                // 简写二:多个loader不需要其他属性
                use: ["style-loader", "css-loader"],
            },
        ],
    },
};
```



### 6.4.2.css-loader 

### 6.4.3.style-loader 

### 6.4.4.less-loader

## 6.5.postcss工具

**作用**

* PostCSS是一个通过Javascript来转换样式的工具
* 这个工具可以进行一些`CSS的转换和适配`，比如自动添加浏览器前缀、css样式的重置
* 实现这些功能需要`借助PostCSS对应的插件` 

**使用**

* 第一步：查找PostCSS在构建工具中的扩展，比如webpack中的postcss-loader；
* 第二步：选择可以添加你需要的PostCSS相关的插件；

**安装**

`npm install postcss-loader -D`

### 6.5.1.使用postcss里面的插件

#### **autoprefixer**

* autoprefixer是给需要添加css前缀的属性添加前缀
* 需要下载npm install autoprefixer -D

```javascript
module: {
    rules: [
        {
            test: /\.less$/,
            use: [
                "style-loader",
                "css-loader",
                "less-loader",
                {
                    loader: "postcss-loader",
                    options: {
                        postcssOptions: {
                            plugins: ["autoprefixer"],
                        },
                    },
                },
            ],
        },
    ],
},
```

* 上面的配置让页面显得太复杂了
* 可以使用一个`postcss.config.js`文件

**postcss.config.js**

```javascript
module.exports = {
  plugins: ["autoprefixer"],
};

```

**webpack.config.js**

```javascript
{
    rules: [

        {
            test: /\.less$/,
            use: [
                "style-loader",
                "css-loader",
                "less-loader",
                "postcss-loader",
            ],
        },
    ],
},
```

#### postcss-preset-env

* 事实上，在配置postcss-loader时，并不需要autoprefixer插件
* 可以使用`postcss-preset-env`插件
* 这个插件可以自动将一些`现代的CSS特性`，`转换成大多数浏览器认识的CSS`，并且会根据目标浏览器或者运行时环境添加所需的polyfill
* 也包括自动帮助我们添加autoprefixer(相当于内置了autoprefixer插件)

postcss.config.js：

```javascript
module.exports = {
    plugins: ["postcss-preset-env"],
};

```



## 6.6.asset module type

* webpack5之前加载类似图片资源、文字资源需要用对应的loader进行处理，webpack5内置了这些loader，只需要设置对应的type即可
* 资源模块类型
  * `asset/resource`：生成一个单独的文件并导出URL
    * 之前通过使用file-loader使用
  * `asset/inline`：导出一个资源的data URI
    * base64格式
    * 之前通过使用url-loader实现
  * `asset/source`：导出资源的源代码
    * 拿到文件的二进制代码
    * 之前通过raw-loader实现
  * `asset`：在asset/inline和asset/resource之间自动选择
    * 之前通过url-loader，并配置资源体积实现

### 6.6.1.**根据图片大小设置不同资源类型**

* 开发中我们往往希望小点的照片使用base64格式，大点的照片生成一个单独的文件
* 只需要两步
  1. 将type设置为asset
  2. 添加一个parser属性，并且制定dataUrl的条件，添加maxSize属性
* 如果`大于maxSize`的文件将会使用`asset/resource`类型，反之使用`asset/inline`类型

```javascript
module: {
    rules: [
        {
            test: /\.(jpe?g|png|svg|gif)$/,
            // type: "asset/resource",
            // type: "asset/inline",
            // type: "asset/source",
            type: "asset",
            parser: {
                dataUrlCondition: {
                    maxSize: 60 * 1024,
                },
            },
        },
    ],
}
```

### 6.6.2.生成自定义的文件名

* 可以在output的assetFilename中设置
* 也可以在`匹配对应规则之后设置(常用)`
  * 在`generator`属性中指定

```javascript
module: {
    rules: [
        {
            test: /\.(jpe?g|png|svg|gif)$/,
            // type: "asset/resource",
            // type: "asset/inline",
            // type: "asset/source",
            type: "asset",
            parser: {
                dataUrlCondition: {
                    maxSize: 60 * 1024,
                },
            },
            generator: {
                // 占位符
                // name:指向原文件的名称
                // ext：拓展名
                // hash：webpack生成的hash值
                // [hash:8]:指定截取hash的数量
                // 还可以指定一个文件夹 在文件前面加文件夹名称即可
                filename: "images/[name]_[hash:8][ext]",
            },
        },
    ],
}
```

## 6.7.babel工具

* Babel是一个工具链，主要用于旧浏览器或者环境中将ECMAScript 2015+代码转换为向后兼容版本的JavaScript；
* 包括：语法转换、源代码转换等；

### 使用babel中的插件

* 单独设置babel-loader是不会将箭头函数、const这些转成es5的代码的，需要使用插件进行配置
* 这里使用Babel的预设插件
  * 这个插件内置了很多常用的插件

**安装**

`npm install @babel/preset-env -D`

```javascript
module: {
    rules: [

        {
            test: /\.js$/,
            use: [
                {
                    loader: "babel-loader",
                    options: {
                        plugins: ["@babel/plugin-transform-arrow-functions"],
                    },
                },
            ],
        },
    ],
}
```

* 上面的配置还可以抽成一个单独的配置文件`babel.config.js`

* 如果是预设，那么要添加在`presets`属性中

* ```javascript
  module.exports = {
      // plugins: ["@babel/plugin-transform-arrow-functions"],
      presets: ["@babel/preset-env"],
  };
  ```

**常见的预设**

* env：将es6转为es5
* react
* TypeScript

## 6.8.resolve模块

* resolve用于设置模块如何解析
  * 在开发中会有各种各样的模块依赖
  * resolve可以帮助webpack从每个require/import语句中，找到需要引入到合适的模块代码

**webpack能解析三种文件路径**

* 绝对路径
  * 给出了绝对路径，因此不需要在做进一步的解析
* 相对路径
  * 使用import或require的资源文件所处的目录，被认为是上下文目录
  * 在import/require中给定的相对路径，会拼接此上下文路径，来生成模块的绝对路径
* 模块路径
  * 在resolve.modules中指定的所有目录检索
    * 默认值是`['node_modules']`，所以默认会从node_modules中查找文件

**确定文件还是文件夹**

*  如果是一个文件

  * 如果文件具有拓展名，则直接打包
  * 否则，将使用resolve.extensions选项作为文件拓展名解析

  ```javascript
  module.exports = {
      resolve: {
          extensions: ['js','json','vue'] 
      }
  }
  ```

  

* 如果是一个文件夹

  * 会在文件夹中根据resolve.mainFiles`配置选项中指定的文件顺序查找`
    * resolve.mainFiles的默认值是['index']
    * 再根据resolve.extensions来解析拓展名

```javascript
module.exports = {
    resolve: {
        mainFiles: ['index']   
    }
}
```

### **alias**

* 起别名

```javascript
module.exports = {
    resolve: {
        mainFiles: ['index'],
        alias: {
            '@': path.resolve(__dirname, './src')
        }
    }
}
```

## 6.9.webpack常见的插件(plugin)

### 6.9.1.认识plugin

* Loader是`用于特定的模块类型`进行转换
* Plugin可以用于`执行更加广泛的任务`，比如打包优化、资源管理、环境注入等

### 6.9.2.CleanWebpackPlugin

* 每次打包之后将之前的打包文件夹删除

**安装**

npm install clean-webpack-plugin -D

**配置文件**

```javascript
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
module.exports = {
    entry: "./src/main.js",
    output: {
        filename: "bundle.js",
    },
    module: {}

    plugins: [new CleanWebpackPlugin()],
};
```

### 6.9.3.HtmlWebpackPlugin

* 生成一个html文件

**安装**

npm install html-webpack-plugin -D

**配置**

* 自定义模板数据填充
  * title：生成html的title属性
  * template：按照模板生成html文件

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
    entry: "./src/main.js",
    output: {
        filename: "bundle.js",
    },
    module: {}

    plugins: [  new HtmlWebpackPlugin({
    title: "我是大帅逼",
    template: "./index.html",
})],
};
```

### 6.9.4.DefinedPlugin

* DefinedPlugin`允许在编译时创建配置的全局常量`
* 是一个webpack的`内置插件`，无需安装

**配置**

* 注意：配置的值在解析的时候`会当成代码解析`，所以要`加引号`让他变成字符串

* 默认注入的全局变量

  * `process.env.NODE_ENV`：判断当前的开发环境

* ```javascript
  const { DefinePlugin } = require("webpack");
  module.exports = {
      entry: "./src/main.js",
      output: {
          filename: "bundle.js",
      },
      module: {}
  
      plugins: [  new DefinePlugin({
      named: "'Jack'",
  })],
  };
  ```

## 6.10.Mode配置

* 可以告知webpack`使用相应的内置优化`
  * 默认值是`production`
  * 可选值有：'none' | 'development' | 'production'

## 6.11.devServer

* 监听代码变化，自动编译并且刷新浏览器
* webpack-dev-server在编译之后`不会生成文件到文件夹中`，而是启动一个本地服务，将编译好的代码放入到本地服务中，浏览器在过来请求拿到文件

**安装**

`npm install webpack-dev-server -D`

**命令**

`webpack serve`

### 6.11.1.热模块替换(HMR)

#### 定义

* HMR的全称是`Hot Module Replacement`
* 模块热替换是指在 `应用程序运行过程中`，`替换、添加、删除模块`，而`无需刷新整个页面`

**优点**

* 不重新加载整个页面，这样可以保留某些应用程序的状态不丢失
* 只更新需要变化的内容，节省开发时间
* 修改了css、js源码，会立即在浏览器更新

**使用**

* 默认情况下，webpack-dev-server已经支持HMR，只需要开启即可(`默认开启`)
* 在不启用HMR的情况下，当修改了源代码之后，整个页面会自动刷新，使用的是live reloading

修改webpack配置

```javascript
module.exports = {
    devServer: {
        hot: true
    }
}
```

**指定哪些模块需要热更新**

* 注意：如果入口文件发生了变化，还是会刷新整个网页的

```javascript
if (module.hot) {
    console.log("hot");
    module.hot.accept("./utils/demo.js", () => {
        console.log("mian.js is changed");
    });
```

* 在`框架中`，框架已经对每个组件开启了HMR，所以`不用手动设置`

### 6.11.2.devServer配置

**port**

* 设置端口号

**host**

* 配置主机
* 设置成：`local-ipv4`
  * 监听IPV4上所有的地址，在根据端口找到不同的应用程序

**open**

* 是否打开浏览器

**compress**

* 是否为打包后的代码压缩 gzip compression
* 默认值是`true`

#  7.git

## 7.1.用户名和邮箱配置

**命令**

```
git config --global user.name  "名称"
git config --global user.email  "邮箱"
```

## 7.2.文件的状态

* `未跟踪(Untracked)`：默认情况下，git仓库下的文件没有添加到git仓库中，需要通过add命令来操作
* `已跟踪(tracked)`：添加到Git仓库的文件处于已跟踪状态，git可以对其进行各种跟踪管理

**已跟踪文件状态**

* `staged（暂缓）`：暂缓区的文件状态
* `Unmodified（未修改）`：commit命令，可以将staged中的文件提交到Git仓库
* `Modified（修改）`：修改某个文件后，会处于Modified状态

## 7.3.git 相关命令

**查看状态**

```
git status  // 查看文件的状态
git status -s // 查看文件的状态(简洁显示)
git status --short // 查看文件的状态(简洁显示)
```

暂存

```
git add .  // 暂存全部文件
git add 文件名  // 暂存某个文件
```

**提交**

```
git commit -m 'message'  // 添加到本地仓库
git commit -a -m '自定义内容'   // 添加到暂缓去并且提交的本地仓库
```

 **log**

```
git log  // 查看提交历史 最近的更新排在最上面

git log --pretty=oneline  // 每行显示一条信息

git log --pretty=oneline --graph  // 如果有多个分支合并到主分支，此操作可以查看每次提交来自于哪个分支
```



## 7.4.忽略文件

* 创建一个`.gitignore`文件用于忽略某些不需要提交的文件
* 一般不需要自己手动编写，脚手架会自动生成



## 7.5.git的校验和

* git中所有的数据在存储前都计算校验和，然后以`校验和`来引用
  * git用以计算校验和的机制叫做SHA-1散列(hash)
  * 这是一个由40个十六进制字符(0-9和a-f)组成的字符串，基于git中文件的内容或目录结构计算出来
* 每次commit的时候都有一个id 这个id就是校验和



## 7.6.版本回退

* 如果想要进行版本回退，需要直到目前处于哪个版本：git通过HEAD指针记录当前版本
  * HEAD是当前分支引用的指针，他总是指向该分支上的最后一次提交
  * 可以将`HEAD`看作是`该分支的最后一次提交的快照`

**命令**

* 上一个版本就是HEAD^，上上一个版本就是HEAD^^
* 如果上10个版本，可以使用HEAD~10
* 也可以指定某一个版本的commit id

```
git reset --hard HEAD^
git reset --hard HEAD~10
git reset --hard commitID  // commitID 只需要取不同的即可(比如取前8位)
```

**reflog**

* 记录了所有的操作
* 可以用来撤销回退的版本
  * 拿到回退之前的版本，然后回退过去

```
git reflog
```

## 7.7.验证方式-SSH密钥

* Secure Shell(安全外壳协议，简称SSH)是一种加密的网络传输协议，可在不安全的网络中为网络服务提供安全的传输环境
* SSH以非对称加密实现身份验证

**生成密钥的命令**

```
ssh-keygen -t ed25519 -C '邮箱'
```

* 将生成的公钥放到服务器即可

## 7.8.管理远程仓库

**查看远程仓库地址**

```
git remote   
git remote -v  // 完整版
```



**添加远程地址**

```
git remote add 仓库名 仓库地址
eg: git remote add origin http://www.xxx.com
```



**操作远程仓库**

```
// 重命名远程地址  
git remote rename  旧名称  新名称

// 移除远程地址
git remote remove 仓库名
```

**操作远程仓库的完整流程(非clone的仓库)**

```
1. 创建本地仓库
git init
2. 提交本地代码到本地仓库
git add .
git  commit -m 'xxx'
3.远程连接
git romote add origin  git@gitee.com:wscymdb/main-process-of-scaffold.git
4. 拉取并且合并远程仓库到本地
git pull
```

```
第四步会遇到两个问题：
问题一：
	原因是git本地仓库要和远程仓库的哪个分支相互链接
解决：
	git pull <remote> <branch>
	git pull origin master
但是每次pull的时候都要写后面的太麻烦,所以可以设置上游分支，这样以后就可以直接pull了

设置上游分支
	// git branch --set--upstream-to=origin/<branch> master
	
	git branch --set-upstream-to=origin/master master
	
问题二：

报错：fatal: refusing to merge unrelated histories

原因：
	Git2.9版本之后，不允许两个没有共同基础的两个分支相互合并，因为如果本地仓库是一个没有经验或没有规范的提交，那么本地就会有很多提交历史，如果合并到远程仓库，会导致远程仓库也有很多历史提交，显然这些是无用的，所以Git2.9之后就不被允许了
	
解决：
	添加：–-allow-unrelated-histories
	git pull –-allow-unrelated-histories
```

### 7.8.1.远程仓库的交互

**获取代码**

```
git clone <远程仓地址>
```

**推送**

* `默认情况下是将当前分支(比如master) push到 origin远程仓库的`

```
git push
git push origin master
```

* `注意`：

  * 当使用本地仓库链接远程仓库的时候，如果当前push所在的本地仓库和云仓的分支不一样的时候，使用git push默认情况下是使用的当simple，如果云端没有当前本地的分支那么就会报错
  * Git2.0版本之后push.default的值是`simple`
  * 所以设置push的默认值为`upstream`即可

  ```
  git config push.default upstream
  
  push.default的值
  1.simple(默认)：推送当前分支到远程仓库同名分支，没有同名分支会报错
  2.current：推送当前分支到远程仓库同名分支，没有同名分支会在远程仓库创建一个同名的分支
  2.upstream:推送当前分支的上游分支到远程仓库同名的分支
  ```

  

**获取最新代码**

* `默认情况下是从origin中获取代码`

```
git fetch
git fetch origin master
```

* `获取代码后默认并没有合并到本地仓库，许哟啊通过merge来合并`

```
git merge 
git merge origin/master  // 表示将origin仓库的master分支合并到当前分支
```

**获取并合并**

* `是git fetch和gitmerge的合并写法`

```
git pull
```

## 7.9.Git标签

* 对于重大的版本常常会打上一个标签，以表示他的重要性
* 比较由代表性的是人们会使用这个功能来标记发布节点

**创建标签**

* Git支持两种标签：轻量标签(lightweight)和附注标签(annotated)

```
轻量标签
git tag v1.0.0

附注标签
git tag -a v1.1.1 -m"附注标签描述"
```

**查看**

```
git tag
```



**推送标签到远程仓库**

* 默认情况下，git push命令并不会传送标签到远程仓库服务器上

```
git push origin  v1.0.0 // 推送指定tag
git push origin --tags  // 推送全部tag
```



**删除tag**

```
// 删除本地tag
git tag -d <tagname>
// 删除远程tag
git push <remote> -delete <tagname>
```

**检出tag**

* 回到某个tag的版本
* 正常情况下，要是想修改某个tag的内容，应该创建当前tag的分支，在该分支上进行修改

```
git checkout <tagname>
```

## 7.10.分支

**创建**

```
创建分支
git branch <branchname>

创建并切换到该分支
git checkout -b <branchname>
```

**查看**

```
查看所有分支
git branch

查看所有分支，并查看最后一次提交
git branch -v

查看所有合并到当前分支的分支
git branch --merged

查看所有没有合并到当前分支的分支
git branch --no-merged
```

**删除**

```
删除某个分支
git branch -d <branchname>

强行删除某个分支
git branch -D <branchname>
```

### 7.10.1.Git远程分支

* 远程分支也是一种分支结构
  * 以\<remote>/\<branch>的形式命名
    * origin/master

**远程分支的管理**

* **`推送分支到远程`**

  ```
  git push <remote> <branch>
  
  eg:
  git push origin develop
  ```

* **`跟踪远程分支`**

  * 当克隆一个仓库时，通常会自动地创建一个跟踪origin/master的master分支

  * 也可以手动的设置其他的跟踪分支

    ```
    git checkout --track <remote>/<branch>
    
    简写
    git checkout <branch>
    
    // 1. 查看本地是否有该分支
    // 2. 查看云仓是否有，有的话就在本地创建一个分支并且跟踪远程的这个分支
    ```

* **``删除远程的分支``**

  ```
  git push origin -d <remotebranchname>
  ```

## 7.11. commit操作相关

**修改你最近一次的 Git commit 信息**
  你可以使用以下命令来修改你最近一次的 Git commit 信息：

```bash
git commit --amend
```

这将打开你的默认文本编辑器，你可以在其中编辑提交消息。保存并关闭编辑器后，新的提交消息将替换旧的。

如果你已经推送了这个提交到远程仓库，你需要强制推送更新：

```bash
git push --force
```

请注意，强制推送可能会覆盖其他人的工作，所以在团队协作时要谨慎使用。

如果你需要修改更早的提交，可以使用交互式 rebase：

1. 首先，启动交互式 rebase，指定你想要修改的提交之前的一个提交：

    ```bash
    git rebase -i HEAD~n
    ```

    其中 `n` 是你想要回溯的提交数量。

2. 在打开的编辑器中，将你想要修改的提交前的 `pick` 改为 `edit`。

3. 保存并关闭编辑器。Git 会暂停在你指定的提交上。

4. 使用 `git commit --amend` 修改提交信息。

5. 使用 `git rebase --continue` 继续 rebase 过程。

6. 最后，强制推送更新：

    ```bash
    git push --force
    ```

希望这些步骤能帮到你！如果有其他问题，请随时告诉我。
