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

