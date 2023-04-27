#  1.source-map

## 1.1.定义

* 使用打包工具会将代码进行编译，然后跑在浏览器上
  * 比如`ES6的代码`转成`ES5`的代码
  * 比如对应的`代码行号、列号`在经过编译后肯定与之前不一样
  * 比如代码`丑化压缩`时，会将`编码名称`等修改
* **但是，当代码报错，需要调试（debug），那么控制台中报错的文件名、行数是不对的**
* **所以可以使用source-map**
  * source-map会`将已经编译的文件中的代码`映射成`未编译前的代码`
  * 这样`可以使的浏览器重构原始代码`，所以即使在调试的时候也可以知道哪个文件的哪一行等信息报错

## 1.2.使用

* **第一步**：根据源文件，生成source-map文件`(webpack在打包时，可以通过配置文件生成source-map)`
* **第二步**：在转换后的代码，最后添加一个注释，它指向对于当前代码的source-map文件
  * eg: ` //# sourceMappingURL=boundle.js.map`
* **浏览器会根据我们的注释，查找相应的source-map，并且根据source-map还原打包前的代码，方便调试**



**上面两步 只需要配置devtool就行**

```javascript
// 这个注释必须紧挨着配置对象 这些才有代码提示
/** @type {import('webpack').Configuration} */
module.exports = {
  mode: 'production',
  devtool: 'source-map', // 生成source-map文件
  },
}
```

## 1.3.devtool的值

* webpack为我们提供了非常多的选项(目前有26个)，来处理source-map
* `值不同`，生成的source-map会`稍微有差异`，打包的过程也会有`性能的差异`，可以根据不同情况进行选择
* 有些值可以用在production，有些则不能，看一下链接文档的说明
* https://www.webpackjs.com/configuration/devtool/#root



**下面几个值是不会生成source-map文件的**

* **false**：不使用source-map，也就是没有任何source-map相关的内容
* **none**：`mode：production`时候的默认值，不生成source-map （`不能主动设置否则会报错`，真当production是自动设置）
* **eval**：`mode:development`时的默认值，不生成source-map
  * 但是它会在eval执行的代码中添加 //# sourceURL= xxx
  * 然后就会被浏览器在执行的时候解析，并且在调试面板中生成对应的文件目录，方便我们调试代码
  * 虽然可以知道是哪个文件，但是`在哪行`，`有时候是不准确的`



**devtool : source-map**

* 会生成完整的source-map，但是打包或者运行代码时相比于上面的速度会很慢

# 2.Babel

## 2.1.定义

* Babel是一个工具链，主要用于旧浏览器或者环境中将ES6+的代码转换为向后兼容的javascript
* 包括：语法转换、源代码转换、Polyfill实现目标环境缺少的功能等



## 2.2.单独使用Babel

* **Babel可以作为一个单独的工具来使用**，可以不和webpack等构建工具配合来使用

* 如果想要在命令行单独使用，需要安装以下的库

  * `@babel/core` : babel的核心代码，必须安装

  * `@babel/cli`：可以让我们在命令行中使用Babel

  * ```
    npm i @babel/core @babel/cli -D
    ```

**使用babel来处理源代码**

* `--out-dir` : 指定要输出的文件夹dist
* `--plugins=xxx,xxx`：要使用的插件，多个插件之间用逗号隔开

```
npx babel ./src --out-dir ./dist --plugins=@babel/plugin-transform-block-scoping

使用babel来转换src文件夹下的所有代码，将转换完后的代码输出到dist目录下，转化的时候使用的是xxx插件
注意：使用的插件是需要自行安装的
```



## 2.3.babel的预设插件

* 上面的示例中可以看到，每转换一种语法(es6->es5)都要安装一个插件
  * 比如：转化const、let需要安装@babel/plugin-transform-block-scoping
  * 转化 尖头函数需要安装@babel/plugin-transform-arrow-functions
  * 等等
* 这样如果要转换的语法多了，那么命令行中需要写很多个插件，太繁琐了
* 
* **所以Babel提供了Babel preset（babel预设）**
* `npm i @babel/preset-env -D`

```
npx babel ./src --out-dir ./dist --presets=@babel/preset-env
```

## 2.4.babel的底层原理

* 其实Babel就可以看做一个编译器
* Babel也拥有编译器的工作流程
  * 解析阶段(parsing)
  * 转换阶段(transformation)
  * 生成阶段(code generation)

## 2.5.babel-loader

* 实际开发中，我们通常会在构建工具中通过配置babel来对其进行使用，比如在webpack中
* 那么就需要安装相关的依赖了`(安装babel-loader的时候会自动安装@babel/core)`
  * `nmp i babel-loader @babel/core`

**配置babel**

* 上面案例中我们知道，`默认使用Babel是不会转换es6->es5代码的`
* 需要在执行命令的时候告诉Babel使用什么plugin来转换，webpack中也是一样

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            // 默认情况下 使用babel-loader 不会转换es6的代码，需要配置
            // plugins: ['@babel/plugin-transform-arrow-functions'],
            presets: ['@babel/preset-env'],
          },
        },
      },
    ],
  },
}
```

## 2.6.常见的babel-preset

* 如果我们一个个去安装使用插件，那么需要手动来管理大量的babel插件，所以可以直接`给webpack提供一个preset`，webpack会`根据提供的preset`来`加载`对应的`插件列表`，并将其传递给babel



**常见的preset有三个**

* **env**
* **react**
* **TypeScript**



**安装env**

* `npm i @babel/preset-env`

## 2.7.浏览器兼容

* 开发中如果处理浏览器的兼容问题呢？

  * 这里说的浏览器兼容`不是指屏幕大小的变化适配`

  * 这里指的兼容性是`针对不同的浏览器支持的特性`，比如css特性，js语法特性直接的兼容

  * https://caniuse.com/usage-table 可以通过这个网站查看浏览器的市场占有

* 可以通过`broswerslist`工具来处理兼容性的问题



### 2.7.1.**broswerslist**

* broswerslist是一个在`不同的前端工具之间`，`共享目标浏览器和Nodejs版本的配置`
* broswerslist会查询上面的网站，然后`共享给不同的插件告诉它们要适配哪些浏览器`
* 下面的插件都会使用broswerslist来查询
  * Autoprefixer
  * Babel
  * postcss-preset-env
  * eslint-plugin-compat
  * ...
  

**举例子：**比如我们上面说到babel preset-env 会根据环境来给我们自动选择需要哪些插件来解析代码，其实他的环境来源就是browserslist

### 2.7.2.broswerslist编写规则

**下面数字都可以替换的**

* **defaults**：broswerslist的默认浏览器（>0.5%, last 2 versions, Firefox ESR, not dead）
  * 这是browserslist的默认配置，可以在配置文件中写defaults，也可以不写配置文件 都有一样的效果
  * `表示市场占有率>0.5%,最后两个版本，没有淘汰的都要兼容`
* **5%**：场占有率是5%，也可使用>=、>、<、<=
* **dead**:24个月内没有官方支持或者更新的浏览器，都表示dead了
* **last 2 versions:** 每个浏览器的最后2个版本
  * last 2 Chrome version: 最后两个版本的chrome浏览器
* **ios 7** ：直接指定浏览器的版本

* ...

### 2.7.3.使用browserslist

* 安装babel的时候会自动下载browserslist
* `npm i browserslist`



**命令行使用**

```
 npx browserslist ">50%,last 5 versions, not dead"
```



**配置文件中使用(开发中使用)**

* 常见的配置文件有`两种`
* **package.json文件**
* **.browserslistrc文件**

package.json

```json
{
  "name": "01-source-mp",
  "version": "1.0.0",
  ...,
    "browserslist": [
    "> 0.1%",
    "last 2 version",
    "not dead"
  ]
}
```

.browserslistrc文件

```
> 5%
not dead
last 2 versions
```



**注意**

上面的每个规则`用逗号`隔开或者`换行写`都表示`or`的意思

```
> 5%
not dead
last 2 versions

等价于or（或）
> 5% or not dead or last 2 versions

还可以使用and（且）
> 5% and last 2 versions

还可以使用 not （非）
> 5%  not last 2 versions
```

## 2.8.Babel的配置文件

* 上面配置babel都是在`webpack.config.js`中的，那么如果`babel的配置一多`，会导致webpack.config.js文件`难以维护`
* **所以可以将babel的配置单独放在一个配置文件中**
* 配置文件名有两种方式
  * **babel.config.json**(推荐这种写法)
    * 或者以.js、.cjs、.mjs结尾
    * `eg: babel.config.js`
  * **.babelrc.json**(早期的写法)
    * 或者是.babelrc、.js、.mjs
    * `eg:.babelrc`
* **运行时webpack会自动读取babel.config.js文件然后将配置项给到babel-loader**



**babel.config.js**

```javascript
module.exports = {
  // 默认情况下 使用babel-loader 不会转换es6的代码，需要配置
  // plugins: ['@babel/plugin-transform-arrow-functions'],
  presets: ['@babel/preset-env'],
}

```

**webpack.config.js**

* **运行时webpack会自动读取babel.config.js文件然后将配置项给到babel-loader**

```javascript
/** @type {import('webpack').Configuration} */
module.exports = {
 ...
  module: {
    rules: [
      {
        test: /.js$/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },
}
```

## 2.9.pollyfill

### 2.9.1.定义

* 可以将pollyfill理解为`填充物（垫片）`，一个`补丁`，可以帮助我们更好的使用JavaScript
* 在高级代码转换低级原代码的时候，对于低版本js没有的api，pollyfill会将其添加进来
* **举个栗子**
  * 当我们写一个es6的语法new promise 和 string.includes()的时候要将其转为es5的语法
  * 用babel转换的时候，是`不会转换的` 因为new 本来就是一个普通的语法，string.includes()本来就是一个普通的方法调用，babel转换的时候`只会将高级语法转为低级语法`(const、let -> var)，然后跑在低版本的浏览器上就会报错
  * 这时候`可以利用pollyfill给js打一个补丁`，它会给js添加`(填充)`一个 function Promise ，添加`(填充)`一个string.prototype.includes方法
  * 那么在低版本的浏览器上调用new promise或string.includes的时候其实就是调用pollyfill提供的方法

### 2.9.2.使用

**安装**

* **babel7.4.0**之后的安装方式都是以下两个包
* `npm i core-js regenerator-runtime`

**配置**

* 在babel的配置中(独立文件或webpack)

* 案例中演示的是`独立文件`

* ```javascript
  // 说明为啥presets中既可以写string又可以写array
  // 写array是为了，给当前的preset设置更多的配置
  // 前面说到babel里面有很多的preset
  module.exports = {
    presets: [
      [
        '@babel/preset-env',
        {
          corejs: 3, // corejs的版本
          useBuiltIns: 'usage',
        },
      ],
    ],
  }
  ```

* 

**配置属性和值的含义**



**corejs**

* 表示core-js的版本 ，每个版本的打包方式是有区别的



**useBuiltIns**：打包的时候如果使用polyfill

* `false`
  * 打包的文件不使用polyfill来进行适配
  * 并且这个时候是不需要设置corejs属性的
* `usage`
  * 会根据代码中出现的语言特性，自动检测所需要的polyfill
    * 比如转换的代码有new Promise 那么就会引入对于的polyfill
  * 这样可以确保最终包里的polyfill数量的最小化，打包的包相对会小一些
  * 可以设置coresjs属性来确定使用的corejs的版本
* `entry`
  * 如果我们使用了dayjs三方库，那么dayjs中也使用了新的特性，比如使用了includes方法
  * 但是因为我们使用的是usage，所以打包的时候是不会对dayjs进行polyfill操作的
  * 那么跑在低版本浏览器就会报错
  * 这时候可以使用entry
  * 但是需要在`入口文件(比如vue中在main.js)`引入`import 'core-js/stable';import 'regenerator-runtime/runtime'`
  * 会根据`browserslist`查询到要适配的浏览器，然后将这些浏览器需要的polyfill打包，但是这也会导致包会比较大

## 2.10.解析TypeScript

### 2.10.1使用ts-loader解析

* 如果希望在webpack中使用TypeScript，可以使用ts-loader来处理ts文件

  * ​	`npm i ts-loader`

* 配置ts-loader

* ```javascript
  module.exports = {
    module: {
      rules: [
        {
          exclude:'/node_modules'
          test: /\.ts$/,
          use: ['ts-loader'],
        },
      ],
    },
  
  }
  ```

* 

### 2.10.2使用babel-loader解析

* **开发中建议使用这个**
* 因为 ts-loader使用的是typescript- compiler，它里面是没有polyfill的
* 如果ts代码中使用比较新的特性，那么使用ts-loader打包的时候，这些新特性是不会转换的，最终在低版本的浏览器上是跑不起来的
* 如果使用babel-laoder的时候可以给ts添加polyfill

**使用**

* 可以使用插件：@babel/tranform-typescript
* `更推荐使用预设：@babel/preset-typescript`

**安装**

* `npm i @babel/preset-typescript -D`

**示例**

babel.config.js文件

```javascript
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        corejs: 3, // corejs的版本
        useBuiltIns: 'usage',
      },
    ],
    // 给typescript设置polyfill
    [
      '@babel/preset-typescript',
      {
        corejs: 3, // corejs的版本
        useBuiltIns: 'usage',
      },
    ],
  ],
}

```



### 2.10.3.ts-loader和babel-loader的选择

**区别**

* `ts-loader`
  * 直接来编译TypeScript，那么只能将ts转为js，且`会对类型进行检测`
  * `无法实现polyfill`
* babel-loader
  * 直接来编译TypeScript，可以将ts转为js，并且`可以实现polyfill的功能`
  * 但是在编译的过程中`不会对类型错误进行检测`

**选择**

* 真是开发中如果编写的ts代码`不需要兼容低版`本的浏览器(且浏览器可以使用新特性)，那么使用ts-loader即可

* 如果`希望兼容低版本浏览器`，那么可以babel-loader和ts-loader`结合使用`

  * 在打包之前`先使用`ts-loader来进行类型检测  `yarn ts-check`
  * `最后用`babel-loader来完成打包  `yarn ts-check-watch`

  ```json
  // package.json文件
  // 需要在脚本中添加一下两个命令来完成类型检测
  // --noEmit 表示不输出文件 只检测类型
  // --watch 表示实时监测文件的类型
   {
      "scripts": {
      "build": "webpack",
      "ts-check": "tsc --noEmit",
      "ts-check-watch": "tsc --noEmit --watch"
    },
   }
  
  ```

  

# 3.dev-server

## 3.1.webpack-dev-server

* 启动一个本地服务
* `npm i webpack-dev-server -D`
* webpack-dev-server在编之后不会写入到任何的输出文件，而是`将打包的文件保留在内存中`
  * 所以他的效率很高的
  * 事实上，webpack-dev-server使用了一个库叫memfs
* `当启动服务`，会将当前入口文件的内容进行打包然后将其`放在内存中`,我们访问的时候就是访问这一块内存里的文件

### 3.1.webpack-dev-server配置项

* 所有的配置项都在`devServer`对象中配置

#### static

* devServer.static对于我们直接访问打包后的资源其实并没有太大的作用，他主要的作用是用于我们打包后的资源又依赖其他的一下资源，那么就需要指定从哪里来找这个内容

* **举个例子**

  * 上面说到，webpack-dev-server打包的内容会放到内存中
  * 打包的文件中的html文件中会引入打包后的js文件\<script src="./boundle.js">
  * 但是我们在这个html文件中还引入了别的js(引入的可以是任何文件)文件，比如引入了content文件夹的abc.js
  * 这个文件是没有被打包的，因为入口文件没有配置这个
  * 那么当在浏览器中运行代码就会报错，内容是找不到这个文件

* 这时候就可以`配置static`来解决

* **注意**：在index.html文件中引入时候直接写文件名，不要写文件夹的名字，因为在static中已经配置了

  * \<script src="./abc.js">` 正确写法`
  * \<script src="./content/abc.js"> `错误写法`

* webpack会自动的找到当前文件夹进行匹配

* static的默认值是`static:['public']`

  * 也就是说如果是引入public文件夹中的静态资源是不需要配置的

* 但是`如果手动的配置了`static,那么`默认值就会失效`

* ```javascript
  module.exports = {
    mode: 'development',
    	...
    devServer: {
      // 如果有同名的文件在不同的文件夹 在前面的文件夹会匹配 后面的不会匹配
      static: ['content'],
    },
  
  }
  ```



#### host

* 设置主机地址
* 默认是`localhost`
* 如果希望同一个局域网下的其他设备也可以访问，可以设置`host:0.0.0.0`
  * 但是测试发现 不配置别的设备也可以访问

**localhost和0.0.0.0区别**

* localhost本质上是一个域名，通常情况下会被解析成127.0.0.1
* 127.0.0.1是一个回环地址(Loop Back Address)，变大的意思是我们主机自己放出去的包，直接被自己接收
  * 正常的数据的流程是 应用层->传输层->网络层->数据链路层->物理层
  * 而回环地址，在网络层就直接被截获了，不会经过数据链路层、物理层
* 0.0.0.0：监听IPV4上的所有地址，再根据端口找到不同的应用程序
  * 同一网段下的主机中，可以通过ip访问

#### open

* 编译成功后，是否自动打开浏览器
* 默认值：` false`

#### compress

* 是否为打包后的代码压缩
* 默认值是`true`

#### proxy

* 代理 解决跨域问题
* 可以参考之前写的nodejs代理解决跨域，原理都是一样的(因为他们用的都是`http-proxy-middleware`中间件)

```javascript
module.exports = {
    devServer: {
    static: ['public', 'content'],
    port: 8000,
    proxy: {
      '/api': {
        target: 'http://124.221.241.135:8888/',
        pathRewrite: {
          '^/api': '',
        },
      },
    },
  },
}
```

#### historyApiFallback

* 主要作用是解决spa页面在路由跳转之后，进行页面刷新，返回404的错误
* 默认是false
* **举个例子**
  * 我们在使用vue/react搭建项目的时候，如果选择了history路由模式，那么浏览器url是这样的http://abc.com/about
  * 这时候我们处于的是about这个模块，我们通过js来进行跳转是没问题的
  * 但是千万不能刷新页面，一刷新就会报错404
  * **原因：**因为我们使用的是history模式，当我们刷新的时候浏览器会默认打开http://www.abc.com这台服务器的about下的index.html文件，但是我是没有这个文件的，所以就会报错404
  * **解决：**将`historyApiFallback:true`即可
  * 之后我们刷新网页 浏览器会将/about当作是history的一部分来解析

# 4.webpack性能优化

* webpack的性能优化比较多，可以对其进行分类
  * **优化一**：`打包后的结果`，上线时的性能优化（比如：分包处理、减少包体积、CDN服务器等）
  * **优化二**：`优化打包速度`，开发或者构建时优化打包速度（比如exclude、cache-loader等）
* 大多数情况下，更加`侧重于优化一`，这对一线上的产品影响更大
* 大多数情况下webpack都帮我们做好了该有的性能优化
  * 比如配置mode为production或者development时，默认webpack都有做相关配置
  * 但是我们也可以针对性的优化自己的项目



## 4.1.代码分离(Code Splitting)

* 代码分离是webpack一个非常重要的特性,**这在性能优化中非常重要***（简称分包）
  * 它主要的目的就是`将不同的代码分离到不同的bundle(包)中(`就是打包的时候将不同的代码打包到单独的文件)，自后我们可以`按需加载`，或者`并行加载这些文件`(下载文件的时候不是单线程的下载而是多线程的下载)
  * 比如默认情况下，`所有的js代码`(业务代码、三方依赖、暂时没有用到的模块)`在加载首页的时候会全部加载`
  * 代码分离可以分`离出更小的bundle`，以及`控制资源加载的优先级`，提高代码加载的性能
  * 分包还`有利于浏览器的缓存策略`
    * 案例在下面的笔记中
* **webpack中常用的代码分离有三种**
  * `入口起点`：使用entry配置手动分离代码(`多入口`)
  * `防止重复`：使用Entry Dependencies或者SplitChunksPlugin去重和分离代码
  * `动态导入`：通过模块的内敛函数调用来分离代码

### 4.1.1.入口起点

* 其实就是让webpack在打包的时候`对多入口文件或者依赖单独打包`
* 比如有两个入口index.js和main.js 他们有不同的逻辑，打包的时候要将他们分开，就可以使用多入口
* 比如只有一个入口，但是不想让三方包(dayjs)也打包在当前代码中



**手动配置**

webpack.config.js

```javascript
module.exports = {
  mode: 'development',
  // 使用多入口 key是可以自定义的
  entry: {
    main: './src/main.js',
    index: './src/index.js',
  },
  output: {
    // 因为使用的是多入口，那么打包的时候是webpack不知道讲那个文件代码放到bundle.js中
    // 这时候可以使用占位符，会自动获取入口的那么然后命名
    filename: '[name]-bundle.js',
    path: path.resolve(__dirname, 'build'),
    clean: true,
  },
}

```

**上述代码中会存在问题**

* 问题一：两个入口如果引用了相同的包，比如都用了dayjs，那么`每个入口文件的包都会打包一份`dayjs(`主要问题`)
* 问题二：所有的依赖都是和入口文件放在一起，难以阅读

**解决**

* `将依赖单独打包`,然后每个入口引用即可

* `让入口的每个文件变成对象的形式`，这样可以添加很多配置项给每个入口文件
  * `import`: 启动时需加载的模块
  * `dependOn`:当前入口所依赖的入口
    * 比如：main.js文件依赖了axios和lodash,那么不可以直接在dependOn中写，要早entry中用一个变量(自定义名称)引用着他的依赖

```javascript
modules.exports = {
  entry: {
    main: {
      dependOn: 'shared',
      import: './src/main.js',
    },
    index: {
      import: './src/index.js',
      dependOn: 'shared2',
    },
    shared: ['axios', 'lodash'],
    shared2: ['dayjs', 'axios'],
  },
  devtool: false,
  output: {
    // 因为使用的是多入口，那么打包的时候是webpack不知道讲那个文件代码放到bundle.js中
    // 这时候可以使用占位符，会自动获取入口的那么然后命名
    // entry的key都是name
    filename: '[name]-bundle.js',
    path: path.resolve(__dirname, './build'),
    clean: true,
  },
}
```

### 4.1.2.动态导入(dynamic import)

* webpack提供了两种实现动态导入的方式
  * 第一种，使用ECMAScript中的`import()`语法来完成，目前`推荐的方式`
  * 第二种，使用webpack遗留的require.ensure，不推荐
* **使用动态导入的包，打包的时候会单独打包出一个包**



**案例**

* webpack打包的时候发现用了import()语法，那么就认为这是个动态导入，所以会单独打包(假设叫a包)
* 下面代码在浏览器运行的时候是不会下载a包的，只有点击btn的时候才会下载a包
* 这个就是vue中懒加载的原理

```javascript
// index.js

const btn = document.createElement('button')
btn.textContent = '加载a文件'

document.body.append(btn)

btn.onclick = () => {
  // 添加魔法注释 在对分包命名的时候的name就是我们的魔法注释名
  import('/*webpackChunkName:'哈哈router'*/./router/a')
}

```



**分包命名**

* 对分出独立的包进行命名
* `chunkFilename`

```javascript
module.exports = {
    output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, './build'),
    // 针对分包的文件命名
    // 这个name是可以更改的，在impot()的时候添加魔法注释即可
    chunkFilename: '[name]-chunk.js',
    clean: true,
  },
}
```

### 4.1.3.自定义分包

* 这种分包模式是`splitChunks`，它底层使用的是`SplitChunksPlugin`来实现的

  * 改插件webpack以及默认安装和集成，只需要`提供SplitChunksPlugin相关的配置信息`即可

* webpack提供了SplitChunksPlugin默认的配置，我们也可以i手动来修改他的配置

  * 比如默认配置中，chunks仅仅对于异步（`async`）请求，我们也可以设置为`all`
    * `import()`就属于异步,对于import语法的引入就会单独打包
  * 如果给设置了`all`，记得入口文件名字就不能写死，`要用占位符`

  ```javascript
  module.exports = {
    entry: './src/index.js',
    devtool: false,
    output: {
      filename: '[name]-bundle.js',
    },
    // 优化配置
    optimization: {
      splitChunks: {
        chunks: 'all',
      },
    },
  }
  ```

  

**splitChunks部分属性**

地址：https://webpack.docschina.org/plugins/split-chunks-plugin/#splitchunkschunks

*  `maxSize` （byte）
  *  当一个包大于指定的大小时继续拆包,
* `minSize`（byte）
  * `默认值20000`
  * 拆的包最小不能小于minSize 但是如果一个包是一个整体的话它大于了maxSize 是拆不了的 如果这个包引用了其他依赖可以拆
* `cacheGroups`
  * 自定义分组，我们可以自定义匹配规则
  * 注意要打包的文件最小是minSize的默认值哦，不然分包不了，当然也可以更改minSize默认值



```javascript
module.exports = {
  entry: './src/index.js',
  devtool: false,
  output: {
    filename: '[name]-bundle.js',
  },
  // 优化配置
    optimization: {
    splitChunks: {
      chunks: 'all',
      // // 当一个包大于指定的大小时继续拆包
      // maxSize: 20000,
      // // 拆的包最小不能小于minSize 但是如果一个包是一个整体的话它大于了maxSize 是拆不了的 如果这个包引用了其他依赖可以拆
      // minSize: 10000,
      cacheGroups: {
        vendors: {
          test: /[\//]node_modules[\//]/,
          filename: '[id]_vendors.js',
        },
        // 把所有abc文件夹下的文件单独拆包
        abc: {
          test: /abc/,
          filename: '[id]_abc.js',
        },
      },
    },
  },
}
```

### **4.1.4.chunkIds**

* 用于告知webpack模块的id采用什么算法生成
  * 使用占位符的时候`[id]` 这个属性就行修改id以什么算法生成
* 常见的值有三个
  * **natural**：按照数字的顺序生成id
  * **named**：`development下的默认值`，一个可读的名称id，就是把用到的文件夹和文件名拼接
  * **deterministic**：`production下的默认值`,确定性的，在不同的编译中不变的短数字
    * webpack4中没有这个值的
    * 如果使用`natural`，那么当我们修改过某个文件但是并没有修改改文件引入的三方包的时候，那么所有的文件都要重新打包一次，这是有点浪费性能的
    * 如果我们讲文件部署到浏览器上了，这时候我们修改了某些东西然后重新上线，那么到我们打开改网站的时候，浏览器会下载用到的包，在下载之前浏览器会先检查本地缓存时候有同名文件内容一致的包，如果有就使用缓存，假想一下，如果第一次部署的三方包叫1—xx.js 第二次由于修改了某些文件内容，然后重新打包了，名称换成了2-x x.js那么浏览器发现名称不一样了，所以又下载了一遍，这样浪费性能，而且影响页面加载时间
    * 如果使用`deterministic`就不会出现这种情况，只要打包的包中如果没有内容修改就不会重新打包
* 最佳实践
  * 开发过程中，推荐使用named
  * 打包过程中推荐使用deterministic

```javascript
module.exports = {
  optimization: {
    
    chunkIds: "named",
    
    splitChunks: {
      chunks: 'all',
    }
  },
}
```



### 4.1.5.解决注释的单独提取

* 默认情况下，webpack在进行分包时，有对包中的注释单独提取到一个文件中
* 原因是由于一个插件的默认配置所导致的`TerserPlugin`
* TerserPlugin插件是对webpack打包时js代码压缩一个插件
* **webpack5中以及默认集成了，无需下载**
* `在mode：production的情况下 extractComments默认值是true`,所以会提取一个注释文件

```javascript
// 直接引入即可 无需下载webpack5已经默认集成
const TerserPlugin = require('terser-webpack-plugin')
module.exports = {
  mode: 'production',
 
  // 优化配置
  optimization: {
    minimize: false, // 告知 webpack 使用 TerserPlugin 或其它在 optimization.minimizer定义的插件压缩 bundle。  development默认值是false production默认值是true
    // minimizer:允许你通过提供一个或多个定制过的 TerserPlugin 实例，覆盖默认压缩工具(minimizer)。
    minimizer: [
      new TerserPlugin({
        extractComments: false, //打包的时候不提取注释到单独文件
      }),
    ],
  },
}

```





### 4.1.6.prefetch和preload

* **webpack v4.6.0+ 增加了对预获取和预加载的支持**
* 在使用`import()加载文件时`，使用下面这些内置指令，来告知浏览器
  * `prefetch(预获取)`：
    * 比如我们进入一个网站，他有很多导航，那么默认使用import()导入的导航文件只有当点击了某块导航的时候，浏览器才下载、加载文件，这是消耗时间的，虽然下载的时间可以小的忽略不计
    * 这时候我们如果使用prefetch，那么就会等浏览器空闲的时候去下载这些文件，而不是等到用户点击某个导航才进行下载（用户浏览某个导航内容是需要时间的，所以可以利用这个空闲时间）
  * `preload(预加载)`
    * 如果我们浏览某个网页的某个导航页的内容，比如这个导航的页脚部分的代码没有和该导航的代码打包在一起，而是分包打开，但是我浏览当前的导航肯定是要加载页脚部分的不管用户看不看
    * 那么就可以使用preload，他会和当前导航文件`并行加载`
* **prefetch指令和preload指令区别**
  * 父chunk：比如主包是index.js  里面引入了分包b.js 那么index.js就是b.js的父 chunk
  * preload会在父chunk加载时，以并行的方式开始加载，prefect会在父chunk加载结束后开始加载
  * preload具有中等优先级，会在父chunk下载的时候立马下载，prefetch会在浏览器闲置的时候开始下载
  * preload会在父chun中立即请求，用在当下时刻，prefetch用于未来的某个时刻

```javascript
 import(/*webpackChunkName:'a-router'*/ /*webpackPrefetch:true*/ './router/a')
import(/*webpackChunkName:'b-router'*/ /*webpackPreload:true*/ './router/b')
```



### 4.1.7.runtime chunk

* **什么是runtime相关代码**
  * runtime相关的代码指的是在运行环境中，`对模块进行解析、加载、模块信息相关的代码`
  * 比如`我们有component、bar两个通过import函数相关的代码加载`，就是`通过runtime代码完成的`
* 所以在打包代码的时候`会将运行时代码和业务代码一起打包`,这样会导致包的体积变大，所以可以将runtime chunk单独抽离出来
*  **抽离出来后，有利于浏览器缓存的策略:**
  *  比如我们修改了`业务代码(main)`，那么runtime和component、bar的chunk`是不需要重新加载的`
  * 比如我们修改了`component、bar的代码`，那么main中的代码是`不需要重新加载的`

* 设置的值:
  * `true/multiple`:针对每个入口打包一个runtime文件;
  * `single`:打包一个runtime文件;
  * `对象`:name属性决定runtimeChunk的名称;

```javascript
module.exports = {
  mode: 'production',
 
  // 优化配置
  optimization: {
 		runtimeChunk:{
      name:'runtime'
    }
  },
}
```



## 4.2.CDN加速

### 4.2.1.什么是CDN

* CDN全称Content Delivery Network，即`内容分发网络`。
* CND加速`主要是加速静态资源`，如网站上面上传的图片、媒体，以及引入的一些Js、css等文件。
* CND加速需要依靠各个网络节点，例如100台CDN服务器分布在全国范围，从上海访问，会从最近的节点返回资源,提升资源下载速度，提高网站访问速度。

### 4.2.2.开发中使用CDN方式

**CDN服务器需要付费购买哦 看公司财力决定要不要使用CDN加速**

* **开发中，使用CDN主要有两种方式**
  * `方式一`：打包所有的静态资源，放到CDN服务器，用户访问所有资源都是通过CDN服务器加载的
  * `方式二`：一些三方资源放到CDN服务器上(比如axios、dayjs的下载)

**方式一**

* 如果使用方式一，那么可以直接修改过publicPath

* webpck在打包的时候，会在html引入script的时候拼接上这个地址

* ```javascript
  module.exports = {
    	...
      output: {
      publicPath: 'http://abc.cba.com',
    },
  }
  
  // 打包的html引入打包文件自动拼接地址
  <script defer src="http://abc.cba.com/main-bundle.js"></script>
  
  ```




**方式二**

* 通常一些`比较出名的开源框架`都会将打包后的源码放到一些`比较出名的、免费`的CDN服务器上
  * 国际上使用较多的是unpkg、JSDelivr、cdnjs
  * 国内比较好用的是`bootcdn`
* **使用**
  * 在项目中我们需要在配置文件中的`externals`属性配置哪些三方资源是需要cdn引入的
  * 然后`在html模版中`将这些三方资源的cdn地址(`去查bootcdn上面查`)引入即可，注意是在html模版中引入，不是在打包后的html引入，那样每次打包都要引入

`webpack.config.json`

```javascript
module.exports = {
  mode: 'development',
    externals: {
    // key表示的意思：表示我们在自己项目中引入的三方包的名称
    // eg：import http from 'axios'  那么axios就是本地引入的名字

    // value:表示从CDN地址请求下来的js中提供对应的名称
    //eg：cdn上lodash的全局变量叫 _  jquery的全局变量叫 $
    axios: 'axios',
    lodash: '_',
  },
}
```

`模版index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body></body>
<!--在这里引入-->
  <script src="https://cdn.bootcdn.net/ajax/libs/axios/1.3.6/axios.js"></script>
  <script src="https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.21/lodash.core.js"></script>
</html>

```

