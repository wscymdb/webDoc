# 1.坐标系

* **CSS3 transform属性允许再二维或三维空间中直观地变换元素**
  * 用transform属性变换的元素会受`transform-origin`属性值的影响，该属性用于指定`形变的原点`
* **元素的坐标**
  * CSS中每一个元素都有一个坐标系，其原点位于元素的左上角，`左上角被称为初始坐标`
  * `使用transform时`，坐标系原点`默认会移动到元素的中心`
    * 因为transform-origin属性的默认值为50% 50%，即该原点将作为变换元素的中心点
  * 使用transform属性`旋转`或`倾斜`元素，`会变换或倾斜元素的坐标系`，并且该元素`后续`的`所有变换`都将`基于新的坐标系`进行变换

# 2.3D动画

## 2.1.常见的函数transform function：

* 平移：translate3d(x, y, z)
* 缩放：scale3d(x, y, z)
* 旋转：rotate3d(x, y, z, a)

**3D形变函数会创建一个合成层来启用GPU硬件加速，例如：translate3D、translateZ、scale3d、rotate3d等**

## 2.2.3D透视 perspective

* 透视
  * 定义了`观察者与z=0平面的距离`，使得具有三维位置变换的元素产生透视效果
  * z>0的三维元素比正常的大，z<0时则比正常的小
  * 自我理解：perspective就好比人的眼睛，通过赋值来决定眼睛与屏幕的距离
* 取值
  * 必须是`长度`或者`none`
* 使用
  1. 在父元素上定义CSS透视属性
  2. 在元素本身定义，使用perspective函数

```css
/*
1.在父元素上定义
*/
.fat {
    width: 200px;
    height: 200px;
    background-color: #faf;
    margin: 10% auto;
    perspective: 50px;
}
.son {
    width: 70px;
    height: 70px;
    background-color: #aff;
    margin: auto;
    transform: rotateY(60deg);
}

/*
2.在元素自身定义
*/
.son {
    width: 70px;
    height: 70px;
    background-color: #aff;
    margin: auto;
    transform: perspective(50px) rotateY(60deg);
}
```



## 2.3.3D空间 transform-style

* **变换式**：transform-style
  * 该CSS属性用于`设置元素的子元素`是`定位在3D空间中`还是`平展在元素的2D平面中`
  * 言外之意此css属性是定义在父元素之中的
* **值类型**
  * flat：指示元素的子元素位于元素本身的平面内`(2D)`
  * preserve-3d：指示元素的子元素`位于3D空间`

## 2.4.3D背面可见性-backface-visibility

* 定义
  * 指定某个元素当背面朝向观察者时是否可见
* 值
  * visible：背面朝向用户时可见
  * hidden：背面朝向用户时不可见

# 3.浏览器渲染过程

* 1、解析HTML，`构建DOM Tree`
* 2、对CSS文件进行解析，解析出对应的规则数
* 3、DOM Tree + CSSOM生成`Render Tree`
* 4、布局（Layout）：计算出每个节点的宽度、高度和位置信息
  * 页面元素位置、大小`发生变化`，往往会导致其他节点联动，需要重新计算布局，这个过程称为`回流(Reflow)`
* 绘制(Paint)：将可见的元素绘制在屏幕中
  * `默认标准流`是在`同一层上绘制`，一些`特殊属性`会`创建新的层绘制`，这些称为`渲染层`
  * 一些不影响布局的CSS修改也会导致该渲染层重绘(Repaint)，回流必然会导致重绘

# 3.CSS3动画性能优化

* **创建一个新的渲染层(减少回流)**
  * `有明确的定位属性(relative、fixed、sticky、absolute)`
  * `透明度(opacity小于1)`
  * `有CSS transform属性(不为none)`
  * 当前对于`opacity、transform、filter、backdrop-filter`应用动画
    * filter、backdrop-filter，慎用，会消耗大量内存，尽量少用
  * backface-visibility属性为hidden
  * ...
* **创建合成层**
  * **`合成层会开启GPU加速页面渲染，但是不能滥用`**
  * `对opacity、transform、filter、backdropfilter应用了animation或transition`
    * 开启的动画或过度中有以上属性就会开启GPU加速
    * 但是这些动画或过度都是`执行中的才会开启`，执行完毕后就不开启了
  * `will-change`设置为opacity、transform、top、left、bottom、right
    * 比如给一个元素添加属性：will-change:opacity,transform
    * 就会对当前元素创建一个合成层
    * 如果设置left\top\right\bottom，则当前元素必须要有定位
* **少用高斯模糊和渐变**
  * 很消耗内存，导致页面卡顿
* **控制动画时间**
  * a页面跳转至b页面，a页面的动画还在执行
* **换种方式**
  * 以图片或视频的方式代替动画

# 4.Canvans

## 4.1.定义

* **Canvas提供了非常多的JavaScript绘图API，结合\<canvas>元素可以绘制各种2D图像**
  * 比如绘制路径、矩形、圆、文本和图像等方法
  * Canvas API`主要用于绘制2D图形`，当然也可以使用Canvas提供的WebGl API来绘制3D图形

## 4.2.应用场景

​	**可以用以动画、游戏画面、数据可视化、图片编辑以及实时处理视频等方面**

## 4.3.canvas优缺点

* **优点**
  * Canvas提供的功能更加原始，`适合像素级处理、动态渲染和量大数据的绘制`
    * 如：图片编辑、热力图、炫光尾迹特效等
  * Canvas非常适合图像密集型的`游戏开发`，适合频繁重绘大量的图形对象
  * Canvas能够已.png或.jpg格式保存对象，`适合对图片进行像素级处理`
* **缺点**
  * 在移动端如果Canvas使用数量过多，会使内存占用超出了手机承受范围，可能会导致浏览器崩溃
  * 放大会失帧，因为Canvas由像素绘制

## 4.4.Canvas使用注意事项

* \<canvas>和\<img>元素很像，不同就是它`没有src和alt属性`
* \<canvas>标签`只有两个属性`--width和height
  * 默认单位px
  * 没有宽高时，canvas回初始化宽300px高15px
* \<canvas>必须是双标签
* 可以通过`canvas.getContext方法是否存在`来监测浏览器是否支持Canvas

## 4.5.Canvas Grid(坐标空间)

**Canvas Grid可以理解为网格或坐标空间**

* \<canvas>元素默认`被网格所覆盖`
* 通常来说网格中的`一个单元相当于canvas元素中的一个像素`
* 网格`原点坐标位于画布左上角`，所有图形都相对于该原点绘制

## 4.6.认识路径

**定义**

* `路径是点列表，由线段连接`。这些线段可以具有不同的形状、弯曲、不弯曲、连续、不连续、不同宽度和不同颜色
* `路径`可由`多个子路`径构成，这些子路径都是在一个列表中，列表中所`有子路径(线、弧)将构成图形`
* 绘制一个路径，甚至一个子路径，通常都是闭合的（会调用closePath来闭合）



**使用路径绘图的步骤**

1. 首先需要创建路径起始点（beginPath）
2. 然后使用绘图命令去画出路径（arc、lineTo）
3. 之后把路径闭合（closePath，非必须，建议写）
4. 一旦路径生成，就能通过`stroke(描边)`或`fill(填充)`来渲染图形



**绘制路径时，所用到的函数**

* beginPath
  * 新建一条路径，生成之后，图形绘制命令被指向到新的路径上绘图，`不会关联到旧的路径`
* closePath
  * 闭合路径，闭合之后图形绘制命令又重新指向到beginPath之前的上下文中
  * 如果使用stoke填充，会将画笔的结束位置和画笔最开始的位置连接
  * 如果使用fill填充，路径会自动闭合，使用stroke不会
* stroke
  * 通过线条来绘制图形描边（针对当前路径图形）
* fill
  * 通过填充路径的内容区域生成实心的图形（针对当前的路径图形）
  * 如果绘制的是`单条线`fill填充没用，`要用stroke`



**常用指令**

* **moveTo(x, y)：移动画笔**
* **lineTo(x, y)：绘制一个条从当前位置指定(x, y)位置的直线**
  * (x, y)表示坐标系中直线结束的点
  * 开始点和之前绘制的路径有关，`之前路径结束点就是接下来的开始点`
  * 比如开始移动画笔到(0,1),然后lineTo(30, 30),再次lineTo(),那么(30, 30)就是开始点
  * 当然开始点也可以通过moveTo()函数来改变
* 

## 4.7.绘制矩形(Rectangle)

**Canvas支持两种方式来绘制矩形：`矩形方法`和`路径方法`**

* **路径方法**
  * rect(x, y, w, h)
    * 绘制一个左上角坐标为(x, y), 宽高为w和h的矩形
    * 当该方法执行时，`首先moveTo(x, y)方法自动设置坐标参数(0,0),然后才执行rect的x,y`
* **矩形方法**
  * fillRect(x, y, width, height)：绘制一个填充的矩形
  * strokeRect(x, y, width, height)：绘制一个矩形的边框
  * clearRect(x, y, width, height)：清除指定矩形区域，让清除部分完全透明



## 4.8.绘制圆弧(Arc)、圆(Circle)

**弧度**

* 弧度(radian)，是平面角的单位。
* `1单位弧度为：圆弧长度等于半径时的圆心角`
* 一个`完整的圆`的`弧度是2Π`，`半圆弧度是Π`

**角度和弧度之间的转换**

* `弧度 = 角度*（Π/180）`
  * 旋转90°：Math.PI/2
  * 旋转180°：Math.PI
  * 旋转360°：2*Math.PI
  * 旋转-90°：-Math.PI/2

**绘制圆弧或圆的方法**

**arc(x, y, radius, startAngle, endAngle, anticlockwise)**

* x, y 表示圆心坐标
* radius：圆弧半径
* startAngle, endAngle：指定开始和结束的`弧度`，以x轴为基准
* anticlockwise：是一个布尔值，true表示逆时针方向，false是顺时针方向，`默认是false`

##   4.9.色彩

**图形上色分为描边上色和填充上色**

* 填充上色

  * `fillStyle = color`
  * `需要在fill()函数调用执行调用`

* 描边上色

  * strokeStyle = color
  * `需要在stroke()函数调用执行调用`

* ```javascript
  let canvasEl = document.querySelector("#tutorial");
  let ctx = canvasEl.getContext("2d");
  
  ctx.beginPath();
  ctx.rect(100, 100, 100, 100);
  ctx.fillStyle = "#f00";
  ctx.fill();
  ```

* 

**color颜色取值**

* color可以是表示CSS颜色值的字符串，也可以是关键字、十六进制、rgb、rgba格式
* `默认情况下color颜色值是黑色`

**注意**

* `一旦设置`了strokeStyle或fillStyle的值，那么这个`新值`就会`成为`下一个`新绘制图形颜色`的`默认值`

## 4.10.透明度

* 方法一

  * 使用strokeStyle和fillStyle属性结合RGBA

  * ```javascript
    ctx.fillStyle = "rgba(255, 0, 0, 0.5)";
    ```

  * `只针对当前绘制的图形生效`

* 方法二：globalAlpha

  * globalAlpha = 0 ~ 1

  * `此属性会影响到canvas里所有图形的透明度`

  * ```javascript
    let canvasEl = document.querySelector("#tutorial");
    let ctx = canvasEl.getContext("2d");
    ctx.globalAlpha = 0.9;
    // 进行绘制
    ```

  * 

## 4.11.线形 Line styles

**设置线条的形状（宽、高、末端样式...）**



**lineWidth**

* 用来设`置线条宽度的属性`，`值必须是正数`。默认值是1px，不需要单位（0，负数、infinity和NaN都会被忽略）
* `线宽是值给的路径的中心到两边的粗细`。换句话说就这在路径的两边各一半绘制。
* **假如要绘制一条从(3, 1)到(3, 5)，宽是1px的线条，其实会占用两个像素**
  * 当前路径的左边填充一半，右边填充一半，加起来正好1px
  * 左右剩下的半个像素，`会以当前颜色的一半来填充`,就会出现一个虚边的效果
  * 所以可以总结出如果宽度是偶数就不会出现上述问题，如果是奇数就会有这个问题
  * 如果是宽度奇数，但是又想准确的显示怎么办？
    * 可以将(3, 1)到(3, 5)变成(3.5, 1)到(3.5, 5)
    * 这样填充的就刚好是1px了

**lineCap**

* 该属性决定了**`线段端点显示的样子`**
* 取值
  * butt：截断（默认值）
  * round：圆形
  * square：正方形

**lineJoin**

* 该属性决定了**`图形线段连接处所显示的样子`**
* 取值
  * round：原形
  * bevel：斜角，（在连接处斜切一刀，类似于平角的效果）
  * miter：斜槽规(默认值)（效果是尖角）

## 4.12.绘制文本

**Canvas提供了两种方法来绘制文本**

* fillText(text, x, y [,maxWidth])
  * 在(x,y)位置填充文本text
  * 绘制的最大宽度(可选)
* strokeText(text, x, y [,maxWidth])
  * 在(x,y)位置绘制文本text
  * 绘制的最大宽度(可选)

**文本的样式(需要在绘制文本之前调用)**

* font = value 
  * value取值和CSS font属性相同的语法(是font的简写属性，必须有font-size 和 font-family)
  * 默认值是：10px sans-serif
* textAlign = value
  * value取值有：start、end、left、right、center
  * left指的是字体最左边跟当前形状的起始坐标对齐
  * right指的是字体最右边跟当前形状的起始坐标对齐
  * 默认值是start
* textBaseline = value
  * 表示基线对其方式
  * value取值有：`top`、hanging、`middle`、alphabetic、ideographic、`bottom`
  * 默认值是 alphabetic

## 4.13.绘制图片

**绘制图片，可以使用drawImage方法将图片渲染到canvas里面，drawImage方法有三种形态**

* drawImage(image, x, y)
  * iamge：表示image对象或canvas对象
  * x, y表示其在目标canvas中的起始坐标
* drawImage(image, x, y, width, height)
  * width和height是用来控制image的缩放效果(宽高)
* drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)
  * 用来裁剪图片的
  * mdn:https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/drawImage

**Image的来源**

* `HTMLImageElement`
  * 可以是由Image构造函数构造出来的
  * 也可以是\<img>元素，通过document.getElementById()等选择器获取的
* HTMLVideoElemen
  * 用一个HTML的\<video>元素作为图片源，可以从视频中抓取一帧作为图像
* HTMLCanvasElement
  * 可以使用另一个\<canvas>元素作为图片源

**注意**

* 如果绘制的图形出现了重叠，按照后面绘制的覆盖前面绘制的原则
* 比如先绘制了图片，在绘制了图形，两个元素在同一个位置，那么图形会覆盖图片

**案列**

`绘制一个图片`

```html
<canvas id="tutorial" width="500" height="500"></canvas>

<script>
    window.onload = () => {
        let canvasEl = document.querySelector("#tutorial");
        /** @type {CanvasRenderingContext2D} */
        let ctx = canvasEl.getContext("2d");
        let image = new Image();
        image.src = "url";

        image.onload = () => {
            ctx.drawImage(image, 0, 0);

            ctx.beginPath();
            ctx.moveTo(20, 80);
            ctx.lineTo(40, 90);
            ctx.lineTo(60, 30);
            ctx.lineTo(80, 80);
            ctx.lineTo(100, 200);
            ctx.lineTo(120, 20);
            ctx.stroke();
        };
    };
</script>
```

## 4.14.Canvas绘画状态-保存和恢复

**Canvas绘画状态**

* 是当前绘画时所产生的`样式和形变的一个快照`
* Canvas在绘画时，会产生相应的绘画状态，可以`将某些绘画状态存储在栈中后续进行复用`

**保存和恢复绘画状态**

* save()：保存画布的所有绘画状态
  * 每调用一次就保存当前画布的所有绘画状态
* restore：恢复画布的所有绘画状态
  * 因为存储结构是`栈`，符合`先进后出`的顺序，每次调用都是从最上面拿

**Canvas绘画状态包括(哪些状态会被存储)：**

* 当前应用的`变形（移动、旋转和缩放）`
* 以及以下这些属性
  * stokeStyle、fillStyle、globalAlpha、lineWidth、lineCap、lineJoin、font、textAlign、textBaseline……
* 当前的裁切路径

**示例**

```javascript
window.onload = () => {
    let canvasEl = document.querySelector("#tutorial");
    /** @type {CanvasRenderingContext2D} */

    let ctx = canvasEl.getContext("2d");

    ctx.fillStyle = "red";
    ctx.fillRect(10, 30, 30, 10);
    ctx.save();

    ctx.fillStyle = "green";
    ctx.fillRect(50, 30, 30, 10);
    ctx.save();

    ctx.fillStyle = "blue";
    ctx.fillRect(90, 30, 30, 10);
    ctx.save();

    ctx.restore();
    ctx.fillRect(90, 50, 30, 80);

    ctx.restore();
    ctx.fillRect(50, 50, 30, 80);

    ctx.restore();
    ctx.fillRect(10, 50, 30, 80);
};

```

## 4.15.变形 Transform

**Canvas和CSS3一样 也支持形变，都不支持3d形变**

**Canvas形变的4中方法**

* translate(x, y)
  * 无需单位
* rotate(angle)
  * 用以原点为中心旋转canvas，即`沿着z轴旋转`
  * angle单位是`弧度`，是`顺时针旋转`
* scale(x, y)
  * 缩放，支持负数
* transform(a,b,c,d,e,f)
  * 用矩阵做形变，了解

**注意**

* 在做形变之前`调用save方法`保存状态，避免做完形变不知道坐标点在哪里，这是个好习惯
* `先形变在绘图`

## 4.16.Canvas动画

* **canvas如果要实现动画，就需要对画布上的所有图形进行一帧一帧的重绘**
* canvas中有三种方法可以实现动画
  * setinterval
  * setTimeout
  * requestAnimationFrame



**Canvas画出一帧动画的基本步骤（如需画出流畅动画，1s需60帧）**

1. `用clearRect方法清空canvas`。除非接下来绘制的内容完全充满canvas(如背景图),否则需要清空所有
2. `保存canvas状态`，如果加了canvas的状态(例如：样式、形变之类)，又想在每画一帧之时都是原始状态的话，需要先保存以下canvas状态，然后在恢复为原始状态
3. `绘制动画`，即绘制动画中的每一帧
4. `恢复canvas状态`，如果已保存了canvas的状态，可以先恢复，然后在重绘下一帧



**setInerval和setTimeout定时器的缺陷**

* setInterval和setTimeout定时器不是非常的精准，因为他们的回调函数都是放在了红任务队列中 
* 如果`微任务队列`中`一直有未处理完`的`任务`，那么定时器的回调函数就`有可能不会在指定的时间内触发回调`
* 如果想要精确的回调可以使用requestAnimationFrame来执行回调



**requestAnimationFrame**

* 告诉浏览器--你希望执行一个动画，并且要求浏览器在下一次重绘之前调用requestAnimationFrame函数的回调函数来更新动画

* 该方法`会在浏览器下一次重绘之前执行`

* `若想在浏览器下一次重绘之前继续更新下一帧动画，那么在回调函数自身内必须再次调用requestAnimationFrame()`

* 通常浏览的重绘是`每秒60次`左右，也可能会降低(看运行设备的情况)

* ```javascript
  requestAnimationFrame(test)
  
  function test() {
      console.log('todo')
      requestAnimationFrame(test)
  }
  ```

* 