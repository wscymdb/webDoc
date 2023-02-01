# 1.typeScript的特点

* ts的语法是基于js来的，节省开发者的学习成本
* ts可以编译处纯净、简洁的JavaScript代码，并且可以运行在任何浏览器、NodeJs环境中和支持ECMAScript3或以上版本的js引擎中
* ts是一个强大的工具，构建大型项目更加安全
* ts拥有先进的js语法，ts是紧随着js的更新而更新

# 2.ts语法

## 2.1.变量的声明

* ts文件中定义的变量需要指定`**标识符的类型**`

* 完整格式

  * 声明了类型后ts就会进行`类型检测`，声明的类型可以称之为`类型注解(Type Annotation)`

  * ```javascript
    var/let/const 标识符：数据类型 = 赋值；
    let name:string = 'zs'
    ```

## 2.2.变量的类型推导(推断)

* 在开发中，为了方便并并不需要在每一个变量定义时都写上对应的数据类型，可以通过ts本身特性推断出对应的变量类型
  * 在第一次给变量赋值的时候，会根据赋值的内容来推断出变量的类型

## 2.3.typeScript的类型

**js的类型ts都支持，简单的就不写在文档了**

### 2.3.1.Array类型

* 数组类型的定义有两种方式

* Array\<string> 这是一种泛型的写法

* ```javascript
  const names: string[] = ['1', '2', '3']
  const names2: Array<string> = ['1', '2', '3']
  ```

### 2.3.2.object类型

```javascript
let info: {
    name: string;
    age: number;
} = {
    name: "zs",
    age: 19,
};
```

### 2.3.3.函数的类型

* 函数的返回值的类型可以写，亦可以不写， 不写的时候ts会推导出来

```javascript
function sum(n1: number, n2: number): number {
    return n1 + n2;
}

const res = sum(1, 2);
```

### 2.3.4.匿名函数的参数

* 当一个匿名函数被`作为另一个函数的参数时`，最好`不要手动添加类型注解`

* 该函数会**`自动指定类型`**

* ```javascript
  const names = ["s", "s", 1];
  names.forEach((item, index, arr) => {
      console.log(item, index, arr);
  });
  ```

* ts会根据forEach函数的类型以及数组的类型`推断出匿名函数参数的类型`

* 这个过程称为`上下文类型`，因为函数执行的上下文可以帮助`确定参数和返回值的类型`

### 2.3.5.函数和对象类型结合使用

```javascript
// 属性之间的的分割符号可以是逗号，也可以是分号，如果是分号，换行写的情况下可以省略分号
// 属性后面添加问号，表示这个属性是可选的属性
type PonintType = {
    x: number
    y: number
    z?
};

function pointCoordinate(point: PonintType) {
    console.log(point.x);
    console.log(point.y);
}

pointCoordinate({
    x: 123.22,
    y: 223.33,
});
```

### 2.3.6.any类型

* 某些情况下，确实`无法确定`一个`变量的类型`，并且它`可能`会发生一些`变化`，这时候，可以使用any类型
* 可以对any类型的变量`进行任何操作`，包括获取不存在的属性，方法都不会报错
* 可以给一个any类型的变量`赋值任意类型的值 `

```javascript
let id: any = 100100;
id = "12312321";
id = {
  name: "zs",
  age: 22,
};
```

### 2.3.7.unknown类型

* unknown类型用于`描述不确定的变量`
* 在unknown类型的值上`做任何操作都是不合法的`
  * 获取变量的长度、方法等
* 如果想要对unknown类型的值`做操作`，`需要`对该变量进行`类型缩小`

```javascript
let id: unknown = 100100;

// 可以赋值别的类型的值
id = "123123";

//  unknown类型不可以对变量做任何操作
// 必须进行类型的校验(类型缩小),才能根据缩小之后的类型做对应的操作
if (typeof id === "string") {
    console.log(id.length);
}
```

### 2.3.8.void类型

* void通常用来指定`一个函数没有返回值`，那么它的返回值就是void类型
* 如果一个函数没有写任何类型，那么它默认返回值的类型就是void，
* 也可以显示的来指定返回值是void

* **注意**

  * `可以将undefined赋值给void类型`，也就是函数可以返回undefined

* **当基于上下文类型推导，推导出返回类型为void的时候， 并不会强制函数一定不能返回内容**

  * ```javascript
    // forEach根据类型推导出来的函数返回值是void 但是依然可以写返回值
    [1,2,3].forEach((itme) => {
        return 123
    })
    ```

```javascript
// 可以作为函数的返回值的类型
// 如果显示的确定返回的类型 函数的返回值除了undefined之外不可以返回任何东西
function foo(): void {}

// 函数本身也是对象，也有自己的类型
// () => void; 表示函数类型
type barType = () => void;
const bar: barType = () => {};
bar();

// 列子
type ExecuFnType = (...args: any[]) => void;
function delayFn(fn: ExecuFnType, ...args: any[]) {
    setTimeout(() => {
        fn(...args);
    }, 1000);
}

delayFn(() => {}, 123, 321);
```

### 2.3.9.never类型(了解)

* **表示永远不会发生的值**
  * 比如有一个函数是一个死循环或抛出一个异常，那么这个函数就不会返回东西，写void或其他类型都不合适，这时候就可以写never类型
* 一般业务不会用到，`封装和框架中会用到`

**例子**

```javascript
function foo(): never {
    throw new Error("123");
}

foo();

// 例子
// 下面的工具函数 后面的同事在维护的时候
// 原来参数只能是string和number  但是现在的同事想添加boolean
// 然后就在形参的类型加了一个boolean
// 但是没有对这个类型做任何的操作， 那么default的内容就会报错
// 用来提示开发者将新加入的类型做操作，更加严谨
function handleMessage(message: string | number | boolean) {
    switch (typeof message) {
        case "string":
            console.log(message.length);
            break;

        case "number":
            console.log(message);
            break;

        case "boolean":
            console.log(message);
            break;

        default:
            const check: never = message;
    }
}
```

### 2.3.10.tuple(元组)类型

* **tuple和数组的区别**
  * `数组中通常建议存放相同的元素，不同类型的元素是不推荐放在数组中。`(可以放在对象或元组中)
  * `元组中每个元素都可以有自己特定的类型，根据索引值获取到的值可以确定对应的类型`

```javascript
// 数组写法
const info: (string|number)[] = ["code", 22, 1.88]
const item1 = info[0]  // 不能确定类型

// 元组写法
const info: [string|number] = ["code", 22, 1.88]
const item1 = info[0]  // 一定是string类型
```

* 一般来说用在函数返回值类型中比较多
* 可以确定返回的值的每个下标对应的值是什么样的类型

```javascript
function useState(initialState: number): [number, (newValue: number) => void] {
  let val = initialState;

  function setValue(newValue: number) {
    val = newValue;
  }

  return [val, setValue];
}
// 接收的值就知道setCount是函数类型，count是number类型
const [count, setCount] = useState(1);
setCount(1);

```

## 2.4.typeScript的语法细节

### 2.4.1.联合类型

**ts的类型系统允许使用多种运算符，从现有类型中构建新类型**

* 联合类型是`由两个及以上`的其他类型组成的类型
* 表示`可以是这些类型中的任何一个值`
* 联合类型中的每一个类型称为`联合成员(union's members)`

```javascript
let id: string | number = '1001011'
```

### 2.4.2.类型别名

* 使用type关键字声明

```javascript
// 类型别名 type
type MyNumber = number;
const age: MyNumber = 12;

//
type IDType = number | string;
const id: IDType = "123";

//
type InfoType = {
  name: string
  age: number
};
let info: InfoType = {
  name: "zs",
  age: 19,
};

```

### 2.4.3.interface(接口)

* 除了可以用type来声明对象的类型
* 还可以`使用接口(interface)`来`声明对象`的类型

**总结**

* 开发中如果是非对象的定义使用type
* 如果声明对象使用interface（未来代码扩展性更强）

**interface和type的区别**

*  interface只能定义对象的类型
*  interface可以`重复定义同名的类型`, 重复定义的类型规则将会合并
* interface`支持继承`
* ......

```javascript
// 1.  interface只能定义对象的类型
interface Iinfo {
    name: string;
}

// 2. interface可以重复定义同名的类型
// 重复定义的类型规则将会合并
interface IPoint {
    x: number;
    y: number;
}

interface IPoint {
    z?: number;
}

    const coorfinate: IPoint = {
        x: 12,
        y: 22,
    };

    // 3. interface支持继承
    interface IPerson {
        name: string;
        age: number;
    }

    interface IKun extends IPerson {
        slogan: string;
    }

    const p1: IKun = {
        name: "zs",
        age: 19,
        slogan: "你干嘛",
    };
```

### 2.4.4.交叉类型

* 交叉类型表示`需要满足多个类型的条件`
* 交叉类型使用`&`符号
* `大多数用于对象接口的定义`

```javascript
interface IKun {
    name: string;
    age: number;
}

interface ICode {
    name: string;
    coding: () => void;
}

const info: IKun & ICode = {
    name: "zs",
    age: 22,
    coding() {},
};

```

### 2.4.5.类型断言

* 有时候ts无法获取具体的类型信息，这时候就可以使用类型断言(Type Assertions)
* 使用 `as` 关键字

```javascript
// 获取DOM元素  <img class="img" />

// 不使用类型断言 根据类型推导的结果是  Element | null
// 如果做别的操作，需要进行类型缩小
// 使用类型断言  类型就是断言的类型  可以直接做对应的操作

const imgEl = document.querySelector(".img") as HTMLImageElement;
imgEl.alt = "你干嘛";
```

 **类型断言的规则**

* ts只允许类型断言`转换为更具体` 或者 `不太具体`(any/unknown) 的类型版本，此规则可以防止不可能的强制转换

```javascript
// 错误  类型推导name是string 所以不能断言是number
const name = "zs" as number

// ts类型监测是正确，但是js语法层面是错误的，不要这么写
// 先将具体的string类型 转为不太具体的any类型 再将不太具体的any类型转为具体的number类型
const name = ("zs" as any) as number
```

### 2.4.6.非空类型断言

* 使用`!`，表示可以`确定某个标识符是有值的，跳过ts在编译阶段对他的检测`

```javascript
interface IPerson {
    name: string;
    age: number;
    friend?: { name: string };
}

const person: IPerson = {
    name: "zs",
    age: 19,
};

// 因为接口IPerson的friend是可选类型，所以不能直接对friend进行操作

// 解决方案一  类型缩小
if (person.friend) {
    person.friend.name = "ss";
}

// 解决方案二  非空类型断言
// 这种方法有些危险，除非真的确定friend存在
person.friend!.name = "sd";
```

### 2.4.7.字面量类型（literal type）

* 本身单独使用字面量类型的意义不大
* 所以可以`将多个字面量类型联合起来形成一个新的类型`

```javascript
// 基本用法
const name: "zs" = "zs";
let age: 18 = 18;

// 示例
type MethodType = "get" | "post" | "delete";
function request(url: string, method: MethodType) {}

// 直接写值作为参数
request("http://abc.com", "get");

// 使用对象的值作为参数
const info = {
    url: "xxx",
    method: "post",
};

// 报错
// info的method是string类型，不符合MethodType类型
// request(info.url, info.method);

// 字面量推理方案
// 解决方案一 method参数使用类型断言为字面量类型
request(info.url, info.method as MethodType);
// 或者
request(info.url, info.method as "post");

// 解决方案二 让info对象类型是一个字面量类型
const info2: { url: string; method: MethodType } = {
    url: "xxx",
    method: "post",
};

// 或者 类型断言info是一个const  （ts提供的方案）
// "xxx"类型本身就是字符串类型 符合 url的string类型
const info3 = {
    url: "xxx",
    method: "post",
} as const;

request(info2.url, info2.method);
request(info3.url, info3.method);
```

### 2.4.8.类型缩小(type Narrowing)

**定义**

* 在给定的执行路径中，`通过类型保护的方法`，`缩小比声明时更小的类型`，这个过程称为`类型缩小`，也可以称为`类型保护(type guards)`
* 简单来说就是用判断条件来缩小比类型声明时更小的范围

**常见的类型保护**

* typeof
* 平等缩小(eg：===、!==)
* instanceof
* in
* 等等...