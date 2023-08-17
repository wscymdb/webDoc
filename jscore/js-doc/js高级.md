# JavaScript高级

# 1.函数

## 1.1. this绑定规则优先级

* 隐式绑定的this    ``大于``    默认的this
* 显式绑定的this   ``大于``   隐式绑定的this
* new绑定this   ``大于``   隐式绑定的this
* new绑定的this   ``大于``   bind绑定的优先级
* bind绑定的this   ``大于``  apply、call绑定的this
* new >  bind  >  apply、call > 显 > 隐 >default

``备注：``

* new关键字不可以和call/apply一起使用，但是可以和bind一起使用
  * 因为bind返回的是一个新的函数
* 默认绑定：独立函数调用，函数没有被绑定到某个对象上进行调用，比如直接调用一个函数，this就是window
* 隐式绑定： 通过某个对象发起的函数调用，比如 obj.foo()
* 显示绑定： 手动的设置this的指向，比如 foo.call('aa')

```javascript
function foo() {
    console.log(this)
}
// 显示绑定的优先级高于隐式绑定
// 1.1  apply,call 高于默认绑定
let obj = {foo:foo}
foo.call(obj)  // {foo: foo} 

// bind绑定高于默认绑定
let bar = foo.bind('aaa')
let obj = {name:'zd', baz:bar}
obj.baz()  // aaa

// new绑定的优先级高于隐式绑定
  let obj = {
        name: "zs",
        baz: foo,
      };
  obj.baz(); // {name: 'zs', baz: ƒ}
  new obj.baz(); // {}

// new绑定的优先级高于隐式绑定
// bind 会返回一个新的函数
  let bindFn = foo.bind("aaa");
  bindFn(); // aaa
  new bindFn(); // {}

// bind绑定的this 高于 apply、call绑定的this
  let bindFn = foo.bind("aaa");
  bindFn(); // aaa
  bindFn.call("bbb"); // aaa
  bindFn.apply("bbb"); // aaa
```





## 1.2.this绑定之外的情况

* 非严格模式下，显示的将this绑定成null或undefined，this的指向是默认绑定（window）
* 严格模式下，显示的将this指向null或undefined，this指向就是null或者undefined

```javascript

function foo() {
    console.log(this);
}

foo.call(null);  //  window对象
foo.call(undefined);  // window对象
foo.call("q");  //  包装类 String{q}  就是字符串q

// 
```

* 非严格模式下，创建一个函数的间接调用，this的指向是默认指向
* 由以下案列可知，赋值语句的返回值是要赋的值

```javascript
let obj1 = {
    name: "obbj1",
    foo: function () {
        console.log(this);
    },
};

let obj2 = {};
console.log((obj2.foo = obj1.foo)); // foo函数
// 因为赋值过程的返回值是当前函数，直接调用就是全局调用，this指向window
(obj2.foo = obj1.foo)(); // this指向window
```

## 1.3.箭头函数

* 箭头函数``不会绑定this和arguments和super``

* 箭头函数``没有显式原型``

* 箭头函数不能用作构造函数(因为没有显示原型)

* 如果函数体只有一行代码，默认值返回的是一个对象，必须用括号将对象包裹

  * ```javascript
    // 如果不用括号包裹   （） => {}  这样js引擎解析不出来
    [1,2,3].map(item => ({a:1,b:2})) 
    ```

* 箭头函数中没有this绑定

  * ```javascript
    // 那为什么再箭头函数中可以使用this
    // 因为再当前作用域中没有会向上一级作用域找
    let foo = () => {
        console.log(this)
    }
    // 正常情况下，bind绑定this -> aaa
    // 但是fn的this是window，并没有替换成aaa
    // 由此可见 箭头函数中没有this
    let fn = foo.bind('aaa')
    fn()   // window对象
    
    ```



## 1.4.函数的属性

* name ：表示当前函数的函数名
* length：返回当前函数的参数（形参）
  * rest（剩余参数）参数是不参与参数的个数

## 1.5.arguments

* arguments是一个类数组对象

arguments转换成数组

```javascript
foo(1,2,3,4)
function foo() {
    let arr；
    //方法0 遍历
    // 方法一
    arr = Array.from(arguments)
    // 方法二
    arr = [...arguments]
    // 方法三
    // slice返回的是截取后的数组，没有参数，截取全部
    arr = [].slice.apply(arguments)
}
```

## 1.6.纯函数

函数式编程中有一个非常重要的概念叫``纯函数``，JavaScript符合函数式编程的范式

纯函数的维基百科定义

* 在程序设计中，若有一个函数符合以下的条件，那么这个函数就被称之为``纯函数``
* 此函数在``相同的输入值时``，``产生相同的输出``
* 函数的``输入和输出值意外的其他隐藏信息或状态无关``，也和由I/O设备产生的外部输出无关
* 该函数``不能有语义上可观察的函数副作用``，诸如，‘触发事件’，使输出设备输出，或更改输出值以外的内容等

```javascript
// 调用这个函数，比如输入10，20，结果是30
// 那么在任何地方foo(10,20) 值都是30
function foo(num1,num2) {
    return num1 + num2
}
foo(10,20) //30
foo(10,20) //30

// 以下的情况，如果num发生了变化，那么输出的结果也会变化，不符合纯函数的定义，所以就不是纯函数
var num = 100
function foo(num1, num2) {
    return num1 + num2 + num 
}

foo(10,20) //130
num = 200
foo(10,20) // 230

```

总结纯函数

* ``确定的输入，一定会产生相同的输出``
* ``函数在执行过程中，不能产生副作用``

### 1.6.1.函数副作用

* 副作用（side effect）其实本身是医学的一个概念，比如经常说吃什么要会有副作用
* 在计算机科学中，也引用了副作用的概念，表示在``执行一个函数``时，除了``返回函数值外``，还对调用函数产生了附加影响，比如``修改了全局变量，修改参数挥着改变外部的存储``

## 1.7.柯里化函数

* 柯里化也属于函数式编程里面的一个非常重要的概念
* 不仅被用于JavaScript，还被用于其他编程语言

维基百科解释：

* 在计算机科学中，柯里化（Currying），又被译为卡瑞化、加里化
* 把接收多个参数的函数，变成接收一个单一参数的函数，并且返回接收余下的参数，而且返回结果的新函数的技术

自己的理解：

* 只``传递给函数一部分参数来调用它``，让``他返回一个函数去处理剩余的参数``,这个过程就称之为柯里化
* 柯里化是一种函数的转换，将一个函数从可调用的f(x,y,z),转换为可调用的f(x)(y)(z)
* 柯里化不会调用函数，他只是对函数进行转换

```javascript
function foo(x,y,x) {
    return ...
}
foo(10,20,30)

// 将上面的函数转化成下面的函数的过程就是柯里化

function foo(x) {
    return function(y) {
        return function(z) {
            return ...
        }
    }
}
foo(10)(20)(30)
    
```

### 1.7.1.使用箭头函数转换柯里化

```javascript
let foo = x => y => z => console.log(x,y,z)
foo(1)(2)(3) // 1,2,3
```



### 1.7.2.自动化柯里化函数封装

```javascript
function autoCurrying(fn) {
        return function currying(...arg) {
          if (arg.length >= fn.length) {
            return fn(...arg);
          } else {
            return function (...newArr) {
              console.log(arg);
              return currying(...arg.concat(newArr));
            };
          }
        };
      }

      function foo(x, y, z) {
        // console.log(x + y + z);
        console.log(x + y + z);
      }

      let autoFoo = autoCurrying(foo);
      autoFoo("aa")("bb")("cc"); // aabbcc
```

## 1.8.组合函数

* 组合函数(Compose Function)是在JavaScript开发中的一种对``函数的使用技巧、模式``

```javascript
// 有以下场景，首先对一个数取平方，再乘以2，我们需要用到两个函数才能完成，显得有些麻烦
function dobule(num) {
    return num * 2
}
function pow(num) {
    return num ** 2
}

console.log(dobule(pow(2))) // 16

// 将其转换成组合函数
function composeFn(num) {
    return dobule(pow(num))
}
console.log(composeFn(2)) // 16
```

### 1.8.1.自动化组合函数的封装

```javascript
function compose(...fns) {
  // 1. 边界判断
  let len = fns.length;
  if (!len) return;
  for (let i = 0; i < len; i++) {
    let fn = fns[i];
    if (typeof fn !== "function")
      throw new Error(`index positon ${i} must be function `);
  }

  //
  return function (...args) {
    // 因为做了边界判断，所以第一项一定存在
    // 如果直接遍历不取第一项的值，那么每次遍历传的值都是一样的，取了第一次的结果存起来，下次的参数就是这个结果

    let result = fns[0].apply(this, args);
    for (let i = 1; i < len; i++) {
      let fn = fns[i];
      // 参数加[]是因为apply调用的，参数必须是数组，使用apply可以指定this 更加灵活
      result = fn.apply(this, [result]);
    }

    return result;
  };
}

function foo(num) {
  return num * 2;
}

function fn(num) {
  return num ** 2;
}

function fn1(num) {
  return num ** 2;
}

let newFn = compose(foo, fn, fn1);
console.log(newFn(2)); // 256

```



# 2.浏览器原理

## 2.1.页面渲染过程

![](https://static.oschina.net/uploads/img/201502/10140935_MoRm.png)

### 2.1.2.回流和重绘

#### 2.1.2.1.回流

介绍：

* 也称为重排
* 浏览器解析HTML文件时，``第一次确定``节点的大小和位置称为``布局``（layout）
* 之后对节点的大小、位置修改重新计算称之为``回流``

引起回流的情况

* 比如DOM结构发生了改变（添加一个新的节点或移除节点）
* 比如改变了布局（修改了width、height、padding、font-size等值）
* 比如窗口resize（修改了窗口的尺寸）
* 比如调用getComputedStyle方法获取尺寸、位置信息

#### 2.1.2.2.重绘

介绍：

* 浏览器解析HTML文件时，第一次渲染内容称之为绘制（paint）
* 之后重新渲染称之为重绘

引起重绘的情况

* 比如修改了背景色、文字颜色、边框颜色、样式等

### 2.1.3.如何避免发生回流

回流一定会引起重绘，所以回流是一件很消耗性能的事情

* 修改样式时，尽量一次性修改
  * 比如使用cssText进行修改，或者使用class进行修改
* 尽量避免频繁的操作DOM
  * 使用DocumentFragment或者将要操作的元素拼装好再一并追加到元素上
* 尽量避免通过getComputedStyle获取尺寸、位置等信息
* 对某些元素使用position的absolute或者fixed
  * 这种情况也会引起回流、但是开销相对较小、不会对其他的元素造成影响
  * 这些属性会让元素脱标，重排的时候不会影响标准流的元素

### 2.1.4.特殊解析-composite合成

* 绘制过程中，可以将布局后的元素绘制到都多个合成图层中
  * 这是浏览器的一种优化手段
* 默认情况下，标准流的内容都是被绘制在同一个图层（layers）中
* 而一些特殊的属性，会创建一个新的``合成层``（compositeLayers），并且新的图层可以利用 GPU来加速绘制
  * 因为每个合成层都是单独渲染的

常见的可以从形成新的合成层的属性：

* 3D transforms
* video、canvas、iframe
* opacity动画转换时
* position：fixed
* will-change
  * 一个实验的属性，提前告诉浏览器元素可能发生哪些变化
* animation或transition设置了opacity、transform

``分层确实可以提高性能，但是他以内存管理为代价，因此不应该作为web性能优化策略的一部分过度使用``

## 2.2.script元素和页面解析的关系

* 浏览器在解析HTML过程中，遇到了script元素是不能继续构建DOM树的
* 他会停止构建，首先下载js代码，并且执行js脚本
* 只有等到js脚本执行结束后，才会继续解析HTML，构建DOM树

上述的原因

* 因为js的作用之一就是操作DOM，并且可以修改DOM
* 如果等到DOM数构建完成并且渲染在执行js，会造成严重的回流和重绘，影响页面的性能
* 使用script元素提供的两个属性defer和async解决

### 2.2.1.defer

* defer属性告诉浏览器不用等待脚本下载，再去解析HTML
  * 脚本会由浏览器进行下载，但是不会阻塞DOM Tree的构建过程
  * 如果脚本提前下载好了，他会等待DOM Tree构建完成，在DOMContentLoaded时间之前执行defer中的代码
* 多个带defer的脚本时可以``保持正确的顺序执行``
* 从某种角度来说，defer可以提高页面的性能，并且推荐放到head元素中
* 注意：defer仅适用于外部脚本

### 2.2.2.async

* async也能够让脚本不阻塞页面
* async是让一个脚本完全独立的
  * 浏览器不会因async脚本而阻塞
  * async脚本``不能保证顺序``，他是独立下载，独立运行，不会等待其他脚本
  * asyn不能保证在DOMContentLoaded之前或者之后执行
* defer通常适用于需要在文档解析后操作DOM的js代码，并且对多个script文件有顺序要求的
* asyn通常用于独立的脚本，对其他脚本，甚至DOM没有依赖的

# 3.严格模式

* 严格模式通过抛出错误来``消除一些原有的静默错误``
* 严格模式``让js引擎再执行代码时可以进行更多的优化``（不需要对一些特殊的语法进行处理）
* 严格模式``禁用了再ECMAScript未来版本中可能定义的一些语法``
* 可以给某个文件开启严格模式，也可以单独给函数开启严格模式

开启严格模式后

* 无法意外的创建全局变量
* 严格模式静默失败也报错
* 不允许函数参数有相同的名称
* 不允许0的八进制语法
* 不允许使用with
* ...查mdn

# 4.数据属性修饰符

## 4.1.Object.defineProperty

* 数据属性
  * configurable
    * 默认值true
    * 表示属性是否可以通过delete删除。
    * 一旦设置后续将不能修改
  * enumerable
    * 默认值true
    * 是否可枚举
  * writable
    * 默认值true
    * 是否可修改
  * value
    * 默认情况下时undefined，（通过definedProperty设置属性的情况是）

* 存储属性
  * configurable
  * enumerable
  * set
    * 设置属性时会执行的函数，默认值时undefined
  * get
    * 获取属性时会执行的函数，默认值时undefined（不写get时默认情况）

* 使用存储属性对对象的属性进行操作

  ```javascript
  //  1. 使用Object.defineProperty给obj添加一个name的属性
  // 2. 外界访问name 其实就是访问_name的值，修改也是_name的值
  // 因为return this.name
  let obj = {
          _name: 'zs'
        }
  
        Object.defineProperties(obj, {
          _name: {
            enumerable: false,
            configurable: false
          },
          name: {
            get() {
              // 不能直接返回name 不然会造成死循环
              return this._name
            },
            set(val) {
              this._name = val
            },
            enumerable: true,
            configurable: false
          }
        })
  ```

  

# 5.原型&原型链

* 原型分为对象原型和函数原型

## 5.1.对象原型（隐式原型）

注意：对象中是不能用``对象.prototype``访问的，对象的原型称为``隐式原型``

* JavaScript中，每一个对象都有一个特殊的内置属性[[prototype]]，这个特殊的对象可以指向另外一个对象
* 通过属性key来获取一个value时，会``触发[[Get]]``的操作
* ``首先会检查该对象中是否有对应的属性``，有的话就使用，``没有就访问[[property]]中的``

获取对象原型的方式

* 通过对象的``__proto__``属性可以获取到（但是这是早期浏览器自己添加的，存在一定的兼容性）
* 通过``Object.getPrototypeOf``方法获取

## 5.2.函数原型（显示原型）

函数中的原型叫做``显示原型``

* 可以通过``.prototype``来获取原型
* 可以通过``Object.getPrototypeOf``方法获取
* 可以通过对象的``__proto__``属性可以获取到（但是这是早期浏览器自己添加的，存在一定的兼容性）
* 作用是再``使用new操作符``的时候，将``函数的显示原型``赋值给对``象的隐式原型``，这样对象就可以使用了

![](https://img-blog.csdnimg.cn/2019022822050917.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzIyMDk3,size_16,color_FFFFFF,t_70)

## 5.3.ES5实现继承

寄生组合式继承（思想）

```javascript
      // 函数作用： 创建一个空对象，对象的隐式原型指向o
      function createObject(o) {
        function F() {}
        F.prototype = o;
        return new F();
      }
      // 继承的函数
      function inherit(subType, superType) {
        // 如果适配早期的浏览器，那么就不能使用Object.crate()
        // subTyp.prototype = Object.create(superType.prototype);
        subType.prototype = createObject(superType.prototype);
        Object.defineProperty(subType.prototype, "constructor", {
          enumerable: false,
          configurable: true,
          writable: true,
          value: subType,
        });
      }

      // 实现继承

      function Person(name, age) {
        this.name = name;
        this.age = age;
      }
      Person.prototype.say = () => console.log("say Person");

      function Student(name, age, score) {
        Person.call(this,name, age);
        this.score = score;
      }

      inherit(Student, Person);

      let s1 = new Student("jack", 19, 98);
      console.log(s1);
```

## 5.4.对象的方法（部分）

### hasOwnProperty

* 对象是否有某一个属于``自己的属性``(``不是在原型上的属性``)

### in/for in操作符

* 判断``某个属性``是否在``对象或者对象的原型``上

### instanceof

* 用于``检测构造函数的property``，是否``出现在某个实例对象的原型链``上

### isPropertyOf

* 用于监测某个对象，是否出现在某个实例对象的原型链上

## 5.5.函数也是对象

* 函数也是对象
* 也有隐式原型，他的隐式原型指向Function的显示原型（默认情况下）

# 6.ES6

* 又叫ECMAScript2015或者ES2015，泛指2015及之后发布的版本的统称

## 6.1.class(类)

* 本质上是构造函数和原型链的语法糖

```javascript
      class Person {
        constructor(name, age) {
          this._name = name;
          this.age = age;
        }
// 原型方法
        say() {
          console.log("sss");
        }
      }
```

### 6.1.1.类中的存储器

* set和get除了可以在Object.defineProperty中使用，还可以直接在对象中使用（不常用）

* ```javascript
        let obj = {
          name1: "zd",
          _age: 18,
          get age() {
            return this._age;
          },
          set age(val) {
            this._age = val;
          },
        };
  ```

* 所以在类中也可以使用set和get

* 以下案列的好处是，当要获取坐标，都是成对出现的，我们还需要进行处理（写一个方法...）

* 使用get就不用写方法，直接调用position属性即可

* ```javascript
  class Rectangle {
          constructor(x, y, width, height) {
            this.x = x;
            this.y = y;
            this.width = width;
            this.height = height;
          }
  
          get postion() {
            return { x: this.x, y: this.y };
          }
  
          get size() {
            return { width: this.width, height: this.height };
          }
        }
        let rect1 = new Rectangle(10, 20, 100, 200);
        console.log(rect1.position); // {"x": 10,"y": 20}
  	  console.log(rect1.size); // {"width": 100,"height": 200}
  ```

### 6.1.2.类的静态方法

* 就是ES6之前的``类方法``

* 通常用于定义直接使用类来执行的方法，不需要有类的实例，使用```static关键字```来定义

* ```javascript
  // 静态方法的this也是指向他的调用者，通常就是类本身
  class Foo {
      constructor() {}
      static say() {
          console.log("我睡觉哦");
          console.log(this);
      }
  }
  Foo.say();
  ```


### 6.1.3.继承

* 通过关键字``extends``来实现继承
* class为我们提供了``super``关键字
  * 执行super.method(...)来调用一个父类的方法
  * 执行super(...)类调用父类constructor（只能在当前类的constructor中调用）
* 在当前类的constructor中，``必须要先使用super``才能使用this（将super()放到constructor代码块的首行），否则会报错
* super的使用位置有三个：``子类的构造方法``、``实例方法``、``静态方法``

```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    say() {
        console.log(`${this.name} is say`);
    }
    drink() {
        console.log(`${this.name} is drank`);
    }
}

class Student extends Person {
    constructor(name, age, sno) {
        super(name, age);
        this.sno = sno;
    }
    drink() {
        super.drink();
    }
}

let s1 = new Student("zs", 18, 1001);
s1.say();
s1.drink();
console.log(s1);
```

### 6.1.4.多类继承

* JavaScript是不支持多类继承的
* 使用mixin混入的思想完成

```javascript
// 有running和flayer两个类
// bird类想要继承这两个类

// class Running {
//   run() {
//     console.log('running~')
//   }
// }
// class Flayer {
//   flay() {
//     console.log('flay~')
//   }
// }

// 使用mixin混入的思想
// return class extends basicClass
// 表示 返回一个新的继承basicClass的类
// 外界接收到这个值就是一个新类并且继承了basicClasss类
function mixinRunning(basicClass) {
    return class extends basicClass {
        run() {
            console.log("running~");
        }
    };
}

function mixinFlayer(basicClass) {
    return class extends basicClass {
        flay() {
            console.log("flayer~");
        }
    };
}

class Bird {
    eat() {
        console.log("eating~");
    }
}

// 方案1
// let NewBird = mixinRunning(mixinFlayer(Bird));

// 方案2
class NewBird extends mixinRunning(mixinFlayer(Bird)) {}
let bird1 = new NewBird();
console.log(bird1);
```



## 6.2.new操作符

* 在内存中创建一个新的对象（空对象）
* 将这个对象内部的\__proto__(隐式对象)指向该类的prototype（显示原型）
* 构造函数内部的this，指向这个新的对象
* 执行构造函数的内部代码
* 如果构造函数没有返回空对象，则返回创建的新对象

## 6.3.多态（面向对象的知识，不是es6才出来的）

维基百科定义：

​	``多态``（polymorphism）指为不同数据类型的实体提供统一的接口，或用一个单一的符号来表示多个不同的类型

​	总结：不同数据类型进行同一个操作，表现的行为是不同的，这就是多态

```javascript
// 为不同数据类型的实体提供统一的接口
function sum(a1, a2) {
    return a1 + a2
}
// 调用了同一个接口，但是返回的结果的数据类型却不一样
// 符合多态的定义
sum(10,20)
sum('qq', 'nn')

// 用一个单一的符号来表示多个不同的类型
// foo可以是任意类型的，也符合多态的定义
var foo = 1
foo = 'aa'
foo = []

```

* 从以上的案例和对维基百科的对比来说，Javascript是``一定存在多态的``

## 6.4.字面量的增强

### 6.4.1.属性的简写

```javascript
let name = 'zs'
let obj = {
    name,
    age: 18
}
```



### 6.4.2.方法的简写

* 对象中的函数可以直接简写

```javascript
let obj = {
    drink：function() {}
    say() {}
}
```



### 6.4.3.计算属性的写法

```javascript
//
let key = 'address'

let obj = {
    [key]: '河南'
}

console.log(obj) // {address: '河南'}
```

## 6.5.解构（Destructuring）

### 6.5.1.数组的解构

```javascript
// 采用下标对下标的方式进行解构
// 如果不想声明下标是2的变量，直接空出去，后面的继续写
let arr = [1,2,3,4]
let [c1,c2, ,c3] = arr
console.log(c1,c2,c3) // 1,2,4

// 拓展练习，解构出数组
// 利用了解构和剩余参数的语法
let [c1, c2, ...c3] = arr
console.log(c1,c2,c3) // 1,2,[3,4]

 
// 解构的默认值
// 如果解构对应的值是undefined， 变量的值就是默认值
let [c1='ad',c2,c3='ss',,c4='de'] = arr
// c1 = 1    c4 = de

```



### 6.5.2.对象的解构

```javascript
let obj = {
    name: 'zs',
    age: 18
}

let {name,address:wAddress='hhh'} = obj
// wAddress = hhh

// 对象解构配合剩余参数
let obj = {
    name: 'zs',
    age: 19,
    address: '河南'
}
let {name,...info} = obj
//  name='zs'  info = {age:19, address:'河南'}
```

## 6.6.let&const

### 作用域提升

* let和const声明的变量，没有作用域提升，但是在代码执行的时候还是会被先创建的，只不过只有到变量被赋值的时候才可以被访问

### 暂时性死区

``英文``：TDZ（temporal dead zone）社区给的定义，官方没有这个定义

* 从块级作用域的顶部一直到变量声明完成之前，这个变量处在暂时性死区

  * 在这个区域内，无法访问该变量

* 暂时性死区和定义的位置没有关系，和代码的执行顺序有关系

  * ```javascript
    function foo() {
        console.log(message)
    }
    let message = 'zs'
    foo() // zs
    ```

* 暂时性死区形成后，再该区域内这这个标识符不能够被访问

  * ```javascript
    let message = 'zs'
    function foo() {
        console.log(message)
        let message = 'aa'
    }
    foo()  // 报错
    ```

### 声明的变量不添加window上

### 块级作用域

* 在块级作用域内用``let或const``声明的变量外界无法访问

* 但是在块级作用域内声明的函数在外界允许访问，但是只能``在声明之后``才能访问

  * 因为块级作用域是es6之后才出来的，如果函数写在块级内，那么早期的代码如果在外部访问就会报错，项目中可能会用社区的库，有些库可能不维护了，所以浏览器对块级内声明的函数做了特殊的处理

  ```javascript
  foo() // 报错
  {
      let a = 123
      function foo() {}
  }
  console.log(a) // 报错
  foo()
  ```


## 6.6.模板字符串

### 模板字符串

```javascript
b = 123
a = `${b} + ddd`
//a  123 + ddd
```



### 标签模板字符串

react中会用，暂做了解

```javascript

// 
      function foo(...arg) {
        console.log(arg);
      }

      foo`hahha is ${123} dog ${222}
      .banner {
        color：red;
      }
      `;
// 打印结果
      // [
      //   [
      //     "hahha is ",
      //     " dog ",
      //     "\n      .banner {\n        color：red;\n      }\n      ",
      //   ],
      //   123,
      //   222,
      // ];
```

## 6.7.函数增强

### 6.7.1.默认参数

* 有默认参数的形参尽量写在后面

* 有默认参数的形参，是不会计算在length之内的

* 默认参数之后的所有参数都不参与计算在length之内

  * ```javascript
    function foo(age,name=123,height=2) {}
    foo.length // 1
    ```

* 剩余参数也是放到最后面，但是要放到默认参数之后

```javascript
function foo(name='zs',age=18) {
    console.log(name,age)
}
foo()  // zs,18
```

### 6.7.2.默认参数解构

```javascript
const obj = {name:'zs'}
const {name = 'jack'}  = obj


// 原始写法 
// 当形参是一个对象的时候，设置默认值
function foo(obj = {name:'zs'} ) {
    console.log(obj.name)
}
// 优化
function foo({name} = {name: 'zs'} ) {
    console.log(name)
}
// 进一步优化
function foo({name = 'zs'} = {} ) {
    console.log(name)
}
foo()
```

## 6.8.展开语法（spread syntax）

* 在``函数调用``时使用
* 在``数组构造``时使用
* 在``构建对象字面量``时，也可以使用展开运算符，ES9(ES2018)新添加的
  * 必须是``对象字面量``的形式才可以使用

```javascript
// 基本使用
let obj = {
    name: 'zs'
}
let info = {...obj}

let arr = [1,2,3]
let arr1 = [...arr,9]  // [1,2,3,9]

let hello = 'hello'
let arr2 = [...hello]
```

## 6.9.Symbol

* Symbol是ES6中新增的一个``基本数据类型``，翻译为``符号``。

**Symbol出现的原因：**

* 在ES6之前，对象的key只能是字符串，很容易造成``属性名的冲突``

* 比如传入一个函数的参数是一个对象，函数体中对对象添加一个属性，那么如果传入的对象中已经有函数代码块中同名的key那么就会进行覆盖

  * ```javascript
    function foo(obj) {
        obj.name = 'zs'
    }
    foo({name:'jack'})
    ```

* Symbol就是为了解决这一问题的，用来``生成一个独一无二的值``

  * Symbol值是通过Symbol函数来生成的，生成后可以作为属性名

* ```javascript
  const s1 = Symbol()
  const obj = {
      [s1]: 'li'
  }
  obj[Symbol()] = 'sss'
  ```

* Symbol即使创建多次，他们也是不同的

  * Symbol函数执行后每次创建出来的值都是独一无二的

* 也可以在创建Symbol值的时候传入一个描述description（ES2019新增的特性）

  * ```javascript
     const s1 = Symbol('我是描述哈哈')
    ```

**获取用Symbol属性名的值**

* 不能使用Object.keys()获取key是symbol的键名
* 要通过``Object.getOwnPropertySymbol()``获取

```javascript
const s1 = Symbol()
const obj = {
    [s1]: 'hhh'
}
let symbolKeys = Object.getOwnPropertySymbol(obj)
```

获取两个相同的Symbol

* 通过Symbol.for方法来做到这一点
* 可以通过Symbol.keyFor方法来获取对应的key (当前symbol的描述)

```javascript
const s1 = Symbol.for('des')
const s2 = Symbol.for('des')
consolel.log(s1 === s2) //true

const s3 = ('sdfs')
console.log(Symbol.keyFor(s3)) // sdfs
```

## 6.10.Set

* 类似数组，但是``数据不能重复``
* new Set(iteration)
  * 入参是``可迭代的元素``

常见的属性和方法

* **size**
  * 获取集合元素中的个数
* **add(value)**
  * 添加某个元素，``返回Set对象本身``
* **delete(value)**
  * 从Set中删除和这个值相等的元素，``返回Boolean类型``
* **has(value)**
  * 判断set中是否存在某个元素，``返回Boolean类型``
* **clear()**
  * 清空set中所有的元素，``没有返回值``
* **forEach(callback [, thisArg])**
  * 通过forEach遍历set

```javascript
const s1 = new Set([1,2,3])
s1     //  {1,2,3}
```

### 6.7.1.WeakSet

**弱引用和强引用**

``强引用（strong reference）：``

​	设有个一变量obj，他的值是{name:'zs'}，当js的垃圾回收（GC）运行时，根据可达性的原理，发现{name:'zs'}这个对象被obj变量所引用着，那么就不会将其清理掉，这就是强引用

``弱引用（weak reference）：``

​	设有一个变量obj，他的值是{name:'zs'},虽然他也被obj引用着，但是通过别的方式将这个引用变成弱引用了，那么当GC运行时，就会忽略这个引用，然后{name:'zs'}这个对象就可能被回收

``WeakSet``就是弱集合的意思，也就是说他的引用的值，就是``弱引用``

**WeakSet和Set的区别**

* WeakSet中``只能存放对象类型，不能存放基本数据类型``
* WeakSet``对元素的引用是弱引用``，如果他引用的值没有被其他变量引用，那么GC可以对该引用的值进行回收
* WeakSet是不能遍历的
  * 因为``WeakSet是弱引用``，如果我们遍历获取其中的元素，那么可能造成对象不能正常的销毁

```javascript
cosnt arr = [{name: 'zs'}]
const weakS = new weakSet(arr)
```

## 6.11.Map

* 也是键值对形式
* 键是``任意类型``

常见的属性和方法

* **size**
  * 返回Map中元素的个数
* **set(key, value)**
  * 在Map添加key、value，``并返回整个Map对象``
* **get(key)**
  * 根据key获取Map中的value
* **has(key)**
  * 判断是否包含某一个key，``返回Boolean``
* **delete(key)**
  * 根据key删除一个键值对，``返回Boolean``
* **clear()**
  * 清空所有元素
* **forEach(callback [, thisArg])**

```javascript
const m1 = new Map([
    ['建','值'],
    [1,2]
])
```



### 6.11.1.WeakMap

* WeakMap的``key只能使用对象``，不接受其他类型作为key
* WeakMap的``key对对象的引用是弱引用``，如果没有其他引用引用这个对象，那么GC可以回收该对象
* ``不支持遍历``

```javascript
const obj1 = {name: 'zs'}
const wm = new WeakMap()
wm.set(obj1, 123)
```

# 7.ES7~ES13新特性

## 7.1.ES7新特性

### 7.1.1.Array.includes

查询mdn

### 7.1.2. 指数运算符

求一个数的多少次方，用``**``表示

```javascript
Math.pow(2,3)
2 ** 3
```

## 7.2.ES8新特性

### 7.2.1.Object.values

* 获取对象中所有对值

* ```javascript
  const obj = {
      name: 'zs',
      age: 19
  }
  const val = Object.values(obj)
  // ['zs', 19]
  ```

### 7.2.2.Object.entries

* 可以获取一个数组，数组中``会存放可枚举属性的键值对的数组``

```javascript
const obj = {
    name: 'zs',
    age: 19
}
const ent = Object.entries(obj)
// ent  [ ["name", "zs"], ["age", 19] ]
for (let item of ent) {
	const [key,value] = item
    console.log(key,value)
}

// 如果是一个数组
console.log(Object.entries([1,2,3])) 
// [['0', 1], ['1', 2], ['2', 3]]

// 如果是字符串
console.log(Object.entries('aa'))
//[['0','a'],['1','a']]
```

### 7.2.3.字符串填充

* **padStart(targetLength[, padString])**
* **padEnd(targetLength[, padString])**
* targetLength:当前字符串需要填充的目标长度，``如果小于这个长度``将用padString进行填充

```javascript
// 格式化日期
const hh = "3".padStart(2, "0");
const mm = "15".padStart(2, "0");
console.log(`${hh}:${mm}`);
// 03:15

// 加密身份证或者银行卡
const cardNum = '411989199901011234'
const lastFour = cardNum.slice(-4)
const resNum = lastFour.padStart(cardNum.length, '*')

```

 

## 7.3.ES10

### 7.3.1.flat

* 将一个数组，按照规定的深度遍历，将遍历到的元素和子数组中的元素组成一个新的数组，然后返回
* **flat(depth)**
  * 按照深度去遍历,默认值是1

```javascript
// 更多案例查询mdn文档
const nums = [1,2,[1,2],[[1,2]]]
const flatNum = nums.flat(1)
console.log(flatNum) // [1,2,1,2,[1,2]]
```

### 7.3.2.flatMap

* 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组
* 做了``两步操作``
  * 首先对数组进行``map操作``
  * 然后在对操作之后的数组进行``flat(1)``的操作

```javascript
const msg = ["hello world", "hello zs"];
const newMsg = msg.flatMap((item) => item.split(" "));
console.log(newMsg); // ['hello', 'world', 'hello', 'zs']
```

### 7.3.3.fromEntries

* 将转为entries的对象，变成没转之前的状态

```javascript
const obj = {
    name: 'zs',
    age: 19
}
let objEntries = Object.entries(obj)
// [ ['name', 'zs'],['age', 19] ]
let transObj = Object.fromEntries(objEntries)
// {name: 'zs', age: 19}

```

### 7.3.4.trimStart

* 去除首部的空格

```javascript
const str = ' hello world'
const trimS = str.trimStart()
// 'hello world'
```



### 7.3.5.trimEnd

* 去除尾部的空格

```javascript
const str = ' hello world '
const trimS = str.trimEnd()
// ' hello world'
```

## 7.4.ES11

### 7.4.1.BigInt

* 在早期的JavaScript中，不能正确的表示过大的数字
* 大于``Number.MAX_SAFE_INTEGER``的数值，表示的可能不正确
* 使用``BigInt``即可，在要表示的数后面添加``n``

```javascript
console.log(Number.MAX_SAFE_INTEGER)
// 9007199254740991
console.log(9007199254740991 + 1) // 9007199254740992
console.log(9007199254740991 + 2) // 9007199254740992


// 使用BIGINT
console.log(9007199254740991n + 2n) // 9007199254740993
console.log(9007199254740991n + 5n) // 9007199254740996
```

### 7.4.2.空值合并运算符

* Nullish Coalescing Operator
* 使用``??``，如果??前面的值是``null``或``undefined``,那么就使用``??后面的值``

```javascript
let info = undefined
info = info ?? '默认值'
//  info  默认值  
```

### 7.4.3.可选链操作符

* optional chaining
* 允许读取位于连接对象链深处的属性的值，而不必明确验证链中的每个引用是否有效,如果没有找到，该表达式短路返回值是 `undefined`

```javascript
const obj = {
name:'zs'
}
obj?.asy?.()
```

### 7.4.4.Global this

* 规范了获取全局对象使用``globalThis``

## 7.5.ES12

### 7.5.1.FinalizationRegistry

* 可以让对象在被垃圾回收的时候请求一个回调
* FinalizationRegistry提供一种方法，``当一个在注册表中的对象被垃圾回收时，请求在某个时间点上调用一个清理回调``（清理回调有时被称为finalizer）
* 可以通过``调用register方法，注册热河想要清理回调的对象``，传入该对象和所含的值

```javascript
// register的第二个参数，自定义标识符
// 如果有第二个参数，那么在执行回调的时候，参数就是传入的自定义标识符
let obj = [{ name: "zs" }];
let info = { a: 123 };
const finalization = new FinalizationRegistry((val) => {
    console.log("对象被回收了", val);
});

finalization.register(obj, "obj");
finalization.register(info, "info");

obj = null;

// 检测WeakMap是否会被GC删除
// 结果会调用回调，证明会被GC回收
// 如果将WeakMap换成Map就不会执行回调
let obj = { name: "123" };
let wp = new WeakMap();

wp.set(obj, 123);

const finalization = new FinalizationRegistry(() => {
    console.log("first");
});
finalization.register(obj);
obj = 123;
```

### 7.5.2.WeakRefs

* **WeakRefs: 弱引用**

```javascript
let obj = { name: "zs" };
let wr = new WeakRef(obj);

let finalization = new FinalizationRegistry((val) => {
    console.log("对象被回收", val);
});

finalization.register(obj, "obj");

// 此时obj指向的对象已经被回收了，
obj = null;
```

### 7.5.3.逻辑赋值运算符

* ||=
* ??=
* &&=

```javascript
function foo(message) {
    // 1.||逻辑运算符
    // message = message || '默认值'
    // message ||= '默认值'
    
    // 2. ?? 逻辑运算符
    message = message ?? '默认值'
    message ??= '默认值'
    
    console.log(message)
}
foo(0)  // 0
foo()  // 默认值

let obj = {
    name:'zs'
}
obj = obj && obj.name
obj &&= obj.name
```

### 7.5.4.数字分割

* Numeric Separator
* 使用下划线分割较大的数字

```javascript
const num = 123_0000_0000
```

### 7.5.5.字符串替换

* String.prototype.replaceAll()
* 可以替换全部的字符串
* String.prototype.replace()只能替换一次

## 7.5.ES13

### 7.5.1.Object.hasOwn

* Objcet.hasOwn(obj, propKey)
* 用来替代Object.prototype.hasOwnProperty()方法的

### 7.5.2.method.at

* 查mdn

### 7.5.3.class的新成员

* **Instance publish fields**
  * 实例的公共字段，每个实例都有的字段
* **Static public fields**
  * 类的静态字段
* **Instance private fields**
  * 
* **static block**
  * 类的静态代码块，当类加载的时候执行一次

```javascript
class Person {
    // 所有实例都可有这个属性
    height = 1.88
	constructor() {}
	// 类的静态属性
	static address = '河南'
	// 类的静态私有属性, 只有类内部能够访问
	// 用this.访问
	static #number = 12
    // 类的静态代码块
    static {
        console.log('只有类初始化时候执行一次')
    }
}
```



# 8.Proxy

* 对``一个对象进行代理``，之后``对该对象的所有操作``都是``通过代理对象``来完成的
* new Proxy(target, handler)
* target : 要代理的对象
* handler：被代理对象的行为
  * 内置很多个方法（只列出几个，剩余的查mdn）

```javascript
const obj = {
    name: "zs",
    age: 19,
};

// target表示被代理的源对象
// key 被查看、操作的对象的属性
// val 被修改的新值
// receiver 被代理的对象
const proxyObj = new Proxy(obj, {
    set(target, key, val, receiver) {
        console.log(`监听了${key},值是${val}`);
        target[key] = val;
    },

    get(target, key) {
        console.log(`监听了${key}`);
        return target[key];
    },

    deleteProperty(target, key) {
        delete target[key];
    },

    has(target, key) {
        console.log(`${key}被判断是否存在${target}中`);
        return key in target;
    },
});

proxyObj.name = "jack";
proxyObj.address = "河南";
proxyObj.age;
delete proxyObj.address;
console.log("age" in proxyObj);
```

# 9.Reflect

* Reflect是一个对象，字面意思是``反射``
* 主要提供了很多操作JavaScript对象的方法，有点像Object中操作对象的方法
* 将``Object对象中对 对象本身进行操``作的API迁移到了Reflect上，并且又新增了一些别的方法

Reflect和Object的区别

* 早期的ECMA规范中没有考虑这中``对 对象本身的操作如何设计会更加规范``，所以将API放到了Object上
* 但是``Object作为一个构造函数``这些操作实际放到它身上并不合适
* 另外还包含一些``类似于in、delete操作符``让js看起来会有一些奇怪
* 所以在ES6中``新增了Reflect``，让我们这些操作都集中到了Reflect对象上
* 另外在使用proxy时，可以做到``不操作原对象``

```javascript
const obj = {
    name: "zs",
    age: 19,
};
// 返回 对象本身
console.log(
    Object.defineProperty(obj, "name", {
        configurable: false,
    })
);
// 返回Boolean值  false上面设置了configurable：false
console.log(
    Reflect.defineProperty(obj, "name", {
        enumerable: false,
    })
);

// 都返回Boolean 但是后者的书写的语义更加明确
console.log(delete obj.name); // false
console.log(Reflect.deleteProperty(obj, "name"));  // false
```

## Reflect和Proxy的搭配使用

* 好处一：
  * 代理对象的目的：不直接操作原对象
* 好处二：
  * Reflect.set方法返回Boolean值，可以判断本次的操作是否成功
* 好处三：
  * 见下面案例``好处三``

```javascript
const obj = {
    name: "zs",
    age: 19,
};

Reflect.defineProperty(obj, "name", {
    configurable: false,
    writable: false,
});
// 使用Reflect对对象进行操作，可以进行间接操作对象
// 可以返回一个Boolean值
// 如果对象的属性被保护起来了，那么可以判断操作是否成功
// 避免了严格模式下报错
const proxyObj = new Proxy(obj, {
    set(target, key, newVal, receiver) {
        let isConfig = Reflect.set(target, key, newVal, receiver);
        // let isConfig = target[key];
        console.log(isConfig);
        if (!isConfig) {
            console.log(`${key}属性不支持该操作`);
        }
    },
});
proxyObj.name = "河南";
console.log(proxyObj);
```

案例``好处三``

```javascript
const obj = {
    _name: "zs",
    set name(newVal) {
        this._name = newVal;
    },
    get name() {
        return this._name;
    },
};

//  Reflect的receiver参数表示
//  set/get（obj中的set/get） 方法执行的时候的this

// 如果没有传入receiver，那么obj中的set在调用的时候只监听了一次
// 当调用proxyObj.name的时候其实是调用了两次set方法
// 第一次是调用了proxyObj的set，然后执行了Reflect.set（）
// 调用了obj的set方法，但是此时obj的set中的this指向的是obj
// 操作源对象，proxyObj无法监听到，所以只监听了一次

// 如果传入receiver，
// 当执行Reflect.set()，时候，obj的this已经变成了proxyObj这个代理对象
// this._name其实就是proxyObj._name
// 那么又会执行proxyObj的set方法，所以就可以监听两次

// get方法和上述同理

// 如果不使用Reflect的方法操作对象，那么也只会监听一次，因为obj的this就是指向obj本身

// 所以 如果想要监听两次操作，就使用Reflect，如果不在乎的话，也可以使用target[key]

let proxyObj = new Proxy(obj, {
    set(target, key, newVal, receiver) {
        console.log("设置了值");
        Reflect.set(target, key, newVal, receiver);
    },
    get(target, key, receiver) {
        console.log("获取了值");
        return Reflect.get(target, key, receiver);
    },
});

proxyObj.name = "123";
proxyObj.name;
```

# 10.Promise

* promise是一个类
* 在通过new创建Promise对象时，传入一个回调函数，我们称之为executor
  * 这个回调函数会被立即执行 ，并且给传入两个回调函数resolve，reject
  * 当``调用resolve``回调函数时，``会执行Promise对象的then方法``传入的回调函数
  * 当``调用reject``回调函数时，``会执行Promise对象的catch方法``传入的回调函数

## 10.1.promise的状态

* **pending（待定）**
  * 初始状态，既没有被兑现，也没有被拒绝，
* **fulfilled（已兑现）**
  * 意味着操作成功，执行了resolve时，处于该状态
* **rejected（已拒绝）**
  * 意味着操作失败，执行reject时，处于该状态

## 10.2.注意

* **Executor**是在创建Promise时需要传入的一个回调函数，``这个回调函数会被立即执行``，并且传入两个参数(Executor是传入的回调的叫法)

* Promise的``状态一旦被确定下来，就不会更改``，也不能在执行某一个回调函数来改变状态
  * 一旦``调用了resolve或者reject``，就``不能在调用这两个中的任意一个``，调用了也没有反应

## 10.3.resolve的值

* 如果传入``一个普通的值或者对象``，那么这个值将``作为then方法的回调的参数``
* 如果resolve中传入的是``另外一个Promise``，那么这个``新Promise会决定原来的Promise的状态``
* 如果resolve中传入的是一个``对象``，并且这个``对象中有then方法``，那么会执行then方法，并根据``then方法的结果来决定promise的状态``

```javascript
const p = new Promise((resolve, reject) => {
    resolve("p的resolve");
});
const promise = new Promise((resolve, reject) => {
    // 1. resolve() 普通的值
    // resolve(123);

    // 2. resolve() 一个promise， 这个新的promise会决定原promise的状态
    // resolve(p);

    // 3. resolve() 一个带有then方法的对象
    resolve({
        name: "22",
        then(resolve, reject) {
            // resolve("hhh");
            reject("ss");
        },
    });
});

promise
    .then((val) => console.log(`结果：${val}`))
    .catch((err) => console.error(err));
```

## 10.4.then方法相关

* 可以调用多次then方法，里面的回调都会被调用
* 可以将catch写在then方法中
* 写then的时候后面最好跟一个catch方法，不然如果promise中拒绝了，那么外面就会报错

```javascript
const promise = new Promise((reslove, reject) => {
    reslove('成功的回调')
    // reject('失败的回调')
})

promise.then(() => {})
promise.then(() => {})

promise.then(res => {}, err => {})

```

### 10.4.1.then方法支持链式调用

* then方法的``返回值也是一个Promise``**,所以可以支持链式调用**

```javascript
// then方法返回一个新的promise，这个新的promise的决议（resolve/reject）是等到then方法传入的回调函数有返回值时进行决议的
const promise = new Promise((resolve, reject) => {
    resolve("aaa");
    // reject("ss");
});

// then方法中的返回值的类型(见上面的10.3)
// 普通值
// 新的Promise
// thenable对象（对象中有then方法）
promise
    .then((res) => {
    console.log(res); // aaa
    return "hhh";
    // return {
    //   then(resolve, reject) {
    //     reject(0.11);
    //   },
    // };
})
    .then((res) => {
    console.log(res); // hhh
});

// then方法相当于做了以下的操作
// 所以如果then方法中没有返回值，那么下个then方法中的结果就是undefined
function then(cb) {
    let res = cb();
    return new Promise((resolve, reject) => {
        resolve(res);
    });
}
```

## 10.5.catch方法相关

* 可以调用多次catch方法，里面的回调都会被调用

```javascript
const promise = new Promise((reslove, reject) => {
    reject('失败的回调')
    // reject('失败的回调')
})

promise.reject(() => {})
promise.reject(() => {})
```

### 10.5.1.catch的链式调用

* then方法的``返回值也是一个Promise``**,所以可以支持链式调用**

```javascript
const promise = new Promise((resolve, reject) => {
    reject("ee");
});

// catch方法也会返回一个新的Promise
// catch方法默认返回undefined
// 如果写了return 或者默认的返回，那么决议就是resolve，链式调用的下一级就会进入到then方法中
// 所以默认情况下不能一直.catch().catch()调用
// 如果想使用.catch.catch，那么可以在每个catch中抛出一个异常，这样就可以一直被catch所捕获
promise
    .catch((err) => console.log(err)) //ee
    .then((res) => console.log(res)) // undefined
    .then((res) => console.log(res)); // undefined

const promise1 = new Promise((resolve, reject) => {
    resolve("ee");
});

// 如果有promise的决议是reject的，那么会找到最近的catch方法进行调用
// 所以下面代码会在执行第一个then后，直接执行catch方法
// 使用throw 或者 return一个新的promise（决议是reject）都会进入catch中
promise1
    .then((res) => {
    console.log(res, "asa");
    throw new Error("报错");
})
    .then((res) => console.log(res))
    .catch((err) => console.log(err));

// catch方法相当于做了以下的操作
function catch1(cb) {
    let res = cb();
    return new Promise((resolve, reject) => {
        resolve(res);
    });
}
```

##  10.6.finally方法相关

* es9新增的新特性
* 表示``无论Promise对象无论变成fulfilled还是rejected状态，最终都会被执行的代码``
* finally方法不接受参数，因为无论前面是fulfilled还是rejected状态，他都会执行

## 10.7.Promise的类方法

### resolve方法

* 就是``new promise(resolve => resolve())``的语法糖

### reject方法

* 就是``new promise((resolve,reject) => reject())``的语法糖

### all方法

* 他的作用是``将多个Promise包裹在一起形成一个新的Promise``
* ``新的Promise状态由包裹的所有Promise共同决定``
  * 当``所有的Promise状态变成fulfilled状态``时，``新的Promise状态为fulfilled``，并且会``将所有的Promise的返回值组成一个新的数组``
  * 当``有一个Promise状态为reject时``，新的``Prmise状态为reject``，并且会``将第一个reject的返回值作为参数``，给catch方法

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("11");
    }, 3000);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("33");
    }, 1000);
});

// 如果上面的三个promise中有一个reject，那么不用等到全部promise执行完毕，直接调用catch，剩余没有执行的promise也不会执行了
Promise.all([promise, promise1, promise2])
    .then((res) => {
    console.log(res); //["11", "22", "33"];
})
    .catch((err) => console.log(err));
```

### allSettled

* es11中新增的方法
* 该方法``会在所有的Promise都有结果(settled)``，（无论是fulfilled还是rejected），``才会有最终的状态 ``
* 并且这个Promise的``结果``一定是``fulfilled状态的``

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("11");
    }, 3000);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("33");
    }, 1000);
});
Promise.allSettled([promise, promise1, promise2]).then((res) => {
    console.log(res);
});
// [
//   { status: "rejected", reason: "11" },
//   { status: "fulfilled", value: "22" },
//   { status: "fulfilled", value: "33" },
// ];
```

### race方法

* race是竞赛、竞技的意思
* 表示多个Promise，``谁先有结果那么就使用谁的结果``
* ``无论Promise的状态是fulfilled还是rejected``

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("11");
    }, 300);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("33");
    }, 1000);
});
Promise.race([promise, promise1, promise2])
    .then((res) => {
    console.log(res);
})
    .catch((err) => console.log(err));
// 11
```

### any方法

* es12新增的方法 
* any方法等到一个fulfilled状态，才会决定新Promise的状态
  * 如果``有一个Promise是fulfilled``，那么直接调用then方法，``只会``将当前的内容传进去
* 如果所有的Promise都是rejected，那么也会等到所有的Promise都变成rejected状态，然后会进入catch方法，会报一个``AggregateError``的错误

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("11");
    }, 300);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("33");
    }, 1000);
});
Promise.any([promise, promise1, promise2])
    .then((res) => {
    console.log(res);
})
    .catch((err) => console.log(err));
// AggregateError: All promises were rejected
```

# 11.Iterator(迭代器)

* **在Javascript中，迭代器也是一个具体的对象，这个对象需要符合迭代器协议**（iterator protocol）
  * 迭代器协议定义了``产生一系列值（无论是有限还是无限个）的标准方式``
  * 在JavaScript中这个标准就是一个``特点的next方法``

**next方法有如下的要求**

* 一个``无参数或者一个参数的函数``，返回一个应当``拥有以下两个属性的对象``
* **done(Boolean)**
  * 如果迭代器``可以产生序列中的下一个值``，则为false
  * 如果迭代器``已将序列迭代完毕``，则为true，这种情况下，value是可选的，如果它依然存在，即为迭代结束之后的默认返回值（一般情况是undefined）
* **value**
  * 迭代器返回的任何JavaScript值。``done为true时可以省略``

```javascript
// namesIterator对象就是names的迭代器
const names = ["a", "b", "c"];
let i = 0;
const namesIterator = {
    next() {
        if (i < names.length) {
            return { done: false, value: names[i++] };
        } else {
            return { done: true, value: names[i++] };
        }
    },
};
console.log(namesIterator.next());
console.log(namesIterator.next());
console.log(namesIterator.next());
console.log(namesIterator.next());
```



## 11.1.可迭代对象

* 当一个对象``实现了iterable protocol协议时``，他就是一个可迭代对象
* 这个对象的要求是``必须实现@@iterator方法``，在代码中``使用Symbol.iterator访问该属性``
* 如果想让一个对象变成可迭代的对象，那么需要在这个``对象中返回一个迭代器``
* 这个对象中必须有``属性名是[Symbol.iterator]``方法
  * 方法名 是 Symbol.iterator  这个是Symbol类的方法所以``要用计算属性名``
  * 这个方法返回一个迭代器

```javascript
const infos = {
    name: "jack",
    age: 19,
    height: 1.88,
    [Symbol.iterator]() {
        // 获取key
        // let keys = Object.keys(this);
        // 获取值
        // let values = Object.values(this);
        // 获取entries(键值对)
        let entries = Object.entries(this);
        let index = 0;

        let iterator = {
            next: () => {
                if (index < entries.length) {
                    return { done: false, value: entries[index++] };
                } else {
                    return { done: true, value: undefined };
                }
            },
        };
        return iterator;
    },
};

for (let info of infos) {
    console.log(info);
}
```



## 11.2.可迭代对象的应用

**JavaScript中语法(常用)**

* for...of
* 展开语法（spread syntax）
* yield*
* 解构赋值（Destructuring_assignment）

**创建一些对象时：**

* new Map([iterator]) / new WeakMap([iterator])
* new Set([iterator]) / new WeakSet([iterator])

一些方法的调用

* Promise.all(iterator)
* Promise.race(iterator)
* Array.from(iterator)

## 11.3.自定义类的迭代

```javascript
class Person {
    constructor(name, age, height, friends) {
        this.name = name;
        this.age = age;
        this.height = height;
        this.friends = friends;
    }

    [Symbol.iterator]() {
        let index = 0;
        let values = Object.values(this);
        return {
            next() {
                if (index < values.length) {
                    return { done: false, value: values[index++] };
                } else {
                    return { done: true, value: values[index++] };
                }
            },
        };
    }
}

const p1 = new Person("jack", 18, 1.88, ["rose", "lily"]);

for (let item of p1) {
    console.log(item);
}
```

## 11.4.终端检测器

* 迭代器在某些情况下没有完全迭代的情况下终端
  * 比如遍历的过程中通过break、return、throw中断了循环操作
  * 比如在解构时候，没有解构所有的值
* 就是迭代器的``return方法``

```javascript
const infos = {
    name: "jack",
    age: 19,
    height: 1.88,
    [Symbol.iterator]() {
        // 获取值
        let values = Object.values(this);
        let index = 0;

        let iterator = {
            next: () => {
                if (index < values.length) {
                    return { done: false, value: values[index++] };
                } else {
                    return { done: true, value: undefined };
                }
            },
            return() {
                console.log("进入终端监测");
                return { done: true };
            },
        };
        return iterator;
    },
};
const [name] = infos;
// console.log(name);
```



# 12.Generator(生成器)

## 12.1.什么是生成器

* 生成器是ES6中新增的``一种函数控制、使用的方案``，它可以让我们更加灵活的``控制函数什么时候继续执行、暂停执行``等

* ``生成器``是``由生成器函数``所创造的
* ``生成器函数也是函数``，但是和普通函数有些区别
  * 首先，生成器函数需要在``function后面``加一个符号`` *``

    * 符号挨着function后面，或者挨着函数名前面都行

    * 如果没有function关键字声明的函数，``*``直接加在``名字前面``

    * ```javascript
      let info = {
          *next() {}
      }
      ```
  * 其次，生成器函数``可以通过yield关键字``来控制函数的执行流程
  * 最后，生成器函数返回值是一个``Gererator(生成器)``：

    * 生成器事实上是一种特殊的迭代器

```javascript
function* foo() {
    console.log("first");
    console.log("first");
    yield；
    console.log("first`q");
    console.log("first`q");
}

const gererator = foo();
```

## 12.2.生成器函数执行

* 生成器函数的代码``直接调用是不会执行``的
  * 因为生成器函数``返回一个生成器``
* 通过``调用``返回的``生成器的next方法``来执行生成器函数中的代码块



* 当执行代码的过程中``遇到yield关键字``的时候会``暂停执行``
  * next函数是有返回值的
* 如果想``继续执行``，需要``再次调用next``函数
* ``yield 后面可以跟值``
  * 这个值就会作为next返回值中对象的value的值
* 调用``next的时候可以传参``，但是这个参数会在yield之后的代码中接收到

```javascript
// next函数传参 第一次执行next无法接收到，一般直接传在函数中
function* foo(res) {
    console.log("111", res);
    console.log("111", res);
    const res1 = yield "萘萘萘";
    // return "萘萘萘";
    console.log("222", res1);
    console.log("222", res1);
    const res2 = yield "哈哈哈";
    console.log("333", res2);
    console.log("333", res2);
}

const gererator = foo("第一个");
console.log(gererator.next());
console.log(gererator.next("第二个"));
console.log(gererator.next("第三个"));

// 执行结果如下
// {value: '萘萘萘', done: false} 这种是next函数的返回值
// 如果遇到return后续代码不会执行，next的返回值是 {value: '萘萘萘', done: true}
// 之后在执行next 返回值都是 {value: undefined, done: true}
/*
        111 第一个
        111 第一个
        {value: '萘萘萘', done: false}
        222 第二个
        222 第二个
        {value: '哈哈哈', done: false}
        333 第三个
        333 第三个
         {value: undefined, done: true}
      */
```

## 12.3.生成器函数提前结束

* 除了在函数中直接使用return，也可以用下面的方式

```javascript
function* foo() {
    try {
        console.log("1111");
        const c1 = yield "aaa";
        console.log("2222", c1);
        const c2 = yield "bbb";
        console.log("3333", c2);
    } catch (error) {
        console.log(error);
    }
}
```

### 使用return函数

* return传值后这个生成器函数就会结束，之后调用next不会继续生成值了

* ```javascript
    const genetarorFoo = foo();
        console.log(genetarorFoo.next("@@@"));
        console.log(genetarorFoo.return('###');
        console.log(genetarorFoo.next("$$$"));
  ```

### 使用throw函数

* 抛出异常后可以在生成器函数中捕获异常

* 但是在catch语句中不能继续yield新的值了，但是可以在catch语句外使用yield继续中断函数的执行

* ```javascript
  const genetarorFoo = foo();
  console.log(genetarorFoo.next("@@@"));
  console.log(genetarorFoo.throw(new Error()));
  console.log(genetarorFoo.next("$$$"));
  ```

# 13.生成器替代迭代器

* 生成器本身就是特殊的迭代器
* 生成器函数返回的是一个生成器，可以调用next方法
* 当遇到yield时候，返回值的done是false
* 没有yield时候，返回值的done是true

## 13.1.生成器代替迭代器应用场景

```javascript
const names = ['jack', 'rose', 'lily']

function* createIterator(iter) {
for (let i = 0; i < iter.length; i++) {
yield iter[i]
}
// yield iter[0]
// yield iter[1]
// yield iter[2]
}

const namesIterator = createIterator(names)

console.log(namesIterator.next()) // {value: 'jack', done: false}
console.log(namesIterator.next()) // {value: 'rose', done: false}
console.log(namesIterator.next()) // {value: 'lily', done: false}
console.log(namesIterator.next()) // {value: undefined, done: true}

```

**可迭代对象的优化版本**

* 可迭代对象的标准
  * [Symbol.iterator]方法中返回一个迭代器
* 生成器本身就是一个特殊的迭代器

```javascript
const infos = {
    firends: ["jack", "rose", "lily"],
    *[Symbol.iterator]() {
       yield* this.firends;
    },
};

for (let info of infos) {
    console.log(info);
}
```

## 13.2.yield* 的使用

* yield* 可以产生一个可迭代对象
* yield* 后面跟的必须是一个``可迭代的对象``
* yield*相当于是一种yield的语法糖，他会``依次迭代后面跟着的可迭代对象的内容``

```javascript
const names = ['jack', 'rose', 'lily']
yield* names

相当于

yield 'jack'
yield 'rose'
yield 'lily'


// 简化代码
function* createIterator(arr) {
    yield* arr
}
```

# 14.async\await

## 14.1.异步函数

* async function
* async是asynchronous的缩写，表示异步的

## 14.2.异步函数的执行流程

* 异步函数的内部代码执行过程和普通函数执行是一致的，默认情况下也会被同步执行
* 异步函数有返回值时，和普通函数是有区别的
  * ``情况一``：异步函数也是有返回值的，但是异步函数的返回值相当于被包裹到Promise.resolve中
    * 没有返回值时，默认返回undefined
  * ``情况二``：如果异步函数的返回值是Promise，那么这个返回值的状态由Promise决定
  * ``情况三``：如果异步函数的返回值是一个对象，并且实现了thenable，那么状态会由对象的then方法来决定
* 如果在async中出现了异常，那么程序并不会想普通函数一样报错，而是会作为Promise.reject来传递

```javascript
 async function foo() {
        a.map(ss); // 报错 会在catch中被捕获
        return "aaa";
      }

      // 代码中的报错会在catch中执行
      foo()
        .then((res) => console.log(res))
        .catch((err) => console.log(err));
```



## 14.3.await关键字

* async函数另外一个特殊之处就是可以在``他内部使用await关键字``，而普通函数是不可以的

* await关键字的特点
  * 通常使用await后面``会跟上一个表达式``，这个表达式会``返回一个Promise``
  * await会``等到Promise的状态变为fulfilled状态``，之后``继续执行异步函数``

* 如果await后面是一个``普通值``，那么会``直接返回这个值``

  * 如果该值不是一个 Promise，await 会把该值转换为已正常处理的 Promise，然后等待其处理结果。

  * ```javascript
    async function foo() {
        await 20;
        //相当于 await Promise.resolve(20)
    }
    ```

  * 

* 如果await后面是一个``thenable对象``，那么会``根据thenable的then方法来调用后续的值``

* 如果await后面的表达式，返回的Pomise是``rejected的状态``，那么会将这个``reject结果直接作为Promise的reject值``

# 15.事件循环

**面试题相关的知识储备**

## 15.1.进程和线程

* **``进程(process)``**：计算机``已经运行的程序``，是操作系统管理程序的一种方式
* **``线程(thred)``**：操作系统能够运行``运算调度的最小单位``，通常情况下``他被包含在进程中``

## 15.2.浏览器中的JavaScript线程

* ``JavaScript是单线程的``

**浏览器的进程**

* 目前多数的``浏览器都是多进程的``，当我们``打开一个标签页就会开启一个新的进程``，这样做的目的是为了``防止一个页面卡死而造成所有页面无法响应``，整个浏览器需要强制退出
* 每个进程又有很多线程，其中包括执行JavaScript代码的线程（js是单线程的）

**JavaScript的代码是在一个单独的线程中执行的**

* 这就意味着``JavaScript的代码同一时刻只能做一件事``
* 如果这件事是``非常耗时的``，就意味着当前的线程``会被阻塞``

**耗时的操作不是由JavaScript线程执行的**

* 浏览器每个进程都是多线程的，那么``其余的线程可以来做这个耗时的操作``
* 比如网络请求、定时器、我们只需要在``特定的时候执行应该有的回调即可``

## 15.3.浏览器的事件循环

* 当代码执行的时候如果``遇到了异步的操作``，浏览器会``将这些异步操作交给处理异步操作的线程``
* 等到这些异步操作有结果了，异步操作的线程会将当前异步操作的回调``添加到事件队列里面``
* 等到执行上下文调用栈中的执行上下文全部执行完毕后``（调用栈空了）``，去任务队列中寻找任务，将任务队列中的任务添加到执行上下文调用栈中执行
* 此过程如果又有异步操作，那么继续重复上面的操作，此操作就称为``事件循环``
* 
* DOM监听、XMLHttpRequest、定时器都会被放入到事件队列中
  * 浏览器会监听Dom事件的，然后将其放入到任务队列中
  * ``队列是先进先出的``
* 注意：
* setTimeout(cb,1000)
  * setTimeout是一个函数，里面有一个cb回调
  * 程序执行到setTimeout时，会在调用栈中创建一个函数执行上下文，
  * 然后将cb回调交给执行异步操作的线程
  * 然后setTimeout函数执行上下文弹出栈
  * 等到1000ms后，将其加到任务队列中

```javascript

```



## 15.4.宏任务和微任务

**事件循环中并非只维护着一个队列，事实上有两个队列**

* **``宏任务队列（macrotask queue）``**：Ajax、setTimeout、setInterval、DOM监听、UI Rendering等
* **``微任务队列（microtask queue）``**：Promise的then回调/catch回调、Mutilation Obrserver API、queueMicrotask等

**两个队列的优先级**

* ``main script中的代码优先执行``（编写的顶层script代码）
* 在``执行任何一个宏任务之前（不是队列，是一个宏任务）``，都会``先查看微任务队列中是否有任务需要执行``
  * 也就是宏任务执行前，必须保证微任务队列是空的
  * 如果不为空，那么就优先执行微任务队列中的任务（回调）

## 15.5.面试题

**Promise面试题**

```javascript

console.log("script start");

setTimeout(() => {
    console.log("setTimeout1");
    new Promise(function (resolve) {
        resolve();
    }).then(function () {
        console.log("then4");
    });
    console.log("then2");
});

new Promise(function (resolve) {
    console.log("promise1");
    resolve();
}).then(function () {
    console.log("then1");
});

setTimeout(() => {
    console.log("setTimeout2");
});

console.log(2);

queueMicrotask(() => {
    console.log("queueMicrotask1");
});

new Promise(function (resolve) {
    resolve();
}).then(function () {
    console.log("then3");
});

console.log("script end");

// script start
// promise1
// 2
// script end
// then1
// queueMicrotask1
// then3
// setTimeout1
// then2
// then4
// setTimeout2

/* 解析
        1. main script代码执行
        2. 打印script start
        3. 遇到第一个setTimeout，将其放入宏任务队列
        4. 遇到第一个new Promise 打印promise1，resolve()时 将then回调放入微任务队列
        5. 遇到第二个setTimeout， 将其放入宏任务队列
        5. 打印2
        6. 遇到queueMicrotask，将回调放入微任务队列
        7. 遇到第二个new Promise，resolve()时 将then回调放入微任务队列
        8. 打印script end
        9. 当前调用栈的代码执行完毕，去任务队列中找，根据浏览器在执行宏任务队列之前，会先清空微任务队列的机制，（队列是先进先出的数据结构）
        10. 依次打印 then4  queueMicrotask1  then3
        11. 执行宏任务的第一个回调 打印setTimeout1
        12. 遇到一个new Promise resolve()时 将then的回调放入微任务队列
        13. 打印then2
        14. 当前宏任务执行完毕，执行下一个宏任务，监测微任务队列是否有任务
        15. 打印then4（刚刚推入微任务的回调）
        16. 执行宏任务队列的任务  打印setTimeout2
      */
```

**async、await面试题**

* async函数如果没有返回值，``默认返回undefined``
  * 相当于Promise.resolve(undefined)
* await 后面通常跟一个返回值是Promise的表达式，await会``等到Promise的状态是fulfilled``时，才``执行后续代码``
* 如果Promise的状态是fulfilled，那么Promis``后面的代码相当于被包裹在then函数中``，所以后续代码会被放入``到微任务队列中``

```javascript
async function async1() {
    console.log("async1 start");
    await async2();
    console.log("async end");
}

async function async2() {
    console.log("async2");
}

console.log("script start");

setTimeout(() => {
    console.log("setTimeput");
}, 0);

async1();

new Promise((resolve) => {
    console.log("promise1");
    resolve();
}).then(() => {
    console.log("promise2");
});

console.log("script end");

// script start
// async1 start
// async2
// promise1
// script end
// async end
// promise2
// setTimeput
/*
        1. main script执行
        2. 打印script start
        3. 执行setTimeout，将其给执行异步操作的线程，当时间结束后，异步操作的线程将其放入宏任务队列
        4. 执行async1()
        5. 打印async1 start
        6. 执行 await async2()
        7. 打印 async2
        8. async2函数返回值相当于 Promise.resolve(undefined)
        9. async1中的await等到了fulfilled状态，将其后续代码加入到微任务中
        10. 执行new Promise 打印promise1
        11. 执行resolve（） 将then代码推入微任务队列
        12. 打印script end
        13. 执行微任务队列代码 依次打印async end  promise2
        14. 执行宏任务队列任务， 打印setTimeput
      */
```

# 16.错误处理方案

## 16.1.抛出异常

* 通过throw关键字，抛出一个异常
* throw语句用于``抛出一个用户自定义的异常``
* 当遇到throw语句时，``当前函数执行会被停止``（throw后面的语句不会被执行）

### throw

* throw后面可以跟上``一个表达式来表示具体的异常信息``
* throw后面可以**跟的类型**
  * ``基本数据类型``： 比如number、string、boolean
  * ``对象类型``

### Error类

* JavaScript提供了一个Error类
* Error包含三个属性
  * ``message``： 创建Error对象时传入的message
  * ``name``：Error的名称，通常和类的名称一致
  * ``stack``：整个Error的错误信息，包括函数的调用栈，当我们直接打印Error对象时，打印的就是stack
* Error有一些自己的子类
  * RangeError：下标值越界时使用的错误类型
  * SyntaxError：解析语法错误时使用的错误类型
  * TypeError：出现类型错误时，使用的错误类型



```javascript
function foo() {
    // throw new RangeError("我是错误信息", "ss");
    // throw new TypeError("我是错误信息", "ss");
    throw new SyntaxError("我是错误信息", "ss");
}

foo();
```

## 16.2.异常的捕获

* 当程序``出现异常``，这个异常会``一层一层往上抛``

* 如果整个程序``没有捕获异常``，那么就会抛``给浏览器``，浏览器拿到异常会``在控制台报错，后续代码就不会执行``

* 所以我们应该手动捕获异常

* 使用try  catch来捕获异常

* catch后面可以跟finally

* es10中catch后面绑定的error可以省略

  * ```javascript
    try{
        
    }catch{
        //不用error的信息，可以省略
    }finally {
         console.log("无论是否异常都会执行");
    }
    ```

  * 

```javascript
function foo() {
    // throw new RangeError("我是错误信息", "ss");
    // throw new TypeError("我是错误信息", "ss");
    throw new SyntaxError("我是错误信息", "ss");
}
try {
    foo();
} catch (error) {
    console.log(error.name); //SyntaxError
    console.log(error.message); // 我是错误信息
    console.log(error.stack);
    // SyntaxError: 我是错误信息
    // at foo (异常处理.html:14:15)
    // at 异常处理.html:17:9
}
```

# 17.WebStorage

## 17.1.认识Storage

* **``localStorage：本地存储``**，提供的是一种``永久性的存储方法``，在关掉网页重新打开时，存储内容依然保留
* **``sessionStorage：会话存储``**，提供``本次会话的存储``，在关掉会话（网页）时，存储的内容会被清除

## 17.2.两者区别

* 关闭网页重新打开，localStorage保存，sessionStorage不保存
* 在``同一页面内实现跳转``，localStorage和sessionStorage``都会被保存``
* ``跳转到不同的网页时``(打开一个新的页面)，localStorage保存，sessionStorage不保存

## 17.3.Storage的常见属性

* length
  * 只读属性，返回Storage中对象的个数
* setItem、getItem
* key(index)
  * 返回储存在第index的key名称
* removeItem
* clear

```javascript
localStorage.key(0)
sessionStorage.key(0)
```

# 18.正则表达式

**定义**

* 正则表达式（Regular Expression。简写为regex、regexp、RE）

* 则这表达式使用单个字符串来描述、匹配一系列匹配某个句法规则的字符串
* **简单来说：正则表达式是一种字符串匹配利器，可以帮助我们搜索、获取、替代字符串**

**使用**

* JavaScript中，正则表达式使用``RegExp类``来创建，也有对应的字面量方式

* 正则表达式主要由两部分构成：模式（patterns）和修饰符（flags）

* ```javascript
  const reg = new RegExp('aa','ig')
  const reg1 = /aa/i
  ```

* 

**RegExp实例方法**

* test(str)
  * 执行一个检索，用来查看正则表达式与指定的字符串是否匹配。返回 `true` 或 `false`。
* exec(str)
  * 在一个指定字符串中执行一个搜索匹配。返回一个结果数组或 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/null)。

**字符串中的方法(可以使用正则)(部分方法)**

* match
  * 检索返回一个字符串匹配正则表达式的结果。
* matchAll
  * matchAll的正则必须全局匹配，``加g``
  * 返回一个包含所有匹配正则表达式的结果及分组捕获组的迭代器。
* replace
* replaceAll
* split
* search

**常见的修饰符**

* g
  * 全部的，匹配全部的（global）
* i
  * 忽略大小写（ignore）
* m
  * 多行匹配（multiple）

## 18.1.RegExp中的规则

### 18.1.1 字符类

**字符类（character classes）是一个``特殊的符号``，匹配特定集中的任何符号**

``下面每个符号只能匹配一个``

* **\d**   （digit）
  * 表示0~9的数字
* **\s**   （space）
  * 表示空格符号，包括（制表符\t、换行符\n和其他稀少的字符）
* **\w**  （word）
  * 表示拉丁字母、数字、下划线
* **.**
  * 表示除换行符之外的任何字符

**反向类（inverse classes）**

* **\D** 
  * 表示非数字，除\d以外的任何字符
* **\S** 
  * 表示非空格，除\s之外的任何字符
* **\W**
  * 表示非单位字符，除\w之外的任何字符

### 18.1.2锚点

**符号^和符号$在正则中具有特殊意义，称之为锚点**

* ``^``  表示匹配开头
* ``$``  表示匹配结尾

**词边界**

* 词边界\b是一种检查，像^和$一样，他会检查字符串中的位置E是否是词边界
* 词边界测试\b检查位置的一侧是否匹配\w，而另一侧则不匹配\w

```javascript
//  表示匹配紧挨着name左边的不可以是\w相关的(字母、数字、下划线)这之外其他的都行，
//右边则可以是任意
// 还可以这样 
/\bname\b/  // 表示name两侧紧挨着的不能是\w相关的

const str = "my 我namea is zs";
if (/\bname/gi.test(str)) {
    console.log("first");  
}
// first

//案列  取出时间
const info = `time 12:00 eat some food 234:88`;
const timeRe = /\b\d\d:\d\d\b/g;
console.log(info.match(timeRe));// [12:00]
```

### 18.1.3.集合和范围

**集合**

* 使用``[]``
* 比如说，[eao]意味着在这三个字符e、a、o中任意一个都行

**范围**

* 方括号也可以包含字符范围
* 比如说[a-z]会匹配从a到z范围内的字母
* \d 等同于[0-9]
* \w 等同于[a-zA-Z0-9_]

**排除范围**

* 中括号内开头添加^

* [^0-9] 表示排除0-9

### 18.1.4.量词（quantifiers）

* 用来形容我们所需要的数量的词称为量词
* 数量{n}
* 确切的位数：{5}
* 某个范围的位数： {3，5}
* 某个数字到任意： {2，}

```javascript
/a{3,5}/
// 表示匹配3到5直接数量的a
```

**量词的缩写**

* +
  * 表示一个或多个，同{1,}
* *
  * 表示0个或多个，同{0,}
* ?
  * 表示0个或1个，同{0,1}

### 18.1.5.贪婪模式和惰性模式

* 默认情况下的匹配规则是查找到匹配的内容后，会继续向后查找，一直找到最后一个匹配的内容
  * 这种匹配模式，称之为``贪婪模式（Greedy）``
* ``懒惰模式``中的量词与贪婪模式中是相反的
  * 只要获取对应的内容后，就不再继续向后匹配
  * 可以在量词后面加一个？来启用
  * 所以匹配模式应变成**+?**   ***?**  **??**

* 默认情况下匹配规则是查找到匹配内容后，会继续向后查找，一直找到最后一个匹配的内容
  * 这种匹配方式，称之为``贪婪模式（Greedy）``
* 懒惰模式中的量词与贪婪模式中的是相反的
  * 

```javascript
// 获取书名
const message = "我最喜欢的一本书是《肖申克的救赎》和《小猪佩奇》";
//默认情况 .+ 是贪婪模式
// 获取的结果就是["《肖申克的救赎》和《小猪佩奇》"]
// 因为 》也是 .能够匹配到的
// const bookRe = /《.+》/g;

// 在后面加?表示开启惰性模式
// 表示找到一个符合的就将结果储存，然后继续后续查找
const bookRe = /《.+?》/g;
console.log(message.match(bookRe)); // ['《肖申克的救赎》', '《小猪佩奇》']
```

### 18.1.6.捕获组（capturing group）

* 模式的一部分可以用括号括起来，这称为``捕获组``
* 有两个作用
  * 允许将匹配的一部分作为结果数组中的单独项
  * 将括号视为一个整体

```javascript
      const message = "我最喜欢的一本书是《肖申克的救赎》和《小猪佩奇》";
	// 匹配结果没有加g 也可用matchAll
      const bookRe = /《(.+?)》/;
      console.log(message.match(bookRe)); 
      // 索引0 "《肖申克的救赎》"
      // 索引1 "肖申克的救赎"
```

**命名组**

* 给捕获组添加名称（用数字记录组很困难），这样使用类似match和matchAll中就可以在group中获取了
* 在括号开始的位置放置?\<name>

```javascript
const reg = /(?<hhh>hel)lo/i
```

# 19.防抖和节流

* JavaScript是事件驱动的，大量的操作会出发事件，加入到事件队列中处理
* 对于某些频繁的事件处理会造成性能的损耗，就可以通过防抖和节流来限制事件的频繁发生
* **防抖和节流都是函数**

## 19.1.防抖函数

* 有一个输入框，监听这个输入框的input事件
* 用户想搜索macbook，但是只要一输入就会发送网络请求，这显然是消耗内存的
* 那么就可以用防抖来解决
  * 当用户输入的时候如果连续输入就不发送请求，
  * 如果间隔（比如说间隔200ms）内没有输入，那么就发送一次请求

```javascript
// 基本实现 -取消防抖、是否立即执行
function ymDebounce(cb, delay, immediate = false) {
    // 创建变量
    let timer = null;
    let isInvoke = false;
    // 返回一个函数用来作为oninput的事件
    const _debounce = function (...args) {
        // 判断是否立即执行
        if (immediate && !isInvoke) {
            cb.apply(this, args);
            isInvoke = true;
            return;
        }
        // 如果timer不是空 那么就清空计时器
        // 如果delay时间内再次调用此函数那么上一次的计时器就会被清空
        // 这就是防抖的核心代码
        if (timer) clearTimeout(timer);
        // 创建一个计时器
        timer = setTimeout(() => {
            cb.apply(this, args);
            // 执行过后 将此次的计时器改为null
            // 不写也行，写了更加严谨，让一起回归原本
            timer = null;
            isInvoke = false;
        }, delay);
    };
    // 取消防抖
    _debounce.cancel = function () {
        if (timer) clearTimeout(timer);
        timer = null;
        isInvoke = false;
    };
    return _debounce;
}

// 有返回值
function ymDebounce(cb, delay, immediate = false) {
    // 创建变量
    let timer = null;
    let isInvoke = false;
    let res = undefined;
    // 返回一个函数用来作为oninput的事件
    const _debounce = function (...args) {
        return new Promise((resolve, reject) => {
                // 判断是否立即执行
                if (immediate && !isInvoke) {
                    res = cb.apply(this, args);
                    resolve(res);
                    isInvoke = true;
                    return;
                }

                if (timer) clearTimeout(timer);
                // 创建一个计时器
                timer = setTimeout(() => {
                  try{
                    res = cb.apply(this, args);
                    resolve(res);
                    timer = null;
                    isInvoke = false;
                  }catch(e) {
                    reject(e)
                  }
                  
                }, delay);
        });
    };
    // 取消防抖
    _debounce.cancel = function () {
        if (timer) clearTimeout(timer);
        timer = null;
        isInvoke = false;
    };
    return _debounce;
}
const debounce = ymDebounce(
    (...args) => {
        console.log(args);
        return "eres";
    },
    1000,
    false
);
debounce("jack", 19, 1.88).then((res) => {
    console.log(res);
});
```

## 19.2.节流函数

* 单位时间内，按照固定的频率触发事件
* 比如有一个输入框，监听input事件
  * 如果1秒输入10次，那么就会调用10次请求
  * 现在想1秒内无论输入多少次只发一次请求，这就是节流

```javascript
function ymThrottle(cb, interval, leading = true, trailing = false) {
    // startTime不用Date.now()，用0
    // 因为节流函数默认情况第一次是要触发的
    // 用0的话第一次触发awaitTime一定是<=0
    let startTime = 0;
    const _throttle = function (...args) {
        const nowTime = Date.now();
        // 判断是否立即执行
        // 让awaitTime 大于0即可
        if (!leading && startTime === 0) {
            startTime = nowTime;
        }
        // 这一行代码是核心代码
        // nowTime - startTime 表示代码执行开始到执行中任意时间的间隔
        // interval - (nowTime - startTime) 表示用户传入的时间间隔减去执行的时间间隔
        // 如果这个差值小于等于0的话就执行函数
        const awaitTime = interval - (nowTime - startTime);
        //
        if (awaitTime <= 0) {
            cb.apply(this, args);
            // 将开始时间重新赋值，目的是计算下次的时间间隔
            startTime = Date.now();
        }
    };
    return _throttle;
}

// 实现尾部控制相关-取消和返回值
function ymThrottle(
cb,
 interval,
 { leading = true, traling = false } = {}
) {
    let startTime = 0;
    let timer = null;
    const _throttle = function (...args) {
        return new Promise((resolve, reject) => {
            try {
                const nowTime = Date.now();

                if (!leading && startTime === 0) {
                    startTime = nowTime;
                }

                const waitTime = interval - (nowTime - startTime);

                if (waitTime <= 0) {
                    console.log("first");
                    if (timer) clearTimeout(timer);
                    const res = cb.apply(this, args);
                    resolve(res);
                    startTime = nowTime;
                    timer = null;
                    return;
                }
                // 实现尾部取消
                // 在间隔点之后添加一个定时器
                // 如果是间隔点那么就会取消这个定时器
                if (traling && !timer) {
                    timer = setTimeout(() => {
                        console.log("timer");
                        const res = cb.apply(this, args);
                        resolve(res);
                        startTime = Date.now();
                        timer = null;
                    }, waitTime);
                }
            } catch (error) {
                reject(error);
            }
        });
    };
    // 取消
    _throttle.cancel = function () {
        if (timer) clearTimeout(timer);
    };
    return _throttle;
}
```

# 20.深拷贝&&事件总线

## 20.1.深拷贝

```javascript
// 要用weakMap 用完后没有对newObj进行强引用，newObj会被垃圾回收

function deepCopy(originValue, map = new WeakMap()) {
    // 如果是symbol类型
    if (typeof originValue === "symbol")
        return Symbol(originValue.description);

    // 判断传入的是否是非对象
    if (!isObject(originValue)) return originValue;

    // 函数一般不会深拷贝，函数是用来执行的，深拷贝会造成占用内存
    if (originValue instanceof Function) return originValue;

    // 如果是set类型
    if (originValue instanceof Set) {
        const set = new Set();
        for (const item of originValue) {
            set.add(deepCopy(item));
        }
        return set;
    }

    // 循环引用
    if (map.get(originValue)) return map.get(originValue);

    // 判断是对象还是数组
    const newObj = Array.isArray(originValue) ? [] : {};

    // 循环引用
    map.set(originValue, newObj);

    for (const key in originValue) {
        newObj[key] = deepCopy(originValue[key], map);
    }
    // 如果key是symbol
    const symbolKeys = Object.getOwnPropertySymbols(originValue);
    for (const symbolKey of symbolKeys) {
        newObj[Symbol(symbolKey.description)] = deepCopy(
            originValue[symbolKey],
            map
        );
    }
    return newObj;
}

const s1 = Symbol("我是s1");
const info = {
    name: "zs",
    age: 19,
    address: {
        province: "HENAN",
        city: "信阳",
    },
    color: ["red", "blue", "green"],
};
// 循环引用
info.self = info;

const newInfo = deepCopy(info);
console.log(newInfo);
```

## 20.1.事件总线

```javascript
class ymEventBus {
    constructor() {
        this.eventMap = {}
    }

    // 监听
    on(eventName, eventFn) {
        let eventFns = this.eventMap[eventName]

        if (!eventFns) {
            eventFns = []
            this.eventMap[eventName] = eventFns
        }

        eventFns.push(eventFn)
    }
    // 发射
    emit(eventName, ...args) {
        let eventFns = this.eventMap[eventName]
        if (!eventFns) return

        eventFns.forEach((fn) => fn(...args))
    }

    // 取消
    off(eventName, eventFn) {
        let eventFns = this.eventMap[eventName]
        if (!eventFns) return

        this.eventMap[eventName] = eventFns.filter((fn) => fn !== eventFn)

        if (!this.eventMap[eventName].length) {
            delete this.eventMap[eventName]
        }
    }
}

const bus = new ymEventBus()

const fn1 = (...args) => {
    console.log('vabClick 01', args)
}
const fn2 = (...args) => {
    console.log('vabClick 02', args)
}
bus.on('navClick', fn1)
bus.on('navClick', fn2)

const btnEl = document.querySelector('.nav-btn')
btnEl.onclick = function () {
    console.log('自己触发')
    bus.emit('navClick', '男', 19, 1.88)
    bus.off('navClick', fn1)
    console.log(bus)
}
console.log(bus)
```

# 21.XHR和Fetch

## 21.1.http相关

### 21.1.1.http版本

* **HTTP/0.9**
  * 发布于1991年
  * ``只支持GET``请求方法获取文本数据，当时主要是为了获取HTML页面内容
* **HTTP/1.0**
  * 发布于1996年
  * 支持POST、HEAD等请求方法，支持请求头、响应头等、支持更多种数据类型（不再局限文本数据）
  * 但是浏览器的``每次请求都需要与服务器建立一个TCP连接``，请求处理完成后立即断开TCP连接，每次建立连接``增加性能损耗``
* **HTTP/1.1（目前使用最广泛的版本）**
  * 发布于1997年
  * 增加了PUT、DELETE等请求方法
  * 采用``持久连接(Connection：keep-alive)``，多个请求可以共用一个TCP连接
* **HTTP/2.0**
  * 2015年发布
* **HTTP/3.0**
  * 2018年发布

### 21.1.2.Request Header

* **content-length**
  * 我呢见打大小长度
* **keep-alive**
  * http是基于TCP协议的，但是通常在进行一次请求和响应结束后会立刻中断
  * http1.0中需要手动开启
  * http1.1中默认开启
* **accept-encoding**
  * 请求文件时用到
  * 告知服务器，客服端支持的文件压缩格式（浏览器自动配置）
  * 这样服务的就可以返回对应的压缩格式文件，节省传输时间
* **accept**
  * 请求数据时用到
  * 告知服务器，客户端可接受文件的格式类型

### 21.1.3.response code

* https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status

## 21.2.AJAX

**XMLHttpRequest的state（状态）**

| 值   | 状态             | 描述                                         |
| ---- | ---------------- | -------------------------------------------- |
| 0    | UNSENT           | 代理被创建，但尚未调用open()方法             |
| 1    | OPENED           | open()方法已被调用                           |
| 2    | HEADRES_RECEIVED | send()方法已被调用，并且头部和状态已经可获得 |
| 3    | LOADING          | 下载中，responseText属性已经 包含部分数据    |
| 4    | DONE             | 下载操作已完成                               |

**XMLHttpRequest的事件**

* **onreadyStatechange**
  * 监听readyState的变化
* **loadStart**
  * 请求开始
* **progress**
  * 一个响应数据包到达，此时整个response body都在response中 
* **abort**
  * 调用xhr.abort()取消了请求
* **error**
  * 发生连接错误
* **load**
  * 请求成功完成
* **timeout**
  * 由于请求超时而取消了该请求（仅发生在设置了timeout的情况下）
* **loadend**
  * 在load，error，timeout或abort之后触发

**获取HTTP的响应状态**

```javascript
xhr.status
xhr.statusText
```

**timeout&&abort**

* 在网络请求中，为了避免过长时间服务器无法返回数据，通常会为请求设置一个超时时间（timeout）
  * 当达到超时时间后依然没有获取到数据，那么这个请求会自动被取消掉
  * 默认值为0，表示没有设置超时时间
* 也可以通过``abort``方法，强制取消请求

## 21.3.Fetch

* 使用fetch上传文件时候，``无法查看进度``

* Fetch可以看作是``早期的XMLHttpRequest的替代方案``，他提供了一种更加现代的处理方案
  * 比如返回值是一个Promise
  * 比如不像XMLHttpRequest一样，所有的操作都在一个对象上

### fetch使用

* fetch函数是浏览器提供的一个全局函数，可以直接使用
* fetch函数的返回值是Promise
* 拿到返回值后，如果返回的值是``json``类型，那么就调用json()方法
  * 这个返回值是一个 `Response`（类）对象
  * https://developer.mozilla.org/zh-CN/docs/Web/API/Response/json
  * 如果是其他的类型，就调用相应的方法
* 

```javascript
fetch('http://180.76.235.241:3000/media/list', {
    method: 'get',
    headers: {},
    body: {},
}).then(async (resp) => {
    const res = await resp.json()
    console.log(res)
})
```

