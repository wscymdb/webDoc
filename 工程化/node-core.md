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

// end事件
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

