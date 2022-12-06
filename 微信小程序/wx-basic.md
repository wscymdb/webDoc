# 小程序

# 1.小程序的双线程模型

## 1.1.小程序的架构模型 

* **当小程序基于WebView环境下时，WebView的JS逻辑、DOM树创建、CSS解析、样式计算、Layout、Paint都发生在同一线程，在WebView上执行过多的js逻辑可能阻塞渲染，导致界面卡顿**
* **为此，小程序同时考虑了性能与安全，采用了目前称为``【双线程模型】``的架构**
* **双线程模型**
  * ``WXML模块和WXSS样式``运行于``渲染层``，渲染层``使用WeBView线程渲染``（一个程序有多个页面，会使用多个WebView的线程）
  * ``JS脚本``运行于``逻辑层``，逻辑层``使用JsCore``运行JS脚本
  * 这两个线程都会经由``微信客户端(Native)``进行中转交互

# 2.小程序的配置文件

## 2.1.常见的配置文件

* `project.config.json`
  * 项目配置文件，比如项目名称、appId等
* `sitemap.json`
  * 小程序搜索相关的
* `app.json`
  * 全局配置
  * https://developers.weixin.qq.com/miniprogram/dev/reference/configuration/app.html
* `page.json`
  * 页面配置 

# 3.注册小程序

## 3.1.App函数

文档地址：https://developers.weixin.qq.com/miniprogram/dev/reference/api/App.html

### 

* 判断小程序的进入场景
  * 可以通过好多种方式打开小程序，搜索打开、聊天页面打开等
* 监听生命周期函数
* ``因为App实例只有一个，并且是全局共享的``，所以可以``将一些共享的数据放在这里``

### 3.1.1.注意事项

* app函数选项中可以放自己的数据，用于全局共享，但是数据`不是响应式的`

# 4.wxs

## 4.1.为什么设计WXS

* 在WXML中是`不能直接调用Page/Component中定义的函数的`
  * 因为小程序底层的架构是双线程模型，那么如果WXML想用js中的方法需要通过微信客户端进行交互，这操作太过繁琐，势必会浪费性能
    * 比如WXML页面中要调用一a方法，首先通过通过微信客户端通知js线程去找到a方法
    * 然后调用a方法
    * a方法调用完毕后通过微信客户端把处理的结果给到WXML
* 但是在某些情况，又希望像vue那样可以`在html中调用js中的方法`，所以就设计了WXS
  * `WXS其实是和WXML在一个线程中的`
  * 所以`不应该在WXS中做太过耗时的操作`，否则很可能会阻塞页面渲染，这样就违背了双线程模型的初衷了

## 4.2.WXS使用的限制和特点

* WXS不依赖于运行时的基础库版本，可以在所有版本的小程序中运行
* WXS的运行环境和其他JavaScript代码是隔离的，WXS中`不能调用其他JavaScript文件中定义的函数`，也`不能调用小程序提供的API`
* 由于运行环境的差异，在IOS设备上小程序内的WXS会比Javascript代码快2~20倍，在Android设备上，二者运行效率无差异

#  5.组件

## 5.1.自定义组件

### 5.1.1.细节

* `自定义组件也可以引用别的自定义组件`，引用方式类似于页面引用自定义组件的方式(使用usingComponents字段)
* 自定义组件和所在项目根目录不能以`wx-为前缀`，否则会报错
* 如果在`app.json`的usingComponents`声明某个组件`，那么`所有页面和组件可以直接使用该组件`

### 5.1.2.组件样式的细节

* 组件内的样式对外部样式的影响
  * 组件内的class样式，只对组件的WXML节点生效，对引用组件的page不生效
  * 组件内不能使用id、属性、标签选择器
* 外部样式对内部样式的影响
  * 外部的class只对外部的WXML节点生效，对组件内不生效
  * 外部使用id、属性选择器不会对组件内产生影响
  * 外部使用标签选择器会对组件内产生影响
* 如何让class互相影响
  * 在`Component对象`中(组件的js文件)，可以传入一个`options属性`，options中有一个`styleIsolation属性`
  * styleIsolation属性的值（部分）
    * isolated 表示启用样式隔离
    * apply-shared 表示页面会影响组件，组件不会影响页面
    * shared  组件和页面相互影响

## 5.2.组件之间的通信

### 5.2.1.Properties

* **用来接收页面传来的数据**

* **支持的类型**
  * Number、String、Boolean
  * Object、Array、null（不限制类型）

* **可以使用value属性来设置默认值**

a.wxml(页面)

```html
<info-demo title="我与地坛"></info-demo>
```

info-demo.js(组件)

```javascript
Components({
    properties: {
        title: {
            type: String,
            value: '我是默认值'
        }
    }
})
```

### 5.2.2.externalClasses

**用来接收页面传来的样式，类似vue的no-**pros

b.wxml（组件）

```html
<view class="external-info"></view>
```

b.js

```javascript
Components({
    externalClasses: ['external-info']
})
```



a.wxml(页面)

```html
<b external-info="info" ></b>
```

a.wxss

```css
.info {
    background: green;
}
```

### 5.2.3.methods

* **用来存放组件的方法**
* 使用``trigger-event``来发射自定义事件
* **自定义事件传递的值是放在``event.detail``里面的**

b.wxml(组件)

```html
<view bindtab="handleClick">我是组件</view>
```

b.js

```javascript
Components({
    methods: {
        handleClick() {
            this.triggerEvent('demoClick', 'aaa')
        }
    }
})
```



a.wxml（页面）

```html
<b bind:demoClick="handleClick"></b>
```

a.js

```javascript
Pages({
    handleClick(e) {
        console.log('组件发生点击',e.detail)
    }
})
```

## 5.3.获取组件实例

* 使用`selectComponent`方法可以获取当前组件实例
* **类似vue的ref**

a.wxml

```html
<info class="info-demo"></info>
```

a.js

```javascript
Pages({
    demo() {
        console.log(this.selectComponent('.info-demo'))
    }
})
```



## 5.4.插槽

### 5.4.1.单一插槽

* 写法类似vue

b.wxml(组件)

```html
<view class="my-slot">
    <view class="header">Header</view>
    <view class="content">
        <slot></slot>
    </view>
</view>
```

a.wxml(页面)

```html
<b>
你好陈哈哈
</b>
```



### 5.4.2.解决插槽默认值

* 微信小程序的插槽是``没有默认值``的

**解决方案**

* 其中的一个解决方案
* 利用css来解决的
* 首先在.content平级别的写一个默认值
* 然后利用空伪类和兄弟选择器来解决

.wxml

```html
<view class="my-slot">
<view class="header">Header</view>
<view class="content">
  <slot></slot>
</view>
  <view class="default">你好李焕英</view>
<view class="footer">Footer</view>
</view>
```

.wxss

```css
.default {
  display: none;
}
.content:empty + .default {
  display: block;
}
```

### 5.4.3.多个插槽

* 如果需要用多个插槽，需要将组件的js文件的``options``的``multipleSlots``设置为``true``
* 页面中在元素中使用slot=”插槽名“

b.wxml(组件)

```html
<view class="mul-slot">
    <view class="left">
        <slot name="left"></slot>
    </view>
    <view class="center">
        <slot name="center"></slot>
    </view>
    <view class="right">
        <slot name="right"></slot>
    </view>
</view>
```

b.js  

```javascript
Components({
    options: {
        multipleSlots: true
    }
})
```



a.wxml(页面)

```html
<b>
    <button size="mini" slot="left">left</button>
    <view slot="center">我是帅哥</view>
    <button size="mini" slot="right">right</button>
</b>
```

## 5.5.behavior

* 类似vue的mixin
* 将公共的代码抽离
* 引入behavior的页面会将behavior中的内容合并

a.js

```javascript
export const counterBehaviors = Behavior({
    data: {},
    methods: {},
    ...
})
```



b.js(引入behavior)

```javascript
import {counterBehaviors} form '/behaviors/counter'

Components({
    behaviors: [counterBehaviors]
})
```

## 5.6.组件的声明周期

* 自小程序基础库2.2.3本本起，组件的生命周期也可以写在`lifetimes字段`中了

**最重要的三个生命周期**

* created
  * 组件被创建阶段，不可以访问dom元素
* attached
  * 组件被放到dom树中，可以访问dom元素
* detached
  * 组件被移除阶段



### 5.7.监听组件所在页面的生命周期

* 监听当前组件所在页面的生命周期函数
* 在`pageLifetimes`字段中进行监听

**可监听的函数有**

* show
  * 组件被显示
* hide
  * 组件被隐藏
* resize
  * 组件所在页面尺寸发生变化

# 6.常见的API

## 6.1.网络请求API

**wx.requset**

文档地址：https://developers.weixin.qq.com/miniprogram/dev/api/network/request/wx.request.html

## 6.2.小程序的提示框

**按名字去文档搜索**

* showToast
* showModal
* showLoading
* showActionSheet

## 6.3.分享功能

**转发给朋友**

使用页面的Page函数内部使用``onShareAppMessage``方法开启分享功能

可选参数

* title
  * 分享的标题
* path
  * 打开分享页面时，进入哪个小程序
* imageUrl
  * 分享时候图片的路径

```javascript
Page({
    onShareAppMessage() {
        return {
            title: '',
            path: '',
            imageUrl: ''
        }
    }
})
```

## 6.4.获取设备信息

### 获取设备基本信息

使用``wx.getSystemInfo()``

```javascript
onLoad() {
    wx.getSystemInfo({
        success: (result) => {
            console.log(result);
        },
    })
}
```

### 获取当前位置信息

使用``wx.getLocation``

需要在app.json中开启权限

app.json

```json
{
    "permission": {
        "scope.userLocation" : {
            "desc": "获取你的位置信息，做坏事"
        }
    }
}
```

b.js

```javascript
onLoad() {
    wx.getLocation({
        success(res) {
            console.log(res);
        }
    })
}
```



## 6.5.Storage存储

* 小程序提供用于本地存储的api
* 分为同步存储和异步存储
* 使用``同步``的话后续代码``会等待``当前代码执行完毕后才会执行

**同步存储**

* wx.setStorageSync(string key, any data)
* wx.getStorageSync(key) 
* wx.removeStorageSync(key)
* wx.clearStorageSync()

**异步存储**

- wx.setStorage(string key, any data)
- wx.getStorage(key)
- wx.removeStorage(key)
- wx.clearStorage()

## 6.6.页面跳转

文档地址：https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.switchTab.html

**编程式跳转**

* wx.switchTab
  * 跳转到tabBar页面，并关闭其他所有非tabBar页面
* wx.reLaunch
  * 关闭所有页面，打开到应用内的某个页面
* wx.redirectTo
  * 关闭当前页面，跳转到应用内的某个页面
* wx.navigateTo
  * 保留当前页面，跳转到应用内的某个页面
* wx.navigateBack
  * 关闭当前页面，返回上一页或多级页面

**导航式跳转**

类似vue的router-link

```html
<navigator url="" open-type=""></navigator>
```

### 6.6.1.跳转页面如何携带参数

* 通过query的方式携带

```javascript
goSkip() {
    wx.navigateTo({
        url: '/pages/skip/index?name=zs&age=18',
    })
}
```



### 6.6.2.返回导航如何携带参数

**方案一**

* ``getCurrentPages``可以获取打开的页面的页面栈中所有的页面
* 获取上一级页面后直接修改上一级页面的data的值
* 如果点击小程序提供的返回按钮该如何传值呢
  * 可以在`onUnload`生命周期里面写一下逻辑
  * 返回的时候就会卸载当前的页面

```javascript
goBack() {
    const pages = getCurrentPages()
    const perviousPage = pages[pages.length - 2]
    perviousPage.setData({
        message: '我来自skip页面'
    })
    wx.navigateBack()

}
```

**方案二**

使用`navigateTo的events`，类似eventBus

```javascript
goSkip() {
    wx.navigateTo({
        url: '/pages/skip/index?name=zs&age=18',
        events: {
            // 事件名自定义，可以写多个
            goBack(payload) {
                console.log('c',payload);
            }
        }
    })
}

goBack() {
    const eventChannel = this.getOpenerEventChannel()
    eventChannel.emit('goBack', {name: 'zs'})
    console.log(eventChannel);
    wx.navigateBack()

}
```

## 6.7.登录

 。。。