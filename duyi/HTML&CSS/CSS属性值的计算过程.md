
# 问题

a和p都会继承父元素的color，但是为什么a标签的内容不是红色的，而p标签的内容是红色的？

```html
<style>
	.con {
		color: red !important;
	}
</style>

<body>
	<div class="con">
		<a href="">百度</a>
		<p>这是一个段落</p>
	</div>
</body>
```

# 背景

html中的每个元素会有很多的css属性，这些属性必须都要有值，这样浏览器才知道如何显示它；css属性从无值到有值的过程叫做`属性值的计算过程`

[[计算过程.excalidraw]]
![[计算过程.png]]
#  属性计算过程

`每个html元素`都会经历以下的计算过程，才能获取css的属性值

1. **确定声明值**
	1. 参考样式表中没有冲突的声明，作为CSS属性值
2. **层叠冲突**
	1. **比较重要性**
		作者样式表覆盖浏览器默认样式表，自己写的css优先级高于浏览器默认的css
	1. **比较特殊性(比较权重)**
	2. **比较源次序**
		如果在一个选择器中有多个相同的属性，优先选择顺序靠后的
3. **使用继承**
4. **使用默认值**

[计算过程](https://excalidraw.com/#json=jJzWtk9Mq-VUrjckn1aK1,xpWZA7PlNufB50i3ZhPFnQ)
![[Pasted image 20241201182405.png]]

# 举例

假设有以下代码片段，列举h1的计算过程，只举例h1的color、background-color、text-align、font-size、display这几个属性值的计算过程其余的都是一样的步骤


```html
<style>
/* 作者样式表*/
.fat {text-align:center;}
h1 { color: red;font-size: 20px; }

.red {font-size: 40px; }

div h1.red { font-size: 3em;font-size: 30px;font-size: 50px; }
</style>

 <!-- 浏览器默认样式表!-->
 h1 {display:block;font-size:30px;font-weight:bold;...}

<div class="fat"><h1 class="red">测试</h1></div>
```
## 第一步 确定声明值

**参考样式表中没有冲突的声明，作为CSS属性值**

首先会在`作者样式表`(自己写的css)和`浏览器默认样式表`(每个元素都有默认的样式)，找出没有冲突的属性，案例中就是color属性没有冲突，将其作用到h1元素中，color属性不参与后续的计算过程

## 第二步 层叠冲突

目前在样式表中还有font-size、display、font-wight没有确定值，它们的赋值过程如下

比较重要性阶段-》浏览器默认样式的font-size会被pass，font-wight和display由于作者样式表中没有声明赋值，所以值就是浏览器默认样式表中声明的，他俩不参与后续的步骤了

比较特殊性阶段-》可以看到`div .red.h1` 这个选择器的权重最高，所以其他两个选择器的会被pass

比较源次序-》`div .red.h1`中有三个font-size 取顺序靠后的，所以 font-size的值最终就是50px

## 第三步 使用继承

目前为止还剩下text-align和background-color没有找到值，由于background-color不能被继承，所以到下一步才能获取到值，text-align可以继承他的父元素也就是`.fat.div`，所以text-align的值是center

## 第四步 使用默认值

目前还剩下background-color没有找到值，因为每个css都有默认值，所以background-color使用默认值也就是transparent


# 答案

结合属性计算过程的4个步骤

**第一步**

a元素：可以看到作者样式表中没有给a元素的color赋值，但是在浏览器默认样式表中a元素的color是有值的 a:-webkit-any-link {  color: -webkit-link;...}，所以这一步就已经确定了a的值使用的是-webkit-link(chrome内置的变量是一个颜色值)，所以就不用走后续的3步了

p元素：作者样式表和浏览器默认样式表都没有设置颜色，走到第二步

**第二步**
作者样式表和浏览器默认样式表都没有设置颜色，不存在冲突，走到第三步

**第三步**
因为color属性是可以继承的，a元素的父元素设置了color属性的值是 red，所以p元素的color继承父元素就是red 到此a元素和p元素的color属性值已经获取完毕 这两个属性就不会进入下一步了

