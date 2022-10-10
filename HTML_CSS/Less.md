# Less

​	**less(Leaner Style Sheets)**

## 使用

### 浏览器使用

```html

```

## 1.变量

* 使用```@```前缀声明变量

```less
// @自定义变量名：变量值；
@mainColor: red;

.box {
    color: @mainColor;
}
```

## 2.选择器嵌套

```less
.box {
    .left {
        ...
    }
}
```

### 2.1. &

* 代表当前选择器的父级（官方解释）
* （就是代表当前选择器）

```less
// css中
.link:hover{}

// less中
.link {
    &:hover {
    }
}
```



## 3.运算

算数运算可以对任何数字、颜色或变量进行运算

不常用（有点笨）

* 不同单位之间进行算数运算，单位以最```左侧```的单位为准

```less
.box {
    width:10px + 20px;
    color: #ff0 + #aff;
}
```



## 4.混入（Mixins）

* 混入的选择器建议使用class选择器
* 当混入的时候也要使用class

```less
// a.less文件
.w150 {
    width: 150px;
}

// b.css文件
.box {
    height:100px;
    color: #f00;
    .w150();
}
```

### 4.1.混入传参

```less
// a.less文件
// @width:5px  表示默认值是5px
.main_border(@width:5px, @bcolor:red) {
    border: @width, solid, @bcolor;
}

// b.css文件
.box {
    width: 10px;
    height: 100px;
    .main_border(1px, orange)
}
```

### 4.2混入和映射（map）的结合使用

```less
// a.less
.main {
    width: 100px;
    height: 200px;
}

// b.css
// 只想用.main的width属性
.box {
    width: .main()[width];
    height: 200px;
}
```

以下案列弥补了less中不能定义函数的缺陷

```less
.pxToRem(@px) {
    result:(@px / 10) * 1rem
}
.box {
    width: .pxToRem(100)[result]
}
```

## 5.继承（extend）

```less
// a.less
.main {
    border: 1px solid #ccc;
}

// b.css
.box {
    width:100px;
    &:extend(.main)
}

```

## 6. 作用域（scope）

* 在查找一个变量时，首先在```本地查找```变量和```混合```（mixins）
* 如果```找不到```，则从父级作用域继承

