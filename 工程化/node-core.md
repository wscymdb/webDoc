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



