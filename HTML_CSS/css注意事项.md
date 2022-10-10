# text-align

``text-align:center``

以下案列的``class="cont"``元素不会居中

因为``w3c``给的定义是：针对于行内级（行内元素、行内块）进行居中，div是块级元素，所以不会居中，只需要让div变成行内块即可居中

```html
<style>
    .box {
        height: 200px;
        text-align : center;
    }
    .cont {
        height: 100px;
        width:100px;
    }
</style>
<body>
    <div class="box">
    	<div class="cont"></div>
    </div>
</body>
```

# text-indent

注意：``text-indent``对``行内非替换元素``无效

# line-height

line-height对行内非替换元素无效，但是会有别的效果，自己实验以下

如果元素里面没有文字，继承/设置 line-height无效，line-height对文字生效





# 行内替换元素

特点：

* 和其他的``行内级``元素在同一行显示

* 可以设置宽度和高度

  

## Example：

​	img、input...



# 行内非替换元素

## 特点

* 不可设置宽高

## 特殊性

* `行内非替换元素`设置上下内边距（``padding``）有效，但是``不占据空间``，（比如给span元素设置了上下内边距，下面有个div元素，此时这个div会跑到span元素的内边距上）
* 设置上下``border``也会撑起来，但是不占据空间



# margin



## 上下margin的传递

### margin-top传递

如果块级元素的``顶部线``和父元素的``顶部线``重叠，那么这个块级元素的``margin-top``值会传递给父元素 （给子元素设置margin-top会作用到父元素上）



### margin-bottom传递

如果块级元素的``底部线``和父元素的``底部线``重叠，并且父元素的``高度是auto``,那么这个块级元素的``margin-bottom``值会传递给父元素 （给子元素设置margin-bottom会作用到父元素上）

### 解决

* 使用padding-top/bottom代替margin-top/bottom
* 给父元素设置border
* 触发BFC
* ……

## 上下margin的折叠



* 垂直方向上的相邻的两个margin（margin-top和margin-bottom）,有可能会合并成一个margin，这种现象叫做``collapse``（折叠）
* 水平方向上的margin（margin-left/right）永远不会collapse
* 两个值进行比较后，取``最大的值``

### 解决

* 两个盒子之间只设置一个margin

### 出现情况

* 两个兄弟块级元素一个设置margin-top，一个设置margin-bottom
* 父子块级元素，父元素和子元素都设置了margin-top

# 元素宽度计算





# background-color/color

* 背景色会设置到border``下面``
* 前景色会在border没有设置颜色的情况下，用color的颜色，如果没有设置color，默认是黑色

# position

``定位元素包含块的宽度 = 定位元素的宽度+left+right+margin-left+margin-right``

所以：当定位元素没有设置宽度（默认auto），也没有设置left、right、margin-left、margin-right时，这个定位元素的宽度就是auto(沾满父元素)

注意：设置元素``position：absolute、fixed``时，元素就``没有``块级和行内及的概念了，此时元素的``宽高``是``包裹元素的内容``

```html
  <style>
      .box {
        position: relative;
        background-color: #faa;
        height: 150px;
        width: 300px;
      }
      .d1 {
        position: absolute;
        left: 0;
        right: 0;
        background-color: #afa;
      }
    </style>
  </head>
  <body>
    <div class="box">
      <div class="d1">ssss</div>
    </div>
  </body>
```

## 让定位的元素水平居中

``固定定位也是绝对定位的一种``

原理：定位参照对象的宽度=绝对定位定位元素的实际占用宽度+left+right+ml+mr

包含块宽度是800px，此时定位元素宽度写死了是100px，left和right是0，ml：auto，mr：auto，所以还剩下700px，给到ml和mr浏览器进行等分，所以实现了水平居中,将以上代码的.d1添加如下代码

```css
.d1 {
	mrgin-left: auto;
    margin-right: auto;
}
```

## 让定位元素垂直居中

原理： 定位参照对象的宽度=绝对定位元素实际占用高度+left+right+mt+mb

# 浮动float

## 特点（规则）

- 元素一旦浮动后，脱离标准文档流
  - 朝着向左或向右方向移动，直到``自己边界紧贴着包含块``(一般是父元素)，或其他``浮动元素``的边界为止
  - ``定位元素``会``层叠``在``浮动元素``上面
- 元素如果是向左（右）浮动，浮动元素的左（右）边界不能超出``包含块``的左（右）边界
- 如果一个元素浮动，另一个浮动元素已经在那个位置了，后浮动的元素将紧挨着前一个浮动的元素（左浮找左浮，右浮找右浮）
- 水平方向剩余的空间不足时候，浮动元素将向下移动，直到又充足空间为止
- 浮动元素不能与``行内级内容层叠``，行内级内容将会被浮动元素推出
  - 比如行内级元素、inline-block元素、块级元素的文字内容
- 行内级元素、inline-block元素浮动后，其顶部将与所在行对齐（就是在当前行左右浮动，不会上下浮动）

# 行内级元素并排有空隙

## 原因

正在开发中，为了方便查看，每个元素都不是写在一行，浏览器解析到回车，会解析成空格，所以就导致了空隙

```html
<style>
    span {
        background: #faa;
    }
</style>
<body>
    <div class="box">
        <span>11</span>
        <span>22</span>
        <span>33</span>
    </div>
</body>
```

## 解决方案

1. 不换行，所有的元素都写在一行（不推荐）
2. 设置父元素的``font-size``为0，应为空格的大小是文字的大小控制的，所以给父元素设置字体大小0，子元素就继承了，就没有空格了，缺点是要为每个子元素单独设置字体大小
3. 给每个子元素设置统一方向浮动``float``,浮动有个特点就是会紧挨着浮动的元素
4. 使用``flex``布局



# 高度坍塌

## 解决方案

```css
.clear-fix::after {
    content: "",
    display:block;
    clear: both;
    /*
    一下代码是为了兼容低版本的浏览器
    */
    visibility: hidden; /*兼容浏览器*/
    height: 0; /*兼容浏览器*/
    *zoom: 1; /*兼容IE6/7*/
}
```

# flex布局

## 特点 

- 开启了flex布局的元素叫``flex container``（弹性容器）
- flex container里面的直接子元素叫做``flex item``（弹性项目）

### 当flex container中的子元素变成了flex item时，具有以下特点

- flex item的布局将受flex container属性的设置来进行控制和布局
- ``flex item`` 不再严格区分块级元素和行内元素
- flex item默认情况下时包裹内容的，但是可以设置宽度和高度

### 设置display属性为flex或者inline-flex可以成为flex container

- flex： flex container以``block-level``形式存在
- inline-flex： flex container以i``nline-level``形式存在 

## flex item的宽度



## flex container的属性

### flex-direction

设置主轴的方向：

- ``row(默认)``：主轴从左往右
- row-reverse：主轴从右往左
- column：主轴从向至下
- column：主轴从下至上  

### flex-wrap

设置当项目在一行排不下时候是否换行（flex container是单行显示还是多行）

* ``nowrap（默认）``：单行（不换行）
* wrap：多行（换行）
* wrap-reverse：多行（换行，顺序是由下到上）

### flex-flow

flex-direction和flex-wrap的简写，``顺序任意，每个值不是必写``

flex-flow:<'flex-direction'> || <'flex-wrap'>

### justify-content

设置项目在主轴（main axis）的对齐方式

* ``felx-start(默认值)``：与main start对齐
* flex-end：与main end对齐
* center：居中对齐
* space-between：
  * flex items之间的距离相等
  * 与main start、main end两端对齐
* space-around：
  * flex items之间的距离相等
  * flex items与main start、main end之间的距离是flex items（相邻的两个item之间的距离）之间距离的一半
* space-evenly：兼容性差
  * 两端也有空隙，并且所有的空间进行等分

### align-items

 设置（单行）flex item(项目)在cross axis（交叉轴）的对齐方式

* ``normal（默认）``：在弹性布局中，效果和stretch一样
* stretch：当flex item 在cross axis 方向的size为 auto时，会自动拉伸至填充flex container(``项目的高度是auto时才生效``)
* flex-start：与cross start对齐
* flex-end：与cross end对齐
* center：居中对齐
* baseline：与基准线对齐

### align-content

设置多行flex items在cross axis上的对齐方式，用法与justify-content类似

* stretch(默认值)：与align-items的stretch类似
* flex-start:
* flex-end:
* center:
* space-between
* space-around
* space-evenly：兼容性差

## flex-item的属性

### flex-order

设置flex item的排布顺序

* 可以设置``任意整数``，值越小就越排在前面
* 默认值是0

### align-self

设置flex item在across axis的位置

* 默认值是normal

### flex-grow

设置flex items如何扩展（拉伸/成长）

``说明``：当一个容器里面的项目排列完成，还剩下空间的时候，这些剩下的空间是不会分给项目，让项目填充的，如果设置了flex-grow:1，那么剩余的空间将都会分给设置这个属性的项目，如果一个项目设置了flex：1，一个设置了flex：2，那么剩余的空间将会被等分成3分，设置1的得到1份，设置2的得到2份

* 可以设置``任意非负的数字``（正小数、正整数、0）
* 默认值是``0``
* 当flex container在main axis方向有``剩余的size``时，flex-grow属性``才会生效``
* 如果所有flex items的flex-grow``总和sum超过1``，每个flex item扩展的四则为
  * flex container的剩余size*flex-grow/sum
*  flex items扩展后的最终size不能超过max-width/max-height
  * 比如给一个项目设置了max-width\max-height,那么项目所分配的空间和项目的宽/高之和最多就是max-width\height

### flex-shrink

决定了flex items如何收缩（缩小）

当项目在容器内水平排不下的时候，项目是否会被压缩

* 可以设置``任意非负数字``
* 默认值是``1``
* 当flex  items在main axis方向上``超过flex container的size，flex-shrink属性才会生效``
* 如果所有flex items的flex-shrink总和超过1，每个flex item收缩的size为
  * flex items超出flex container的size*收缩比例/所有flex items的收缩比例之和
*  flex items收缩后的最终size不能小于min-width/min-height



### flex-basis

用来设置flex items 在main axis方向上的basis size（基础大小）

``默认值是auto``

``举例``： 当给一个项目设置了width，里面的内容是一个英文单词的时候，当单词在最后一个的时候放不下整个单词会直接换行，如果设置了flex-basis时候，则不会换行，会自动拉伸当前项目的宽度，知道单词能放下

觉得flex-item最终的basis size的因素（优先级从上到下）

* max-width/height、min-width/height
* flex-basis
* width\height
* 内容本身的size

### flex

是felx-grow | |  flex-shrink | | flex-basis的简写，flex属性可以指定1、2、3个值 

* 值只有一个无单位的数字时，会被当作flex-grow
* 值是一个有效的宽度的时候，会被当作flex-basis
* 关键字 none 、auto、initial

# box-sizing

  box-sizing: 在生效时, 有一个前提, ``有明确的设置宽度和高度`` 



## box-sizing的无作用

当一个元素没有设置宽度或者高度的时候（宽度高度默认是auto），设置box-sizing:border-box是无效的(定位、flex)

# white-space

超屏：超出屏幕，或者时超出包裹元素

* normal(默认)：```合并``所有连续的空白，```允```许单词超出屏幕时自动换行
* nowrap：```合并```所有连续的空白，```不允许```单词超屏自动换行
* pre：```阻止合并```所有连续的空白，```不允许```单词超屏时自动换行
* ......

# FC

fc:Formatting Context 格式化上下文



# BFC

## 解决（float导致的）高度坍塌

### 满足条件

* 浮动元素的父元素触发BFC，形成独立的块级格式上下文
* 浮动元素的父元素的高度时auto

BFC的高度时auto的情况下，是如下计算高度的

* 如果只有inline-level,是行高的底部和顶部的距离
* 如果有block-level，是由最顶层的块上边缘和最底层块盒子的下边缘之间的距离
* 如果有绝对定位元素，将被忽略
* 如果有浮动元素，那么会增加高度以包含这些浮动元素的下边缘

# 媒体查询

## 媒体类型

* all（默认值）：适用于所有设备

* print：适用于在打印预览模式下在屏幕上查看分页材料和文档

* screen：主要用于屏幕

* speech：主要用于语音合成器

* ```css
  <style>
  @media screen and (min-width:500px) {
      ...
  }
  </style>
  ```

## 媒体特性

* 通常会将媒体特性描述为一个表达式

* 每条媒体特性表达式都必须用括号括起来

* ```css
  @media（max-width:800px）and (min-width:400px) {
      ...
  }
  ```

* 

## 媒体查询配合@import一起使用

最小宽度是600时候导入a.css

最小宽度是800时候导入b.css

```css
<style>
	@import("./a.css") (min-width: 600px);
	@import("./b.css") (min-width: 800px);
</style>
```

## link配合媒体查询使用

```html
<link rel="stylesheet" href="./a.css" media="(min-width:300px)" />
```

## 媒体查询单独使用

```html
<style>
</style>
```

# 单位

## 相对单位

### em

* 相对于自己```font-zize```的大小
* 如果自己没有font-size，会继承包裹元素的font-size，所以还是相对于自己的font-size的大小
* 如果```font-size```中有写```em```单位，可以理解成相对父元素，但是更准确的理解依然是相对与自己的
* mdn: 在font-size中使用是相对于父元素的字体大小，在其他属性中使用是相对于自己的字体大小

### rem

* 相对于html元素的字体大小 

### vw / vh

vw : viewport width

​	视口宽度的1%

vh: vierport height

​	视口高度的1%

## px（pixel）

* 物理像素（设备像素）
  * 屏幕出场时候的像素无法通过程序修改
* 逻辑像素（设备独立像素）
  * 操作系统为了平衡各个设备的物理像素，抽象出来的像素
* css像素
  * 就是逻辑像素，当我们设置好逻辑像素后，操作系统会根据设备的物理像素来转换逻辑像素

# DRP、PPI

## DPR

* 2010年，iPhone4的出现，出现了Retina屏幕
* Retina屏幕被翻译成```视网膜屏幕```
* ```一个```逻辑像素在长度上对应```两个```物理像素,这个比例被称为```设备像素比```
* 可通过window.devicePixelRatio获取当前屏幕的DPR值

## PPI

* 每英寸可以放多少个物理像素
* 1in（英寸） = 2.54cm ≈96px(逻辑像素) 

### 