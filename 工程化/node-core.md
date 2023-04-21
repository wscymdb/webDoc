# 1.内置模块

## 1.1.api说明

* api一般分为三种(以fs模块举例)
  * 同步(sync)
    * fs.readFileSync()
  * 异步(callback)
    * fs.readFile(path,cb)
  * 异步(promise)
    * fs.promise.readFile()



## 1.2. fs模块

### 1.2.1.文件描述符

* 在常见的操作系统上，对于`每个进程，内核都维护着一张当前打开着的文件和资源的表格`
* 每打开的文件都分配了一个`称为文件描述的简单的数字标识符`
* 在系统层，`所有文件系统操作都使用这些文件描述符来标识和跟踪每个特定的文件`
* `window系统使用了一个虽然不同但是概念上类似的机制来跟踪资源`
* **为了简化用户的工作，Node抽象出操作系统之间的特定差异，并为所有打开的文件分配了一个数字型的描述文件**



* 拿到这些描述符，`就可以在打开文件后获取到文件的信息`
* `fs.open()`方法用于分配新的文件描述符
  * 一旦被分配，则文件描述符可用于从文件读取数据，向文件写入数据、或请求文件的信息

 

```javascript
const fs = require('fs')

// 为什么不直接打开文件，而要拿到文件的操作符再打开文件，多此一举？
// fs.readFile只能读取文件的内容，但是却读取不到文件的信息(大小、创建时间...)
fs.open('./assets/b.txt', (err, fd) => {
  // 拿到文件的描述符
  console.log(fd)

  // 读取文件的信息
  fs.fstat(fd, (err, stats) => {
    if (err) return

    console.log(stats)

    // 手动关闭文件(虽然进程结束也会关闭当前文件，但是，但是要养成良好习惯)
    fs.close()
  })
})

```

### 1.2.2.文件操作

#### 1.2.2.1. readFile 读取

* fs.readFile(path[,options],cb)
* 读取文件的内容
* 

```javascript
const fs = require('fs')
// 同步读取
const res1 = fs.readFileSync('./assets/a.txt', { encoding: 'utf-8' })
console.log(res1)
console.log('--------------------')

// 异步读取  callback
fs.readFile('./assets/a.txt', { encoding: 'utf-8' }, (err, data) => {
  if (err) return console.log(err)
  console.log(data)
  console.log('--------------------')
})

// 异步读取  promise
fs.promises.readFile('./assets/a.txt', { encoding: 'utf-8' }).then((res) => {
  console.log(res)
})

```

#### 1.2.2.2.writeFile 写入

* **fs.writeFile(path,data[,options],cb)**



**options选项(部分)**

* flag：写入选项
* encoded： 编码方式 `默认utf-8`



**flag常见的值**



**如果是一个只读的话，那么如果放在写入的API中就会报错**

* w(默认)：打开文件写入,如果不存在则创建文件
* w(write)+：打开文件进行读写（可读可写），如果不存在则创建文件
* r(read)：打开文件读取，读取时的默认值
* r+：打开文件进行读写，如果不存在则抛出异常
* a(append缩写)：打开要写入的文件，将流放在文件末尾，如果不存在则创建文件
* a+：打开文件进行读写（可读可写），将流放在文件末尾，如果不存在则创建文件

```javascript
const fs = require('fs')

const text = '我是大帅哥12312'

fs.writeFile(
  './assets/c.txt',
  text,
  {
    flag: 'a+',
  },
  (err, d) => {
    if (err) return console.log(err)
    console.log('写入成功', d)
  }
)

```

### 1.2.3.文件夹操作

#### 1.2.3.1.mkdir 新建

* 如果存在该文件夹，则会报错

```javascript
fs.mkdir('./image', (err) => {
  if (err) return console.log(err)
})
```



#### 1.2.3.2.readdir 读取

```javascript
// 读取文件夹
fs.readdir(
  './image',
  {
    // 当前文件的类型 如果为true  则返回的是一个Dirent类
    // https://nodejs.org/docs/latest-v16.x/api/fs.html#class-fsdirent
    withFileTypes: true,
  },
  (err, file) => {
    if (err) return console.log(err)
    console.log(file)

    file.forEach((item) => {
       // isDirectory 是否是文件夹
      console.log(item.isDirectory())
    })
  }
)

```

#### 1.2.3.3. rename 重命名

不仅可以对文件夹还可以对文件进行重命名

```javascript
const fs = require('fs')

fs.rename('./image', 'ym', (err) => {
  console.log(err)
})
```

## 1.3. events模块

* Node中的核心API都是基于异步事件驱动的
  * 在这个体系中，某些对象（发射器（Emitters））发出某一个事件
  * 我们可以继续监听这个事件（监听器Listeners），并且传入回调函数，这个回调函数会在监听事件时调用
* 发出事件和监听事件都是通过EventEmitter来完成的，他们都属于events对象
  * `emitter.on(eentName, listerner)`：监听事件，也可以使用emitter.addListener
  * `emitter.off(enentName,listener)`：移除事件，也可以使用emitter.removeListener
  * `emitter.emit(eventName[,..args])`：发出事件，可以携带一些参数

```javascript
const EventEmitter = require('events')

// 创建实例
const emmiter = new EventEmitter()

// 监听事件
emmiter.on('ym', ymE)

function ymE(res) {
  console.log(res)
}
// 发射事件
setTimeout(() => {
  emmiter.emit('ym', [2, 3, 4, 5])
  console.log(emmiter)
  //取消事件
  emmiter.off('ym', ymE)
  console.log(emmiter)
}, 2000)
```

### 1.3.1.常见的方法

* `emmiter.eventNames()`：返回当前Event Emitter对象注册的`事件名字组成的 字符串数组`
* `emitter.getMaxListeners()`：返回当前Event Emitter对象的`最大监听器数量`，可以通过setMaxListeners()来修改，`默认是10`
* `emitter.listenerCount(eventName)`：返回当前EventEmitter对象某一个事件名称，`监听器的个数`
* `emitter.listerers(eventName)`：返回当前Event Emitter对象某个事件监听器上`所有的监听器数组`
* `emitter.once(eventName,listener):` 只监听一次
* `emitter.prependListener(eventName,listener)`: 将此次事件添加到整个事件组的第一个
* `emitter.removeAllListener([eventName])`:
  * 无参数时，会将所有的事件监听全部移除
  * 有参数时，只会移除事件名称对应的所有事件

## 1.4.Buffer

* Node为了方便开发者完成更多的功能，提供了一个类Buffer，它是全局的

### 1.4.1.二进制

*  `Buffer中存储的数据都是二进制数据`
  * 可以将buffer看成是一个存储二进制的数组
  * 这个数组中的每一项，都可以`保存8位二进制`(会将其转成16进制的，因为二进制的话不方便展示与阅读)
* 为什么是8位
  * 在计算机中，很少情况会直接操作一位二进制，`因为一位二进制存储的数据非常有限`
  * 所以通常会`将8位合在一起作为一个单元`，这个单元称为`一个字节（byte）`
  * 也就是说1byte=8bit  1kb = 1024byte  1M=1024kb

### 1.4.2.buffer转字符串

* 计算机中 一个英文占一个字节
* 一个中文（生僻字除外）占3个字节

```javascript
const buf2 = Buffer.from('world')
console.log(buf2)
// <Buffer 77 6f 72 6c 64>
// 1. 先将 w 根据ASCII编码转成二进制
// 2. 将二进制转为16进制放入到buffer数组中

const buf2 = Buffer.from('你')
console.log(buf2)
// <Buffer e4 bd a0>
// 1. 先将 你  根据utf-8编码转成二进制
// 2. 将二进制转为16进制放入到buffer数组中

```



## 1.5.Stream

* 程序中的流，可以想象成当我们从一个文件读取数据时，文件的二进制（字节）数据会源源不断的被读到我们程序中
* 而这一连串的字节，就是程序中的流



**为什么不直接通过readFile或writeFile来读写文件呢**

* 直接`读写文件的方式`，`虽然简单`，但是`无法控制一些细节的操作`
* 比如`从什么位置开始读取`、`读到什么位置`、`一次读多少个字节`
* 读到某个位置后，`暂停读取`、某个时刻`恢复读取`等
* 或者这个`文件非常大`、比如一个视频文件、一次性全部读取并不合适

### 1.5.1.文件读写的Stream

* 事实上Node中很多对象都是基于流实现的
  * http的Request和Response等
* 官方文档：`另外所有的流都是EventEmitter的实例`
* Node中有4种基本流类型
  * `Writable`：可以向其写入数据的流（例如：fs.createWriteStream()）
  * `Readable`：可以从中读取数据的流（例如：fs.createReadStream()）
  * `Duplex`：可读可写的流（例如：net.Socket）
  * `Transform`：在读写的过程中对流进行修改（例如：zlib.createDeflate()）

### 1.5.2.Readable 可读流

**可读流的基本使用**

```javascript
const fs = require('fs')

// start :从什么位置开始读取
// end：读取到什么位置（包括end）
//highWaterMark：3  每次读取3个字节 当读到3个字节后就回调函数
const readStream = fs.createReadStream('./demo.txt', {
  start: 6,
  end: 20,
  highWaterMark: 3,
})

// 因为所有的流都是继承EventEmitter所以可以用EventEmitter的方法（看源码接口）
readStream.on('data', (res) => {
  console.log(res.toString())

  // 暂停读取
  readStream.pause()

  // 一秒之后在读取
  setTimeout(() => {
    readStream.resume()
  }, 3000)
})
```



**其它事件的监听**

```javascript
// open事件
// 有一个参数  fd 当前文件的fd
readStream.on('open', (fd) => {
  console.log('通过流将文件打开！', fd)
})

// end事件 必须要监听data事件
readStream.on('end', () => {
  console.log('读取文件结束')
})

//close事件
readStream.on('close', () => {
  console.log('文件被关闭')
})
```

### 1.5.3.Writable 可写流

```javascript
const fs = require('fs')

const writeStream = fs.createWriteStream('./a.txt', { flags: 'a+' })

// 可以调用多个write写入
// write 方法不会自动调用close
// 当写入完毕后 需要手动调用close事件
writeStream.write(' cym')
writeStream.write(' is')
writeStream.write(' handsome')
writeStream.write(' man', (err) => {
  if (err) return console.log(err)
  console.log('写入完成')
  // writeStream.close()
})

// end 方法 会自动调用close
// 做了两件事 1.将内容写入 2.关闭文件

writeStream.end(' 哈哈哈')

//文件关闭事件
writeStream.on('close', () => {
  console.log('文件被关闭')
})

// 文件打开事件
writeStream.on('open', (fd) => {
  console.log('文件被打开', fd)
})

// 当文件写入完成事件
writeStream.on('finish', () => {
  console.log('finish')
})

```

### 1.5.4.pipe方法

* 将读取到的流全部写入到文件中

```javascript
const fs = require('fs')

// 拷贝文件的方式

// 方式一：一次性读取和写入
// 缺点 ： 没法精准控制；文件太大时不合适一次性读取和写入
// fs.readFile('./a.txt', (err, data) => {
//   if (err) return console.log(err)
//   fs.writeFile('./a copy.txt', data, (err) => {
//     if (err) return console.log(err)
//   })
// })

// 方式二  使用可读流和可写流
const readStream = fs.createReadStream('./a.txt')
const writeStream = fs.createWriteStream('./a copy.txt')

// readStream.on('data', (data) => {
//   writeStream.end(data)
// })

// 方式三 在可读流和可写流之间建立一个管道
// pipe 方法会将读取到的流全部写入到文件中
readStream.pipe(writeStream)

```

## 1.6.http模块

### 1.6.1.创建服务器的基本使用

 

```javascript
const http = require('http')

// 创建一个http服务器
const server = http.createServer((req, res) => {
	// res的本质就是一个写入流，所以可以使用end来写入
  res.end('hhh')
})

// 监听端口
server.listen(3000, () => {
  console.log('3000端口监听成功')
})
```

**注意事项**

* 如果通过浏览器访问服务 会访问两次  这是浏览器的特性

* 第一次访问服务本身 第二次访问服务的iconfavor(图标)

### 1.6.2.http请求发送网络请求

```javascript
const http = require('http')

// 使用http模块发起get请求
http.get('http://www.baidu.com', (res) => {
  // res是一个流
  res.on('data', (data) => {
    console.log(data.toString())
  })
})

const req = http.request(
  {
    method: 'POST',
    hostname: '127.0.0.1',
    port: 8000,
  },
  (res) => {
    res.on('data', (data) => {
      console.log(data.toString())
    })
  }
)
// 使用request  需要手动调用end  表示当前请求完成
req.end()
```



### 1.6.3.解析query类型的数据

```javascript
const server = http.createServer((req, res) => {
  // 解析query类型的参数

  // 1.解析url
  const urlStr = req.url
  const userInfo = url.parse(urlStr)

  // 2.解析query
  const queryString = userInfo.query
  const queryInfo = new URLSearchParams(queryString)
  console.log(queryInfo)
  res.end('hello  world')
})
```



### 1.6.4.解析body类型的数据

```javascript
const server = http.createServer((req, res) => {
  // 解析body类型的参数
  // body类型的参数本来就是请求头 请求体不会存在req中
  // req的本质就是stream的EventEmitter  所以可以有事件监听
  // body类型的参数是放在data事件里的

  // 设置编码格式
  req.setEncoding('utf-8')
  let data = null
  req.on('data', (res) => {
    console.log(res)
    // console.log(JSON.parse(res))
    data = JSON.parse(res)
  })
  req.on('end', () => {
    res.end(JSON.stringify(data))
  })
})
```





### 1.6.4.文件上传

```javascript
// 下面的方式的本质就是 拿到客户端传入的数据 将非图片的数据摘出去 只留下图片相关的数据进行保存
//了解即可  正式开发中框架都帮我们封装好了

// 创建一个http服务器
const server = http.createServer((req, res) => {
  // 设置编码格式  binary不是转为二进制  而是使用ISO-8859-1 进行编码
  req.setEncoding('binary')
  //bounary 客户端请求的时候自动生成的
  const bounary = req.headers['content-type'].split('=')[1]

  let formData = ''
  req.on('data', (data) => {
    formData += data
  })
  req.on('end', () => {
    let imgType = 'image/png'
    formData = formData
      .substring(formData.indexOf(imgType) + imgType.length)
      .replace(/^\s\s*/, '')
    formData = formData.substring(0, formData.indexOf(`--${bounary}--`))
    //写入文件 用binary编码方式进行写入
    fs.writeFile('./a.png', formData, 'binary', () => {
      console.log('图片上传完成')
      res.end('file success')
    })
  })
})
```

# 2.express

## 2.1.安装

**脚手架安装方式**

```javascript
安装脚手架
npm i -g express-generator
创建项目
express projecr-name
安装依赖
npm install
启动项目
node bin/www
```

**手动配置**

 



## 2.2.中间件

* 中间件的本质是传递给express的一个回调函数
* 这个回调函数接受三个函数
  * 请求对象（request对象）
  * 响应对象（response对象）
  * next函数（在express中定义的用于执行下一个中间件函数）

**中间件可以做什么**

* 执行任何代码
* 更改request和response对象
* 结束请求-响应周期（返回数据）
* 调用栈中的另一个请求 



### 2.2.1自己编写中间件

* express主要提供了两种方式
  * app.use/router.use
  * app.methods/router.methods
  * methods指的是常用的请求方式，eg：app.get/app.post等



**总结**

* 当express接收到客户端发送的网络请求时，在所有的中间件中开始进行匹配
* 当匹配到`第一个符合要求`的中间件时，那么就会`执行该中间件`
* `后续`的中间件是`否会执行`，取决于上一个中间件是否调用`next方法`



**通过use注册的中间件**

```javascript
// 总结：当express接收到客户端发送的网络请求时，在所有的中间件中开始进行匹配
// 当匹配到第一个符合要求的中间件时，那么就会执行该中间件
// 后续的中间件是否会执行，取决于上一个中间件是否调用next方法

//通过use注册的中间件是最普通的 最简单的中间件
// 通过use组册的中间件 无论什么请求方式都可以被匹配(只要请求了就能调用此中间件)
app.use((req, res, next) => {
  console.log('normal middle')
})
```



**注册路径匹配的中间件**

* 注册路径匹配要求的中间件
* 路径匹配的中间件只对path进行限制，不会对method进行限制的

```javascript
app.use('/home', (req, res, next) => {
  console.log('match /home middleware')
  res.end('home data')
})
```



**注册匹配请求方法和路径的中间件**

```javascript
// 注册中间件：对请求方式 和请求路径进行匹配
// 使用 app.method(path,middleware)
app.get('/home', (req, res, next) => {
  console.log('match /home get middleware')
  res.end('home data')
})
```



**注册多个中间件**

```javascript

// 出了可以单独写多个中间件 还可以在一个中间件函数中写多个中间件
// 后面的中间件是否运行 取决于上一个中间件调用next否
app.get('/home', (req, res, next) => {
  console.log('match /home get middleware1')
  res.end('home data')
},(req,res,next) => {
  console.log('match /home get middleware2')
},(req,res,next) => {
  console.log('match /home get middleware3')
})
```

### 2.2.2.express内置中间件

**解析json类型数据**

```javascript
// 会将数据存放在body中
app.use(express.json())

app.post('/login', (req, res, next) => {
  console.log(req.body)
})
```



**解析x-www-form-urlencoded**

```javascript
// 会将数据存放在body中
app.use(
  express.urlencoded({
    extended: false, //true：使用node内置的qs模块(默认)，但是这个模块慢慢被废弃了;false:使用querystring模块
  })
)

// 编写中间件
app.post('/login', (req, res, next) => {
  console.log(req.body)
  res.end('login api')
})
```

### 2.2.3.三方中间件

**morgan**

* 将请求的日志记录下来

```javascript
// 应用第三方中间件
// 会将每次请求的记录放在目标路径文件中
const writeStream = fs.createWriteStream('./logs/access.log')
app.use(moragan('combined', { stream: writeStream }))
```

**multer**

解析文件

```javascript
// 应用一个三方 的中间件处理文件上传
const upload = multer({
  // 用这种方式可以自定义后缀名
  storage: multer.diskStorage({
    destination(req, file, cb) {
      cb(null, './uploads')
    },
    filename(req, file, cb) {
      cb(null, Date.now() + '-' + file.originalname)
    },
  }),
})

// 编写中间件
// 上传单文件
// upload.single() 会返回一个中间件,处理但文件  参数是客户端文件对应的key   eg；avatar：files
app.post('/upload', upload.single('avatar'), (req, res, next) => {
  console.log(req.file)
  // 如果上传的既有文件又有字段 那么字段会被放入到req.body中
  console.log(req.body)
  res.end('文件上传成功')
})

// 上传多文件
app.post('/photos', upload.array('photos'), (req, res, next) => {
  console.log(req.files)
  res.end('上传多张照片完成')
})
```

## 2.3.解析动态路由

```javascript
// 编写中间件
app.get('/test/:id', (req, res, next) => {
  const id = req.params.id
  res.end(`获取id${id}`)
})
```

## 2.4.响应数据的方式

* res.end()
* res.json()
  * json方法可以传入很多类型：object、array、string、boolean、number、null等，他们会被转换成json格式返回
* res.status()
  * 用于设置状态码
* https://www.expressjs.com.cn/4x/api.html#res

```javascript
// 编写中间件
app.get('/test', (req, res, next) => {
  // 返回数据的方式
  // res.end(),方法用的比较少
  //res.end('test api')

  // res.json(),用的做多
  // res.json({ code: 0, msg: 'success', data: 222 })

  // 用于设置http的状态吗
  res.status(200)
  res.end('创建用户成功')
})
```

## 2.5.路由

* 一个`Router实例`拥有`完整的中间件和路由系统`
* 因此，他也`被称为mini的app`（可以用app的功能）
* 开发中可以讲每个路由对象单独放在一个文件中

```javascript
const express = require('express')

const app = express()

// 将用户的接口定义在单独的路由对象中
// 每个路由都可以看作是一个迷你的app 可以使用app的方法（get\post...）
const userRouter = express.Router()
userRouter.get('/', (req, res, next) => {
  res.json('你好')
})
userRouter.delete('/:id', (req, res, next) => {})
userRouter.post('/', (req, res, next) => {})
userRouter.patch('/:id', (req, res, next) => {})

// 歌词路由
const lryicRouter = express.Router()
lryicRouter.get('/', (req, res, next) => {
  res.json('你好')
})
lryicRouter.delete('/:id', (req, res, next) => {})

// 使用路由
// 当path是users的时候才会执行userRouter中间件
app.use('/users', userRouter)
app.use('/lryic', lryicRouter)
// 监听端口
app.listen(8888, () => {
  console.log('8888端口监听成功')
})
```

## 2.6.静态资源服务器

* 可以添加多个静态资源
* 当在浏览器输入域名+端口时  默认去找 index.html 如果没有就静态资源文件夹中找(默认去最先添加的静态资源文件夹中找)

```javascript
const app = express()
// 部署静态资源
// 可以添加多个
// 当在浏览器输入域名+端口时  默认去找 index.html 如果没有就静态资源文件夹中找(默认去最先添加的静态资源文件夹中找)
//
app.use(express.static('./uploads'))

// 监听端口
app.listen(8888, () => {
  console.log('8888端口监听成功')
})
```

## 2.7.处理错误的中间件

* 一共有4个参数
* `第一个err是调用next传来的参数`

```javascript
app.post('/test', (req, res, next) => {
  const { username, password } = req.body
  if (!username || !password) {
    next(-1001)
  } else {
    res.end('test api')
  }
})

// 使用错误处理的中间件
// err 是当调用next传入的参数
app.use((err, req, res, next) => {
  const code = err
  let message = 'unknow error'
  switch (code) {
    case -1001:
      message = '没有输入用户名或密码'
      break
    case -1002:
      message = '用户名或密码错误'
      break
  }
  res.json({
    code,
    msg: message,
  })
})
```

# 3.koa

* 事实上，koa是express同一个团队开发的一个新的Web框架
  * 目前团队的核心开发者TJ的主要精力也在维护Koa，express已经交给团队维护了
  * Koa旨在为Web应用程序和API提供`更小、更丰富和更加强大的功能`
  * 相对于express具有`更强的异步处理能力`
  * Koa的`核心代码只有1600+行`，是一个`更加轻量级的框架`
  * 我们`可以根据需要安装和使用中间件`

## 3.1.koa基本使用

```javascript
const Koa = require('koa')

// 创建app对象
const app = new Koa()

// 注册中间件
app.use((ctx, next) => {
  ctx.body = 123
})

app.listen(8888, () => {
  console.log('8888端口监听成功')
})
```

## 3.2.中间件

* koa注册的中间件提供了两个参数`ctx和next`
* ctx：上下文(context)对象
  * koa并没有像express一样，将req和res分开，而是将他们作为ctx的属性
  * ctx代表一次请求的上下文对象
  * ctx.request：获取请求对象
  * ctx.response：获取响应对象 
* ctx`有两个请求对象和两个响应对象`
  * ctx.request：koa自己封装的
  * ctx.req：node封装的
  * ctx.response：koa自己封装的
  * ctx.res：node封装的
* next: 本质上是一个dispatch，类似于express的next

```javascript
app.use((ctx, next) => {
  // ctx.req  ctx.res  这是node封装的
  // ctx.request ctx.response 这是koa封装的
  // 所以可以 ctx.res.end() ...
  console.log(ctx)

  ctx.body = 23
})
```

## 3.3.区分请求方式和路径

* koa创建的app对象，注册中间件`只能使用use方法`
  * koa并`没有提供methods的方式来注册中间件`（eg：app.get()...）
  * 也`没有提供path中间件来匹配路径`（eg：app.use('/login',fn)）
* koa中区分路径和method分方式
  * `方式一：根据request自己来判断`
  * `方式二：使用第三方路由中间件`

### 3.3.1.手动区分

```javascript
app.use((ctx, next) => {
  console.log(ctx)
  const path = ctx.path
  if (path === '/users') return (ctx.body = 'users')

  ctx.body = 'network'
})
```

### 3.3.2.路由中间件

* 有两个流行的路由中间件，都可以使用，都需要下载使用
  * koa-roter  三方开发的，很久没有维护了
  * koajs/router 官方开发的

```javascript
const Koa = require('koa')
const KoaRouter = require('@koa/router')

const app = new Koa()

// 创建路由对象
// prefix 路由前缀 和express的 app.use('/users', userRouter)同理
const userRouter = new KoaRouter({ prefix: '/users' })

// 注册路由
userRouter.get('/', (ctx, next) => {
  ctx.body = 'users  get'
})
userRouter.delete('/:id', (ctx, next) => {
  ctx.body = 'users  delete'
})

// 使用路由 使用userRouter.routes() 将会返回所有注册的路由
app.use(userRouter.routes())

// koa中 当匹配不到请求的时候 都是会返回 not found的
// 但是对于没有注册的路由 也返回not found给开发者的提示不友好
// 用下面的方法 则会告诉用户错误的原因 Method Not Allowed
app.use(userRouter.allowedMethods())
```

## 3.4.参数解析方式

### 3.4.1.query和params

* 这两种参数类型一般都是get请求方法的
* 路由中已经封装好了的
* 分别放在`ctx.params`和`ctx.query`中

```javascript
// query
userRouter.get('/', (ctx) => {
  const query = ctx.query
  console.log(query)
  ctx.body = JSON.stringify(query)
})

// json格式解析body的信息 需要借助中间件（koa-bodyparser）
userRouter.post('/', (ctx) => {
  // 数据在ctx.request.body中
  console.log(ctx.request.body)

  ctx.body = 123
})

```

### 3.4.2.json和urlencoded

* 这两种方式需要借助三方中间件（koa-bodyparser）
* 需要安装
* `npm i koa-bodyparser`

```javascript
// json格式解析body的信息 需要借助中间件（koa-bodyparser）
userRouter.post('/', (ctx) => {
  // 数据在ctx.request.body中
  console.log(ctx.request.body)

  ctx.body = 123
})

// urlencoded格式解析信息 需要借助中间件（koa-bodyparser）
userRouter.post('/urlencoded', (ctx) => {
  // 数据在ctx.request.body中
  console.log(ctx.request.body)

  ctx.body = 234
})
```

### 3.4.3.formdata

* 需要借助三方中间件（@koa/multer和multer）
* `npm i @koa/multer  multer`

**非文件上传**

```javascript
// formdata格式的解析 需要借助中间件（@koa/multer和multer）
userRouter.post('/formdata', formParser.any(), (ctx) => {
  console.log(ctx.request.body)
  ctx.body = 345
})
```



**文件上传**

* 用法和express的multer一样
* 这里列举一个

```javascript

const formdata = new multer({ dest: './uploads' })

// 注册路由
const userRouter = new KoaRouter({ prefix: '/upload' })
userRouter.post('/', formdata.single('img'), (ctx, next) => {
  ctx.body = 'upload done'
})

```

## 3.5.静态服务器

* koa没有内置相关功能，需要使用三方库
* **npm install koa-static**
* 使用过程类似express

```javascript
const static = require('koa-static')

// 创建服务器
const app = new Koa()

// 设置静态资源
app.use(static('./uploads'))
```

## 3.6.响应数据类型

* koa中响应数据主要是ctx.body

* ctx.body允许的**类型**

  * string

  * Buffer

  * Stream

  * Object || Array

  * null
  * 如果`response.status尚未设置`，Koa会`自动设置`成200或204(返回值是null的时候)

* 也可以`手动设置状态码`
  * ctx.status = number

## 3.7.处理错误

* koa中处理错误和express中有点区别
* 因为koa中的next是不接受参数的
* 所以采用是事件发射和事件监听的方式

```javascript
// 注册路由
const userRouter = new KoaRouter({ prefix: '/users' })
userRouter.get('/', (ctx, next) => {
  const isAutho = false
  if (isAutho) {
    ctx.body = 'user get'
  } else {
    // 这里可以拿 创建服务的app对象 也可以拿ctx.app 他们俩是一样的
    // 但是如过每个路由对象被放在不同的文件夹中获取app还要传来传去，所以使用ctx.app方便
    // app本身是一个EventEmitter，所以可以发射自定义事件
    // 行业规范 错误的事件 发射error 当然可以不遵循
    // 要把当前的ctx传入 不然没法响应数据
    ctx.app.emit('error', -10001, ctx)
  }
})

// 真实开发中 可以把错误处理 单独抽到一个文件
// 那么就需要导出app 在处理错误文件中导入app
// 因为app是一个EventEmitter 所以可以监听事件
app.on('error', (code, ctx) => {
  const errCode = code
  let message = 'unknow error ~'
  switch (errCode) {
    case -10001:
      message = '未授权'
      break
  }
  ctx.body = {
    code: errCode,
    message,
  }
})
```

# 4.koa和express区别

## 4.1.架构设计上来说

* express是完整和强大的，其中帮我们内置了很多非常好用的功能
* koa是简洁和自由的，他只包含了最核心的功能，并不会对我们使用中间件进行任何限制
  * 甚至是在app中连最基本的get、post都没有提供
  * 我们需要通过自己或者路由来判断请求方式或其它功能

## 4.2.中间件的区别

* express和koa框架他们的核心都是中间件
  * 但是他们的中间件执行机制是不同的，特别是针对某个中间件中包含异步操作时

**koa和express同步执行**

* 由于`两者同步执行代码的顺序是一样的`，案例中使用的是`koa`的代码

* 先执行第一个中间件如果该中间件调用了next，那么就会执行第二个中间件
* 如果第二个中间件也调用了next 那么就会执行第三个中间件
* 然后当第三个中间件内容执行完毕后，在执行第二个中间件next之后的代码（如果有的话）
* 最后执行第一个中间件next之后的代码（如果有的话）
* 这种执行顺序的过程叫做`洋葱模型`
  * `express执行同步代码`的时候`符合洋葱模型`，异步的时候不符合
  * `koa同步异步都符合`

```javascript
app.use((ctx, next) => {
  console.log('koa middleware01')
  next()
  ctx.msg += 'aaa'
  ctx.body = ctx.msg // undefinedcccbbbaaa
})
app.use((ctx, next) => {
  console.log('koa middleware02')
  next()
  ctx.msg += 'bbb'
})
app.use((ctx, next) => {
  console.log('koa middleware03')
  ctx.msg += 'ccc'
})

// 监听
app.listen(8888, () => {
  console.log('8888端口监听成功')
})
```

**koa和express异步执行**

* express的异步执行

```javascript
// 异步代码
// 如果想要在第一个中间件中返回第三个中间件中异步处理后处理的结果
// express中很难办到，机制如此
// 那么只能在第三个中间件中等到结果后 在第三个中间件中返回结果
// 而不能在第一个中间件中返回异步处理后的结果x

// 如果像koa一样在每个中间件的next添加一个await呢
// express的next返回值是void koa中返回值是Promise<any>
// 所以这也就决定了 express即使在next前面加await 也没有用

// 编写中间件
app.use((req, res, next) => {
  console.log('express middleware01')
  req.msg = 'aaa'
  next()
  // res.json(req.msg) // aaabbbccc
})
app.use((req, res, next) => {
  console.log('express middleware02')
  req.msg += 'bbb'
  next()
})
app.use(async (req, res, next) => {
  console.log('express middleware03')

  // 异步代码
  const data = await axios.get(
    'http://180.76.235.241:3000/search?keywords=海阔天空'
  )
  req.msg += data.data.result.songs[0].name
  res.json(req.msg)
})
```

* koa的异步执行

```javascript
// 异步代码
// 默认情况下 如果next之后的代码中有异步操作，koa是不会等待异步结果出来的 而是直接执行下一步

// 解决  koa中的解决方案很简单
// 只需要在next 前面加 await即可
// 那么当前的中间件就变成了异步函数 那么上一个中间件要想等到当前中间件的结果
// 上一个中间件也要在next前面加await

app.use(async (ctx, next) => {
  console.log('koa middleware01')
  ctx.msg = 'aaa'
  await next()
  ctx.body = ctx.msg // aaabbb海阔天空
})
app.use(async (ctx, next) => {
  console.log('koa middleware02')
  ctx.msg += 'bbb'

  await next()
})
app.use(async (ctx, next) => {
  console.log('koa middleware03')

  // 异步代码
  const res = await axios.get(
    'http://180.76.235.241:3000/search?keywords=海阔天空'
  )

  ctx.msg += res.data.result.songs[0].name
})
```

# 5.MySQL-node中使用

## 5.1.数据库驱动

* 要在Node连接MySQL，执行SQL语句可以借助以下两个库
* **mysql**：最早的Node连接MySQL的数据库驱动，现在不在维护了
* **mysql2**：在mysql的基础之上，进行了很多的优化、改进
* 推荐使用mysql2
  * mysql2兼容mysql，并且提供了一些附加功能
  * 更快、更好的性能
  * Prepared Statement （预编译语句）
  * 支持promise

## 5.2.mysql2的使用

**安装**

```
npm install mysql2
```

**基本使用**

```javascript
const mysql = require('mysql2')

// 建立连接
const connection = mysql.createConnection({
  host: 'localhost',
  database: 'jack',
  port: 3306,
  user: 'root',
  password: 'qwerty123',
})

// 执行操作语句，操作数据库
const statment = `SELECT * FROM student`

connection.query(statment, (err, values, fields) => {
  if (err) throw err
  console.log(values)
  console.log(fields)
  
   // 销毁连接
  connection.destroy()
})
```

## 5.3.mysql2特性-Prepared Statement

* prepared statement(预编译语句)
* 为什么要使用预编译语句
* **提高性能**
  * 数据库接收到SQL语句都要经过`解析、优化、转化`三个步骤才能被执行，如果有很多客户端同时使用这一条SQL语句，那么这条SQL就会重复很多遍，会损耗性能
  * 如果使用预编译语句，会将创建的语句模版发送给MySQL，然后MySQL编译（解析、优化、转换）语句模版，`并且存储它，但是不执行`，之后当传递给`？`真正的值的时候才会执行，就算同时有很多客户端连接，那么也只会编译`？`前的语句`一次`，因为已经将编译好的模版存储起来了,所以性能是更高的
* **防止SQL注入**
  * 使用预编译，而其后注入的参数将不会再进行SQL编译。也就是说其后注入进来的参数系统将不会认为它会是一条SQL命令，而默认其是一个参数，参数中的or或者and 等就不是SQL语法关键字了，而是被当做纯数据处理

```javascript
// 执行操作语句，操作数据库
// 使用 ? 占位符
const statment = `SELECT * FROM student WHERE id < ? AND name = ?`

// 使用预处理语句 不能使用query了 query表示执行的是普通语句
// 要使用execute()
// 使用[]存放占位符真正的数据
connection.execute(statment, [4, '小明'], (err, values) => {
  if (err) throw err
  console.log(values)
  
   // 销毁连接
  connection.destroy()
})
```

## 5.4.mysql2特性- Connection Pools

* 当使用数据库直接连接的时候（上面创建的单个连接）
  * 会经过 1建立连接、2操作数据库、3关闭连接三个步骤
  * 在业务量不大的时候，并发量也不大的时候单个连接没啥问题
  * 但是当并发量大起来，达到百级、千级甚至更高的时候，其中建立连接、关闭连接的操作都会造成性能瓶颈，这时候就要考虑使用连接池来优化1和3步骤
* 好在，mysql2给我们提供了`连接池(connection pools)`
  * 连接池可以`在需要的时候自动创建连接`，并且创建的`连接不会被销毁`，`会放入连接池中`，后续可以继续使用
  * 可以在创建连接池的时候设置LIMIT，设置最大创建个数

```javascript
const mysql = require('mysql2')

// 建立连接池
const pool = mysql.createPool({
  host: 'localhost',
  database: 'jack',
  user: 'root',
  password: 'qwerty123',
  connectionLimit: 8, // 连接池连接的最大数量
})

// 执行操作语句，操作数据库
// 使用 ? 占位符
const statment = `SELECT * FROM student WHERE id < ? AND name = ?`

// 使用预处理语句 不能使用query了 query表示执行的是普通语句
// 要使用execute()
// 使用[]存放占位符真正的数据
pool.execute(statment, [4, '小明'], (err, values) => {
  if (err) throw err
  console.log(values)
})
```

## 5.5.Promise的形式

* 只需在调用执行语句前，调用promise()即可实现promise的返回结果
* promise的返回结果是一个数组，`[values,fields]`，第一项是查询的值，第二项是查询的字段

```javascript
// 执行操作语句，操作数据库
// 使用 ? 占位符
const statment = `SELECT * FROM student WHERE id < ? AND name = ?`

pool
  .promise()
  .execute(statment, [5, '小明'])
  .then((res) => {
    console.log(res[0])
  })
```

# 6.登陆凭证

## 6.1.cookie

**cookie一般是服务器帮助设置的，有些特殊情况客户端进行操作（删除cookie等）**



* Cookie(复数：Cookies)，又称之为“小甜饼”，`类型为“小型文本文件”`，某些网站为了辨别用户身份而存储在用户本地终端上的数据

  * 浏览器会在特定的情况下携带上cookie来发送请求，我们可以通过cookie来获取一些信息

    

* Cookie总是保存在客户端中，按在客户端中的存储位置，Cookie可以分为`内存Cookie`和`硬盘Cookie`

  * `内存Cookie`：由浏览器维护，保存在内存中，浏览器关闭时Cookie就会消失，其存在`时间是短暂的`

  * `硬盘Cookie`：保存在硬盘中，`有一个过期时间`，用户手动清理或者过期时间到期，才会被清理

    

* 如何判断一个cookie是内存cookie还是硬盘cookie

  * `没有设置过期时间`，默认情况下是内存cookie，在关闭浏览器的以后会自动删除
  * `有设置过期时间`，并且过期时间不为0或者负数的cookie是硬盘cookie，需要手动或到期时，才会删除
    * 所以给一个cookie设置过期时间是0或负数可以手动删除

### 6.1.1.cookie常见的属性

**cookie生命周期**

* 默认情况下cookie是**内存cookie**，也称之为**会话cookie**，也就是浏览器关闭时会自动被删除

* 可以通过设置以下属性来设置过期时间

  * **expires**：设置的是Date.toUTCString()，格式是：express=date-in-GMTString-format

  * **max-age**：设置过期时间，单位是秒钟，`max-age=60*60*24(一天后过期)`

    

**cookie的作用域：（允许cookie发送给哪些URL）**

* Domian：指定哪些主机可以接受cookie

  * 如果不指定，那么默认是`origin`，`不包括子域名`

  * 如果`指定Domain`，则`包含子域名`

  * ```
    当前是在 baidu.com/a?name=123发送的请求，
    如果不指定Domain那么 在百度的其它子域名下发送请求就不会携带cookie 
    比如。a.baidu.com/list 下就不会携带
    
    如果设置了 Domain=baidu.com. 那么a.baidu.com/list发起请求就会携带
    ```

    

### 6.1.2.客户端设置Cookie

**js直接设置或获取cookie**

```javascript
console.log(document.cookie)
```

**这个cookie会在会话(浏览器)关闭时被删除**

```javascript
document.cookie = "name=jack"
document.cookie = "age=19"
```

**设置cookie同时设置过期时间**

```javascript
document.cookie = "name=jack;max-age=10"  // 单位时s
```

### 6.1.3.服务端设置Cookie

* 下面案例以koa为例子

```javascript
* 服务器设置cookie
* 浏览器拿到cookie，自动设置在浏览器中
* 浏览器发送请求自动携带cookie

userRouter.get('/login', (ctx, next) => {
  ctx.cookies.set('slogan', 'ikun', {
    maxAge: 15 * 1000, // 这里的时间时毫秒
  })
  ctx.body = '登陆成功'
})

userRouter.get('/list', (ctx, next) => {
  const slogan = ctx.cookies.get('slogan')
  console.log(slogan)
  ctx.body = 'list'
})

```

## 6.2.session

**为什么要用session**

* 上面案例中基于cookie来实现凭证，但是凭证的内容都是明文的
* 在传输过程中很不安全，容易被伪造
* 所以session就孕育而生



**下面案例是基于koa演示**



* **在koa中可以使用三方中间件koa- session来实现session认证**
  * npm i  koa-session
  * 这个中间件是基于cookie进行的加密操作

* 单独使用session其实并不是很安全
* 所以要进行加密（加盐操作）
* 这样就形成了双重认证

```javascript
const Koa = require('koa')
const KoaRouter = require('@koa/router')
const koaSession = require('koa-session')

// 创建服务器
const app = new Koa()

// 注册路由
const userRouter = new KoaRouter({ prefix: '/users' })

//  设置session 其实就是一个中间件
app.use(
  koaSession(
    {
      key: 'sessionid', // 返回个客户端cookie的名字，可自定义
      signed: true, // 进行加盐操作
      maxAge: 60 * 60 * 1000, // 单位ms
    },
    app
  )
)
// 加盐操作
// 设置此操作，会给客户端返回两个由session加密的cookie
// 一个是自己设置的，一个是加盐时候设置的
// 加盐的内容可以自定义，其实就是一种算法
app.keys = ['aaa', 'bbb', 'cccdsa']

userRouter.get('/login', (ctx, next) => {
  // 设置cookie
  ctx.session.slogan = 'ikun'
  ctx.body = '登陆成功'
})

userRouter.get('/list', (ctx, next) => {
  // 获取cookie
  const slogan = ctx.session.slogan
  console.log(slogan)
  ctx.body = 'list'
})
```

## 6.3.token

**cookie和session的缺点**

* Cookie会`被附加在每个HTTP请求中`，所以无形之中增加了流量（有些请求可能不需要Cookie）
* Cookie`是明文传输的`，即使是用了session进行加密，存在安全性的问题
* Cookie的`大小限制是4kb`，对于复杂的请求来说是不够用的
* 对于`浏览器外的其它客户端`（比如IOS、Android），必须`手动的设置cookie和session`**（被替代的主要原因）**
* 对于`分布式系统和服务器集群`中如何可以保证其它系统中也可以正确的解析session**（被替代的主要原因）**
  * 因为session使用加盐的方式加密后，每次生成的session是不唯一的
  * 那么当前系统的session在别的系统中要验证是很麻烦的

**所以，在目前的钱后端分离的开发过程中，使用token来进行身份验证是最多情况**

* token可以翻译成`令牌`
* 这个令牌`作为后续用户访问一些资源的凭证`

**所以token的使用应分为两个重要步骤**

* `生成token`：登陆的时候颁发token
* `验证token`：访问某些资源或接口时，验证token



### 6.3.1.JWT实现Token机制

**JSON（JSON WEB TOKE）**



**JWT生成的Token由三部分组成**



`header和payload可以通过反向解密解出来，但是signature很难被解除来，所以secretKey一定不能暴露`

* **header**

  * alg：采用的加密算法，【默认是HMAC SHA256(HS256)，采用同一个密钥进行加密和解密】
  * type：JWT，固定值，通常写成JWT即可
  * 然后用base64Url算法对header进行编码

* **payload**

  * 携带的数据，比如可以将用户的id和name放在payload中
  * 默认也会携带iat(issued at)，令牌的签发时间
  * 也可以设置过期时间：exp（expiration time）
  * payload的内容也会用base64Url算法进行编码

* **signature**

  * 设置一个secretKey，通过将建两个的结果合并后进行，header.alg的加密方式进行加密

  * eg：HMACSHA256(base64Url(header)+ . + base64Url(payload), secretKey)

  * 但是如果secretKey暴露是一件非常危险的事情，因为之后就可以模拟办法token，也可以解密token

    

### 6.3.2.token的使用

* 使用三方库 jsonwebtoken
  * npm i jsonwebtoken

```javascript
const jwt = require('jsonwebtoken')

const secretKey = 'wssb'

userRouter.get('/login', (ctx, next) => {
  // 颁发token
  const token = jwt.sign({ name: '只能' }, secretKey, {
    expiresIn: 1000, // 单位时s
  })
  ctx.body = {
    code: 0,
    token,
    msg: '登陆成功',
  }
})

userRouter.get('/list', (ctx, next) => {
  // 验证token
  try {
    const authorization = ctx.header.authorization
    const verify = jwt.verify(authorization, secretKey)
    console.log(verify)
    ctx.body = verify
  } catch (error) {
    ctx.body = error
  }
})
```

### 6.3.3.非对称加密(rsa)

* jwttoken `默认算法是SHA256`(对称加密算法)
* 这种算法`只有一个密钥`，所以`验证和颁发token都可以用这个密钥`，是不太安全的
  * 比如在分布式系统中，如果黑客攻破了任一个系统，那么拿到密钥后都可以进行伪造然后颁发token
* 所以可以使用`RS256(非对称加密算法)`
* 这个算法`由一个公钥和私钥组成`
* 可以`用私钥来颁发token，公钥来验证token`
  * 那么在分布式系统中，比如在用户系统中来保存私钥，其它系统都可以拿到公钥，这样，就仅仅需要对用户系统做特殊的保护，而不需要对所有的系统做特殊保护

**如何生成公钥和密钥**

```js
mac中直接在terminal中输入, window需要在gitbash中输入

// 1. 输入 openssl 进入 openssl
opensssl
// 2. 生成密钥  rs256要求最低2048字节
genrsa -out 密钥名称 字节
// eg genrsa -out private.key 2048

// 3.生成公钥
rsa -in 密钥文件 -pubout -out 公钥名
// eg：rsa -in private.key -pubout -out public.key
```



```javascript
const Koa = require('koa')
const KoaRouter = require('@koa/router')
const jwt = require('jsonwebtoken')
const fs = require('fs')

// 创建服务器
const app = new Koa()
// 读取密钥文件
const privatekey = fs.readFileSync('./keys/private.key')
const publickey = fs.readFileSync('./keys/public.key')
// 注册路由
const userRouter = new KoaRouter({ prefix: '/users' })

userRouter.get('/login', (ctx, next) => {
  // 颁发token
  const token = jwt.sign({ name: '只能' }, privatekey, {
    expiresIn: 1000, // 单位时s
    algorithm: 'RS256',
  })
  ctx.body = {
    code: 0,
    token,
    msg: '登陆成功',
  }
})

userRouter.get('/list', (ctx, next) => {
  // 验证token
  try {
    const authorization = ctx.header.authorization
    const verify = jwt.verify(authorization, publickey)
    console.log(verify)
    ctx.body = verify
  } catch (error) {
    ctx.body = error
  }
})
```

# 7.跨域问题

## 7.1.什么是跨域

* 想要理解什么是跨域，**先理解浏览器的同源策略**
  * **同源策略**是一个重要的安全策略，它用于限制一个源资源如何能与另一个源的资源进行交互。它能帮助阻隔恶意文档，减少可能被攻击的媒介。
  * 如果两个URL的`端口(port)、协议(protocol)和主机(host)都相同`的话，则这两个URL是`同源`，可以相互访问资源
  * 这个方案也被称为`‘协议/主机/端口元组’`，或者直接是`’元组‘`
* **事实上跨域的产生和前后端分离的发展有很大的关系**
  * 早起的服务端渲染的时候，是没有跨域的问题的
  * 随着前后端分离，目前前端开发的代码和服务器开发的API接口往往是分离的，甚至部署在不同的服务器上
*  **这个时候我们就会发现，访问 静态资源服务器 和 API接口服务器 很有可能不是同一个服务器或者不是同一个端口。**
  * 浏览器发现静态资源和API接口(XHR、Fetch)请求不是来自同一个地方时(同源策略)，就产生了跨域。

## 7.2.解决方案

**常见方案**

* 方案一：静态资源和API部署在同一个服务器中
* 方案二：`CORS`(跨域资源共享)
* 方案三：`node代理服务器`（webpack中就是它）
* 方案四：`Nginx反向代理`

**不常见方案**

* jsonp
* postMessage
* websocket
* ...

## 7.3.CORS

* CORS(Cross-Origin Resource Sharing 跨域资源共享)
  * 这是一种基于http header的机制
  * 它使用额外的http头告诉浏览器，让运行在一个源的web应用允许访问来自不同源服务器上的制定资源
* 浏览器将CORS请求分成两类：**简单请求和复杂请求**
  * `对于复杂请求`，浏览器必须`首先使用 OPTIONS 方法`发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。
* **满足一下两大条件的，就属于简单请求（不满足的就属于复杂请求）**
  * 请求方法是以下三种方法之一
    * HEAD
    * GET
    * POST
  * HTTP的头信息不超出以下几种字段
    * Accept
    * Accept-Language
    * Content-Language
    * Last-Event-ID
    * Content-Type:只限于三个值` application/x-www-form-urlencoded、multipart/form-data、text/plain`

​	

**代码示例**

* 代码中只需要在响应的时候设置请求头即可
* 开发中可以使用全局中间件，避免每次都要单独设置

```javascript
// cors中间件 解决跨域
app.use((req, res, next) => {
  res.set('Access-Control-Allow-Origin', '*')

  res.set(
    'Access-Control-Allow-Headers',
    'Accept,Accept-Encoding,Accept-Language,ConnectionContent-Length,Content-Type,Host,0rigin, Referer,User-Agent'
  )
  res.set('Access-Control-Allow-Credentials', true)

  // console.log(req.method)
  // if (req.method === 'OPTIONS') {
  //   res.status(204)
  //   res.end('')
  // } else {
  //   next()
  // }
  next()
})

// 编写中间件
app.post('/test', (req, res, next) => {
  res.end('test api')
})

```

## 7.4.node代理服务器

**原理**

* 跨域是因为浏览器的同源安全策略所导致的，但是**服务器与服务器、app与服务器、小程序与服务器是不存在跨域的**
* 所以利用nodejs搭建一个代理服务器，将本地开发的前端代码托管到这个代理服务器的静态资源中
* 将静态资源的请求地址换成代理服务器地址
* 在这个代理服务器中使用`http-proxy-middleware`三方中间件来`转发请求和响应`
  * 当客户端发起请求，实际上是请求这个代理服务器（因为前端资源和代理服务器是在一起的符合同源策略），然后代理服务器拿到请求后，向目标服务器发起请求，目标服务器接送到请求然后响应数据给代理服务器，代理服务器再将结果给到客户端
* **其实webpack中的代理服务器也是用到这个中间件的**，总所周知，webpack是基于node的，我们可以想一下，用webpack来启动文件也是需要启动服务的，然后访问静态资源，原理和我们现在编写代码一样



**安装**

```
npm i http-proxy-middleware
```



### 7.4.1.**使用**



**客户端**

```html
<script>
  // 将请求的地址是代理服务器
    const requst = new Request('http://localhost:8001/api/test', {
      method: 'post',
      headers: {
        'Content-Type': 'application/json',
      },
    })
    fetch(requst).then((res) => {
      console.log(res)
    })
</script>
```



**代理服务器**

​	

​	

* `target`:要代理到的服务器，可以是域名也可以是ip+端口号;
* `changeOrigin`:虚拟托管网站，如果为`true`，源服务器获取的req.header的信息就是他自己的header，如果为`false` 那么获取的header信息就是代理服务器的

* `ws`:是否代理`websockets`;

* `secure`:是否代理https请求;

* `pathRewrite`:重写路径，将":"前的路径替换为后面的路径;

```javascript
const express = require('express')
const httpProxyMiddleware = require('http-proxy-middleware')

const app = express()

const options = {
  target: 'http://localhost:8002',
  changeOrigin: true,
  ws: true,
  secure: true,
  pathRewrite: {
    '^/api': '',
  },
}
// 托管静态资源
app.use(express.static('../client'))

app.use(
  '/api',
  httpProxyMiddleware.createProxyMiddleware(options),
  (req, res, next) => {
    next()
  }
)

// 监听端口
app.listen(8001, () => {
  console.log('8001端口监听成功')
})

```

