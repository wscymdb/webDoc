# 1.typeScript的特点

* ts的语法是基于js来的，节省开发者的学习成本
  * ts可以编译处纯净、简洁的JavaScript代码，并且可以运行在任何浏览器、NodeJs环境中和支持ECMAScript3或以上版本的js引擎中
* ts是一个强大的工具，构建大型项目更加安全
* ts拥有先进的js语法，ts是紧随着js的更新而更新

# 2.ts语法

## 2.1.变量的声明

* ts文件中定义的变量需要指定`标识符的类型`

* 完整格式

  * 声明了类型后ts就会进行`类型检测`，声明的类型可以称之为`类型注解(Type Annotation)`

  * ```javascript
    var/let/const 标识符：数据类型 = 赋值；
    let name:string = 'zs'
    ```

## 2.2.变量的类型推导(推断)

* 在开发中，为了方便`并不需要`在每一个变量定义时都写上对应的数据类型，可以通过ts本身特性推断出对应的变量类型
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
const info: [string,number] = ["code", 22, 1.88]
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

```typescript
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

```typescript
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

```javascript
type IDType = string | number;

function bar(id: IDType) {
    if (typeof id === "string") {
        let len = id.length;
    }
}
```

## 2.5.函数类型

### 2.5.1.函数类型表达式(Function Type Expressions)

* 格式
  * **(参数列表) => 返回值**

```javascript
type FnType = () => string

const bar: FnType = () => { return 'hhh' }
```

**注意**

* **`ts对于传入的函数类型的参数个数不进行检测`**

* 但是在调用的时候会检测

* 简而言之：当`函数定义`不会检测参数的个数，但是当`调用`这个函数的时候就`会检测`

* 定义函数类型和定义函数的形参名字可以不一样(形参本来就可以自定义)

* ```javascript
  // ts对于传入的函数类型的参数个数不进行检测
  type CalcType = (n1: number, n2: number) => number;
  
  function calc(calcFn: CalcType) {
      // 调用的时候会检测
      calcFn(10,20)
  }
  // 不传入参数不会报错
  calc(() => 123);
  
  // 为什么ts会有这种机制
  // 举个例子
  // 传入forEach的匿名函数里面的个数不是都需要用到的
  // 如果ts对函数的参数的个数进行校验，那么每个形参都要写，那么就很麻烦
  // 所以就不校验函数的参数个数
  [1, 2, 3].forEach((item, i, arr) => {});
  export {};
  ```

**官网解析**

```javascript
// 官网解释
// https://www.typescriptlang.org/docs/handbook/2/functions.html#optional-parameters-in-callbacks
// 在JavaScript中定义一个函数，如果形参只有一个
// 调用的时候传入了两个，那么多余的一个参数就会被忽略
// ts中也是如此，所以即使定义函数与定义的类型参数不符合也不会报错
// 但是定义的函数的参数不能大于函数类型的参数个数
// 当在定义回调函数的类型的时候永远不要可选参数
// 除非在调用这个回调函数的时候想要忽略某个参数
//
type CalcType = (n1: number, n2: number) => number;

function calc(calcFn: CalcType) {
    // 如果想要忽略某个参数 可以使用可选参数
    // type CalcType = (n1: number, n2?: number) => number;

    calcFn(10, 2);
}

// 不传入参数不会报错
calc(() => 123);
```



### 2.5.2.函数调用签名(call Signatures)

* `函数本身也是对象`，也可以有自己的属性
*  使用`函数类型表达式`，函数`只能被调用`，不能做其他操作(添加属性等)
* 所以可以使用函数调用签名

```javascript
interface IBar {
    name: string;
    age: number;
    // 让函数可以调用：函数调用签名
    (n: number): number;
}

const bar: IBar = (): number => {
    return 123;
};

bar.name = "bar";
bar.age = 22;
bar(1);
```

### 2.5.3.两者使用场景

* 如果只是描述函数类型本身(函数可以被调用)，使用函数类型表达式
* 如果在描述函数作为对象时可以被调用，同时也有其他属性时，使用函数调用签名

### 2.5.4.构造签名(理解)

* JavaScript的函数也可以使用new操作符调用，typescript会认为这个是一个构造函数
* `所以对于构造函数的类型就要用构造签名`

```javascript
class Person {}

interface ICSTPerson {
    new (): Person;
}
function factory(fn: ICSTPerson) {
    const f = new fn();
    return f;
}

factory(Person);
```

### 2.5.5.可选参数

* 可选参数的类型是 `指定的类型和undefined的联合类型`
* 可选参数必须写在必传参数的后面

```javascript
// 可选参数的类型是 指定的类型和undefined的联合类型
//  这个函数的y参数类型 ：number | undefined
function foo(x: number, y?: number) {}

foo(1);
foo(111, 222);
```

### 2.5.6.默认参数

* 参数有默认值的情况下，`参数的类型注解可以省略`
* 有默认值得参数，是可以接收一个undefined的值，
  * 因为默认情况下当值是undefined才会启用默认值

```javascript
function foo(x: number, y = 100) {}
// foo(1) 内部转换的时候相当于 foo(1,undefined) 
foo(1);
foo(111, undefined);
```

### 2.5.7.函数的重载(了解)

* 在ts中，可以去`编写不同的重载签名(overload signatures)`来表示函数可以`以不同的方式进行调用`
* 一般是`编写两个或两个以上的重载签名`，再去`编写一个通用的函数以及实现`

```javascript
//  实现一个sum函数 既可以进行数字的相加，也可以实现字符串的相加

// 利用函数的重载解决方案

// 1.编写函数的重载签名
function sum(num1: number, num2: number): number;
function sum(num1: string, num2: string): string;
// 2.编写通用的函数
function sum(num1: any, num2: any) {
    return num1 + num2;
}

// 注意 通用函数是不能被直接调用的
// 就是传入的参数只能符合重载签名其中一种类型

sum(10, 20);
sum("aaa", "bbb");
```

### 2.5.8.联合类型和重载

* **在可能的情况下，尽量选择使用联合类型来实现**

```javascript
//  需求 定义一个函数 可以传入字符串或者数组，来获取他们的长度

// 1.重载方式
function getLen(a: string): number;
function getLen(a: any[]): number;
function getLen(a: any) {
    return a.length;
}

// 2.联合类型实现
function getLength(a: string | any[]) {
    a.length;
}
```

### 2.5.9.可推导的this

​	**注意**： `下面this会报错的情况，都是开启了检验this的选项`

#### 2.5.9.1.默认情况下的this

* 在没有对ts做任何配置的情况下，ts在编译时，this可以正确去使用
* 以下代码可以正常运行
* `因为在没有指定this的情况下，this默认情况下的类型是any`

```javascript
function foo(this: {}, num: number) {
  console.log(this);
  return 123;
}

```

#### 2.5.9.2.配置情况下的this

* 可以创建一个`tsconfig.json`文件，并且在其中告知VSCode `this必须有明确类型`(不能是隐式的（any）)

tsconfig.json文件

```json
{
    "compilerOptions": {
        "noImplicitThis": true,  // 设置this必须是有明确类型
    }
}
```

* 在设置了`noImplicitThis`为true时，ts会根据上下文推导this，但是当不能正确推导的时候，就会报错，这时候就需要明确指定this的类型

```javascript
// 下面this会报错， 不能正确推导this的类型
function foo() {
    console.log(this)
}

// 下面this不会报错，根据上下文推导出this的类型是整个obj对象
let obj = {
  say() {
    console.log(this);
  },
};

```

**手动指定this的类型**

* 函数的`第一个参数用于声明this的类型(名词必须叫this)`
* 在后续调用函数传参时，`函数的第二个形参(this之后的形参)作为第一个形参`
* this参数会在编译后被抹除
  * 当ts转为js的时候是没有this这个形参的，this形参的出现只是为了做类型约束

```javascript
// this后面跟自己指定的类型
function foo(this:{}, num: number) {
    console.log(this)
}

foo.call('', 2321)
```

#### 2.5.9.3.this相关的内置工具

**案例公共部分**

```javascript
function foo(this: { name: "zs" }, nmu: number) {
    console.log(this);
}

// 获取函数的类型
/*
    type FooType = (this: {
        name: "zs";
    }, nmu: number) => void
*/
type FooType = typeof foo;
```

**ThisParameterType**

* 用于提取一个函数类型的`this的参数类型`

* 如果这个函数类型没有this参数，返回unknown

* ```javascript
  // 1. 获取当前函数的this的类型
  /* type FooThisType = {
      name: "zs";
  } */
  type FooThisType = ThisParameterType<FooType>;
  ```

**OmitThisParameter**

* 用于移除一个函数类型的this参数类型，并且`返回当前函数的类型`

* ```javascript
  // 2. 省略当前函数类型的this，返回一个纯净的函数类型
  // type PureFooType = (nmu: number) => void
  type PureFooType = OmitThisParameter<FooType>;
  ```

**ThisType**

* 这个类型不会返回一个转换过的类型，它用于`标记一个上下文的this类型`

* ```javascript
  // 3. ThisType: 用于绑定上下文的this
  
  // 下面案列中，
  // 如果想要在say或hi方法中直接使用this.name就会报错
  // 因为根据上下文推导得this得类型时IStore 里面没有name
  // 但是调用得时候 store.say.call(store.state); 传入了this
  // 解决方案一 (如果这个对象中有1000个方法就要定义1000次this类型，麻烦)
  //  在每个方法中指定this得类型
  /*   say(this: IState) {
      console.log(this.name);
    },
    */
  
  // 解决方案二  ThisType<类型>
  // 指定一次this的类型，这个对象中的this的类型都指向 指定得这个类型
  interface IState {
    name: string;
    age: number;
  }
  interface IStore {
    state: IState;
    say: () => void;
    hi: () => void;
  }
  
  const store: IStore & ThisType<IState> = {
    state: {
      name: "zs",
      age: 18,
    },
    say() {
      console.log(this.name);
    },
    hi() {
      console.log(this.name);
    },
  };
  
  store.say.call(store.state);
  ```

## 2.6.面向对象

### 2.6.1.类的定义

* ts中定义类，必须要`提前声明成员属性`
* 如果成员属性的类型没有声明，那么默认是any类型
* 也可以给属性设置初始值，那样就可以省去属性的类型声明(类型推导出来)

```javascript
// ts定义类，必须要声明成员属性
class Person {
  // 声明成员属性
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

const p1 = new Person("zs", 22);

console.log(p1.name, p1.age);
```

* 在默认的strictPropertyInitialization模式下(有配置文件)，属性是`必须初始化`的，如果没有初始话编译时就会报错(了解即可，目前没啥用)

```javascript
class Person {
    // strictPropertyInitialization模式下且不能在构造函数中初始化name和age时必须初始化
    name: string = '1';
    age: number = 0;

    // 如果实在不想初始化
    name!: string ;
    age!: number;
}

const p1 = new Person();

console.log(p1.name, p1.age);
```

### 2.6.2.类的成员修饰符

**在ts中，类的属性和方法支持三种修饰符**

* **public**
  * 修饰的是在任何地方可见，共有的属性或方法，`默认编写的属性就是public的`
* **private**
  * 修饰的是仅在同一类中可见，`私有的属性或方法`
* **protected**
  * 修饰的是仅在类自身以及子类中可见，`受保护的属性或方法`

```javascript
class Person {
  name: string;
  age: number;
  private id: number;  /// 仅在当前类中可以使用
  protected score: number; // 当前类和子类中使用

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  say() {
    console.log(this.id);
  }
}

class Student extends Person {
  study() {
    console.log(this.score);
  }
}

const p1 = new Person("zs", 19);
console.log(p1.age);
```

### 2.6.3.只读属性

* 使用`readonly` 修饰符

```javascript
class Person {
  readonly name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

const p1 = new Person("zs", 19);
console.log(p1.age);

// 只读属性 不可以改写
p1.name = "zs";

```

### 2.6.4.setter和getter

* 使得外界可以操作类的私有属性

```javascript
class Person {
  // 定义私有属性 一般使用 _ 开头
  private _age: number;

  constructor(age: number) {
    this._age = age;
  }

  // 使得外界可以访问类的私有属性
  // 不可以和私有属性同名
  set age(newVal: number) {
    if (newVal < 120 && newVal > 0) {
      this._age = newVal;
    }
  }

  get age() {
    return this._age;
  }
}

const p1 = new Person(19);
p1.age = 12;


```

### 2.6.5.参数属性

* ts提供了特殊的语法，可以把一个构造函数参数转为一个同名同值的类属性
* 可以通过在构造函数参数前 `添加一个可见性修饰符 public private protected readonly 来创建参数属性`，最后`这些类属性字段也会得到这些修饰符`

```javascript
class Person {
  constructor(public name: string, private _age: number) {}
}

// 上面的写法是下面的语法糖

// class Person {
//   name: string;
//   _age: number;
//   constructor(name: string, _age: number) {
//     this.name = name;
//     this._age = _age;
//   }
// }

const p1 = new Person("zs", 19);
console.log(p1.name);
```

### 2.6.6.抽象类 abstract（理解）

* 使用 `abstract` 关键字 声明抽象类和抽象方法
* `抽象方法必须存在于抽象类中`

**抽象类有如下特点**

* 抽象类是不能被实例化的
* 抽象类可以包含抽象方法，可以刻包含有实现体的方法
* 抽象方法必须被子类实现，否者该类必须是一个抽象类

**案例**

```javascript
abstract class Shape {
  abstract getArea(): void;
}
class Rectangle {
  constructor(public height: number, public width: number) {}

  getArea() {
    return this.height * this.width;
  }
}

class Circle {
  constructor(public radius: number) {}

  getArea() {
    return this.radius * 2 * Math.PI ** 2;
  }
}

// 通用函数
function calcArea(shape: Shape) {
  return 100;
}

calcArea(new Rectangle(10, 20));
calcArea(new Circle(10));

// 有一个通用函数 传入一个参数 用于获取几何图形的面积
// 本来这个函数是一个比较通用的函数
// 不能每次多一个类型就改变函数参数的类型
// 所以创建一个父类 让后续所有的类都继承这个父类
// 继承父类的子类们要求都要用getArea方法
// 所以可以在父类中使用抽象方法  也就是没有实现体
// 那么后续继承这个父类的子类就必须有getArea这个方法
```

### 2.6.7.ts中类型检测是鸭子类型

```javascript
class Person {
  constructor(public name: string) {}
}

function foo(fn: Person) {}

// foo的参数类型是Person 
// 但是传入对象也不会报错
// 因为ts的类型检测是鸭子类型 
foo({ name: "ss" });
```

### 2.6.8.ts类的具有的特性

```javascript
class Person {}

// 类的作用
// 1. 可以创建类对应的示例对象
const p = new Person();

// 2. 类本身可以作为类型
const p1: Person = new Person();
function foo(fn: Person) {}

// 类也可以当作是一个构造签名的函数
interface IFactory {
  new (): void;
}
function factory(ctor: IFactory) {}
factory(Person);

```

### 2.6.9.对象类型的属性修饰符

* Property Modifiers
* 可选属性（optional Properties）
  * 在属性名后面加?，标记此属性是可选的
* 只读属性
  * 在类型检查的时候 有此修饰符的属性不可以被修改，
  * 不会改变运行时的行为

```javascript
interface IInfo {
  // 属性? 表示此属性可选
  name?: string;
  // age只能读取  不能修改
  readonly age: number;
}

const info: IInfo = {
  name: "zs",
  age: 19,
};

```

### 2.6.10.索引签名(Index Signatures)（理解）

* 有时候你不能提前知道一个类型的全部属性的名字，但是你知道值的形式是啥样的，这时候你就可以使用索引签名去描述 这个类型可能的值(官网翻译过来的)
*  `ts规定阿数组签名的索引类型必须是string或者number其中一个`

```typescript
// 假设getInfo是三方库的一个方法
// 但是我们不知道这个方法返回的对象具体有啥属性
// 但是我们知道大概的返回值
// 这时候可以使用索引签名
// 描述的类型：索引是字符串 返回值是字符串
// [key: string]: string  key可以自定义写啥都行

interface IInfo {
  // 索引签名：可以通过一个字符串索引 去获取一个字符串类型的值
  [key: string]: string;
}
function getInfo(): IInfo {
  return { name: "zs", slogan: "sss" };
}



// 案例

interface ICollection {
  [index: number]: string;
  length: number;
}

// 有一个比较的函数 要求传入的参数必须是ICollection类型的
// 那么就可以做一些比较通用的事情，比如传进来的一定有length属性

// 如果参数类型是 any[]那么如果要是元组类型或别的类型就还要重新定义，这个函数就没有那么通用了
// 用索引签名就能很好的解决这一问题

function getCollection(collection: ICollection) {
  for (let i = 0; i < collection.length; i++) {
    console.log(collection[i]);
  }
}

const names = ["zs", "ls"];
const tuple: [string, string] = ["t1", "t2"];

getCollection(names);
getCollection(tuple);

export {};

```

**注意**

* 如果索取签名中定义的有其他属性，其他属性的返回类型必须符合string类型返回的类型

* ```typescript
  interface IIndex {
      [key: string]: string
      aaa: string  // 正确
      aaa: number  // 报错
  }
  ```

* 

**奇怪的现象解析**

```typescript
//  ts规定阿数组签名的索引类型必须是string或者number其中一个
// 不可是联合类型
interface ICollection {
  [index: number]: string; // 1
  // [index: string]: any;    // 2
  // [index: string]: string; // 3
}

//  将类型1改为类型2 同样也不会报错
// 但是意思是 索引是string 返回值是any  但是数组的索引是number
// 原因：JavaScript在实际转换的时候会将 names[0] 转换成names["0"]

// 将类型1改为类型3的时候就会报错
// 数组还有自己的方法 比如 names.forEach
// 也符合索引是字符串，但是返回值是一个函数

const names: ICollection = ["zs", "ls"];
```



### 2.6.11.接口的继承

* 使用 `extends` 关键字

**特点**

* 减少代码的重复编写
* 提高复用性
  * 比如: 使用第三方库,既想要保留原来的接口特性,又想要有自己的特性,可以使用继承来完成

```javascript
interface IPerson {
  name: string;
  age: number;
}

interface IKun extends IPerson {
  slogan: string;
}

const jack: IKun = {
  name: "zs",
  age: 19,
  slogan: "你干嘛",
};

```

* 接口可以被类实现
* 使用 `implements` 关键字

```javascript
interface IPerson {
    name: string;
    age: number;
    say: () => void;
}

interface IRun {
    run: () => void;
}

// Person类必须具备IPerson的属性
// 类可以同时实多个接口
class Person implements IPerson, IRun {
    constructor(public name: string, public age: number) {}
    say() {
        return 12;
    }
    run() {}
}

```

### 2.6.12.严格的对象字面量赋值检测

**先来看一下ts中奇怪的现象**

```JavaScript
interface IInfo {
    name: string;
    age: number;
}

// 奇怪的现象一

// 会报错
// const info: IInfo = {
//   name: "zs",
//   age: 12,
//   slogan: "你干嘛",
// };

// 不会报错
let p = {
    name: "zs",
    age: 12,
    slogan: "你干嘛",
};

const info: IInfo = p;

// 奇怪的现象二
function printInfo(info: IInfo) {}
// 报错
// printInfo({ name: "zs", age: 123, slogan: "111" });
// 不报错
let obj = { name: "zs", age: 123, slogan: "111" };
printInfo(obj);

// 第一次创建的对象称之为新鲜的对象(fresh)
// 对于一个新鲜的字面量会进行严格的类型检测,必须完全满足类型的要求(不能有多余的属性)

// 当这个对象去别的地方使用了,就不新鲜了, 不会严格的检测,所以会出现这种现象
// 不用纠结 这是ts的规则

```

**原因**

**GitHub成员issue的解释**

* 每个字面量被初始化的时候都是认为是新鲜的

* 当一个新的字面量对象分配给一个变量或传递给一个非空类型的参数时,对象字面量指定目标类型中不存在的属性是错误的

* 当类型断言或对象字面量的类型扩大时,新鲜度就会消失

  * 当一个对象初次创建属于新鲜的,去别的地方用了就不新鲜了

  * ```JavaScript
    let obj = {name: 'zs'} // 新鲜
    let bbj = obj  // 不新鲜
    ```

  * 

### 2.6.13.枚举类型

* 将一组可能出现的值，一个个列举出来，定义在一个类型中，这个类型就是枚举类型
* 枚举允许开发者定义一组命名常量，常量可以是数字、字符串类型
* 使用 `enum` 关键字

```javascript
//  定义枚举类型
enum Direction {
  UP,
  LEFT,
  RIGHT,
}

const d1: Direction = Direction.LEFT;
```

**枚举类型的值**

* 枚举类型默认是有值的，第一个值默认是0，之后的值依次递增
* 也可以给枚举赋值number类型的，后面的枚举的值也是依次递增
* 如果赋值是string 后面的值都要手动赋值

```javascript
//  定义枚举类型
enum Direction {
  UP, // 0
  LEFT = 'left', // left
  RIGHT = 'right',  // right
}

const d1: Direction = Direction.LEFT;
console.log(d1);  // left
```

## 2.7.泛型

### 2.7.1.泛型实现类型参数化

* `让类型变为参数化`
  * 比如调用一个函数的时候让调用者传入类型，这样类型的可扩展性更强
* 需要用到一种`特殊的变量-类型变量(type variable)`，它作用于类型，不作用于值
  * \<Type = type>就是类型变量
  * type 是默认值

```javascript
// 定义一个函数 返回值是传入的形参

// 如果不给形参指定类型 那么r1/r2结果都是any类型, 那么就会丢失了类型限制
// 如果给形参指定一个联合类型 那么r1\r2的结果就都是联合类型
// 但是需要传入的啥类型，返回值就是啥类型
// 所以可以将类型作为参数，让调用者传来

function returnParameter<Type>(arg: Type) {
    return arg;
}

// 完整写法
const r1 = returnParameter<string>("aaa");
const r2 = returnParameter<number>(111);

// 省略写法
// 可以不传入参数的类型，ts会根据实参类型推导出类型
// 如果类型推断不正确 需要手动填写
const r3 = returnParameter("bbb");
const r4 = returnParameter(222);

// 案例
// 元组：useState函数
function useState<Type>(initialState: Type): [Type, (newState: Type) => void] {
  let state = initialState;
  function setState(newState: Type) {
    state = newState;
  }

  return [state, setState];
}

const [count, setCount] = useState(100);
const [message, setMessage] = useState("hello world");
const [banners, setBanners] = useState<any[]>([]);
```

**泛型支持传入多个类型**

```javascript
function foo<T, E>(arg1: T, arg2: E) {
    
}
foo(1,'1')
foo(2, 3)
```

### 2.7.2. 泛型接口

* 接口使用泛型`可以设置默认值`
* 接口使用泛型`无法进行类型推导`

```javascript
// 接口使用泛型
// 接口使用泛型可以设置默认值
interface IKun<Type = number> {
  name: Type;
  age: number;
  slogan: Type;
}

// 接口使用泛型无法进行类型推导
const k1: IKun<string> = {
  name: "zs",
  age: 19,
  slogan: "你干嘛",
};
```



### 2.7.3. 泛型类

* 类使用泛型`可以设置默认值`
* 类使用泛型`可以进行类型推导`

```javascript
class Point<Type = string> {
  constructor(public x: Type, public y: Type) {}
}

// 类使用泛型 可以进行类型推导
const p1 = new Point(10, 20);
const p2 = new Point<boolean>(true, false);
```

### 2.7.4.泛型约束

**Generic Constraints**

**基本使用**

* 有时候希望传入的类型有某些共性，但是这些共性可能不在同一个类型中
  * 比如string和array都是有length的，或者某些对象也可以手动写length属性的
  * 那么只要是拥有length属性都可以作为参数类型

```javascript
interface IInfo {
  length: number;
}

// 有一个函数 要求传入参数的必须有length属性
function getInfo(arg: IInfo) {
  return arg;
}

// 如果使用上面的写法 那么接收的结果类型是IInfo
// 但是传入的是string 和 any[] 就会造成类型缺失
const l1 = getInfo("123");
const l2 = getInfo(["1", 2, 3]);

// 使用泛型约束  即可让结果类型不会缺失
// 又可以限制传入的参数必须有length属性
// <Type extends IInfo> 表示传入的类型必须有IInfo类型的属性，也可以有别的属性
function getInfo2<Type extends IInfo>(arg: Type): Type {
  return arg;
}
const l3 = getInfo2("123");
const l4 = getInfo2(["1", 2, 3]);
const l5 = getInfo2({ length: 123, name: "zs" });
// const l5 = getInfo2(123);  // 报错
```

**对传入泛型类型的参数进行约束**

```javascript
// 此函数  根据key 返回key在obj的值
// 要求传入的key必须是obj中的属性

// K extends keyof O 表示 K必须是 keyof O("name" | "age" | "height") 中的属性
// 那么就可以确保 传入的key一定包含在obj中
function getObjectProperty<O, K extends keyof O>(obj: O, key: K) {
  return obj[key];
}

let info = {
  name: "zs",
  age: 19,
  height: 1.88,
};

getObjectProperty(info, "name");

// keyof 使用说明
// keyof  xxx： 获取所xxx有的key 组成一个联合类型返回

interface IKun {
  name: string;
  age: number;
}

type IKunKeys = keyof IKun; // "name" | "age"
```

### 2.7.5.映射类型

* 有时候，`一个类型需要基于另外一个类型`，但是`又不想拷贝一份`，这时候可以考虑用映射类型
  * 大部分内置工具都是通过映射类型来实现的
  * 大多数类型体操的题目也是通过映射类型完成的

```javascript
// 可以把MapIkun当作一个函数
type MapType<T> = {
  // 下面的操作其实就相当于遍历了一遍 T
  // name: string
  // age: number
  [property in keyof T]?: T[property];
  // 拷贝的时候让类型变为可选类型
  // 如果使用接口的继承那么就没有这么灵活
  // [property in keyof T]?: T[property];
};

interface IKun {
  name: string;
  age: number;
}

// 拷贝一份IKun
type NewKun = MapType<IKun>;
```

**可选类型的修饰符**

* 支持 `readonly` 和 `? `

```javascript
// 在拷贝的时候 可以添加? 或readonly修饰符
type MapType<T> = {
  readonly [property in keyof T]?: T[property];
};

interface IPerson {
  neme: string;
  age: number;
  height: number;
  isGay: boolean;
}

type newPerson = MapType<IPerson>;
```

**可选类型修饰符的符号**

* 可在修饰符前面添加 `-` 或 `+`
* 如果是添加修饰符的话，可以省去`+`，默认就是`＋`

```javascript
// 在拷贝的时候 如果拷贝源 是有修饰符的可以去掉
type MapType<T> = {
  // 表示如果属性的类型有修饰符 那就去掉修饰符
  -readonly [property in keyof T]-?: T[property];
};

interface IPerson {
  neme?: string;
  age: number;
  readonly height: number;
  isGay: boolean;
}
type newPerson = MapType<IPerson>;
```

### 2.7.6.内置工具和类型体操

类型体操地址：https://github.com/type-challenges/type-challenges/blob/main/README.md

类型体操解答：https://ghaiklor.github.io/type-challenges-solutions/zh/



## 2.8.ts知识拓展

### 2.8.1.模块化

**typescript中最主要使用的模块化方案是ES Module**

### 2.8.2.非模块化

* **ts认为什么是一个模块**

  * JavaScript规范声明`任何没有export`的JavaScript`文件`都应该被认为是一个`脚本`，而`非一个模块`，ts亦是如此
  * 在一个`脚本文件中`，`变量和类型`会被声明在共享的`全局作用域`，将多个输入文件合并成一个输出文件，或在HTML使用多个script标签加载这些文件
    * 比如有10个脚本中，他们的作用域都是同一个，所以不能同名

* **如果你有一个文件，现在没有任何import或者export，但是你希望它被作为模块处理，添加如下代码**

  * ```javascript
    export {}
    ```

* **这会把文件改成一个没有导出任何内容的模块，这个语法可以生效**

### 2.8.3.内置类型导入

* ts 4.5 也允许单独的导入， 你需要`使用type前缀`，`表明被导入的是一个类型`
* 也可以不加
* `使用type前缀的好处`是：可以让一个非typescript编译器比如：Babel、swc、esbuild知道什么样的导入可以被安全移除，`提高编译性能`

```javascript
import { type IPerson, type IInfo} from './type'
const p1: IPerson = {name: 'zs',age: 19}
```

**导入多种类型**

```javascript
// 如果导入的是多种类型 也可将type写在最前面，表示导入的都是type
import type { IPerson, IInfo } from './type'
const p1: IPerson = {name: 'zs',age: 19}
```

### 2.8.4.命名空间(了解)

* 命名空间是在ts早期的时候由于ES模块没有出来，所以定制了自己的模块化，就是命名空间
* 但是由于ES模块已经拥有了命名空间的大部分特性，因此更推荐使用ES模块 
* `官方推荐使用ES module 不推荐使用命名空间`

```javascript
namespace price {}
namespace price {}
```

### 2.8.5.类型查询

**`.d.ts`文件**

* 这种文件是用来做类型的声明(declare)，称之为`类型声明(Type Declaration)` 或者 `类型定义(Type Definition)`文件
* 它仅仅用来类型检测，告知typescript我们有哪些类型
* `在这个文件中声明的类型 全局都可以使用`
* 如果三方库或自己编写的代码，没有做类型声明，那么全局无法使用，会报错

**typescript会在哪里查找类型声明**

* 内置类型声明
* 外部定义类型声明(三方库)
* 自定义类型声明



### 2.8.6. 声明文件使用场景

**内置类型声明**

* 是typescript自带的、帮助我们内置了JavaScript运行时的一些标准化的声明文件
  * 包括比如：Function、String、Math等内置类型
  * 也包括运行环境中的DOM API，如Window、Document等 

**外部定义类型声明**

* 外部类型声明通常是使用一些三方库时，需要的一些类型声明
* 这些库通常有两种类型声明方式
  * 方式一：在自己库中进行类型声明(编写.d.ts文件)
  * 方式二：通过社区的一个`公有库Definitely Typed`存放类型声明文件
* 如果三方库中既没有声明类型的文件，社区中也没有对应的文件，那么就需要自己手动写

**自定义声明**

* `平时使用`的代码中用到的类型，直接在`当前文件中进行定义`或者`在业务文件`夹某一个位置`编写类型`即可
* 特殊情况下，需要在全局声明
  * 假设在全局定义了一个变量yy，然后在别的文件中使用就会报错，这时候就需要在.d.ts文件中去声明这个变量

### 2.8.7.declare 

#### 2.8.7.1.声明模块

**语法**

` declare module '模块名'`

可以自己声明模块，比如lodash这个模块不能使用的情况下，可以自己声明这个模块

```typescript
// 大括号表示此模块里有什么内容， 可加可不加
// 加了 后续使用会有提示
declare module "lodash" {
    export function join(arg: any[]): any
}
```

#### 2.8.7.2.声明文件

* 比如在ts文件中使用import导入 一张图片或者其他文件，那么ts会报错，因为没有声明这个模块
* 所以需要在.d.ts文件中声明

```typescript
// 声明文件模块
// 告诉ts  所有以.jpg 的形式的都是模块
// * 表示通配符
declare module '*.jpg'

// declear module '*.svg'

```

#### 2.8.7.3.声明命名空间

* 如果是通过cdn的方式引入的文件，使用声明文件或声明模块，是不太合适的
* 这时候可以使用声明命名空间

```typescript
declare namespace $ {
    export function ajax(settings: any): any
}
```

### 2.8.8.tsconfig文件

* 作用一：让typescript compiler在编译的时候，知道如何去编译typescript代码和类型检测
  * 比如是否允许不明确的this，是否允许隐式any
  * 比如将代码编译成什么版本的js代码
* 作用二：让编辑器(如vs code)，可以按照正确的方式识别ts代码
  * 对语法进行提示、显示错误等

**tsconfig文件的选项**

官网：https://www.typescriptlang.org/tsconfig

###  2.8.9.条件类型

**Conditional Types**

**语法**

​	**SomeType `extends` OtherType  ?  TrueType  :  FalseType**



**基本使用**

```javascript
type IDType = number

//  判断IDType是否继承number类型
// 表达式只能使用extends关键字
// 也就是说只能判断继承不继承  不能IDType === number之类的操作符
type resType = IDType extends number ? number : string

//  举个栗子  函数的重载

// function sum(num1: string, num2: string): string
// function sum(num1: number, num2: number): number

// 上面两个可以复用，只定义一个函数的重载

// 这样写参数的类型虽然都要一样 但是写两个对象也可以
// function sum<T>(num1: T, num2: T): T

// 限制参数的类型,但是返回值是联合类型
// function sum<T extends number | string>(num1: T, num2: T): T

// 利用条件表达式判单返回类型
function sum<T extends number | string>(
  num1: T,
  num2: T
): T extends number ? number : string

function sum(num1, num2) {
  return num1 + num2
}

sum(1, 3)
sum('abs', 'vsx')
```

### 2.8.10.在条件类型中推断（infer）

* 条件类型中提供了`infer关键字`，可以从正在比较的类中推断类型，然后在true分支里引用该推断结果
* infer 只能用于条件类型中

```typescript
type CalcFnType = (n1: number, n2: number) => number

function foo() {
  return 'avas'
}

// T extends (...args: any[]) => infer R ? R : never
//  表示如果条件正确 返回 R
// infer R  ：表示在条件类型中 会推断（infer自动推断）继承的后面函数类型的返回值类型 然后给 R
// 可以将infer理解为占位符，当推断出来类型后，会将类型赋值给其后面跟着的符号

// 自定义获取函数返回值类型工具
type MyReturnType<T extends (...args: any[]) => any> = T extends (
  ...args: any[]
) => infer R
  ? R
  : never

// 获取一个函数的返回值类型
type CalcReturnType = MyReturnType<CalcFnType>
type FooReturnType = MyReturnType<typeof foo>


// 自定义获取函数参数类型工具
type MyParameterType<T extends (...args: any[]) => any> =T extends (...args: infer A) => any ? A :never  


type CalcFnType1 = (n1: number, n2: string) => any
type CalcParamType =  MyParameterType<CalcFnType1>
```

### 2.8.11.条件类型中的分发类型

**当在泛型中使用条件类型的时候，如果传入的是一个`联合类型`，就会变成`分发的(distributive)`**

```typescript
type toArray1<T> = T[]

// 返回的类型是(number|string)[]
type NumAndStrArr1 = toArray1<number | string>

// 分发条件类型
type toArray<T> = T extends any ? T[] : never

// 返回的类型是string[] | number[]
// 会将联合类型的每个类型进行一次调用toArray<>
type NumAndStrArr = toArray<number | string>
```

## 2.9.ts内置工具

### 2.9.1.Partial

* 复制类型将其所有成员变成可选

```typescript
interface IKun {
  name: string
  age: number
  slogan?: string
}

// Partial 复制类型将其所有成员变成可选 (内置工具)
type IKunOptional = Partial<IKun>

// 自己实现
type MyPartial<T> = {
  [P in keyof T]?: T[P]
}

export {}

```

### 2.9.2.Required

* 复制类型将其所有成员变成必选 

```typescript
interface IKun {
  name: string
  age: number
  slogan?: string
}

// Required 复制类型将其所有成员变成必选 (内置工具)
type IKunOptional = Required<IKun>

// 自己实现
type MyRequired<T> = {
  [P in keyof T]-?: T[P]
}
export {}

```

### 2.9.3.Readonly

* 复制类型将其所有成员变成只读

```typescript
interface IKun {
  name: string
  age: number
  slogan?: string
}

// Readonly 复制类型将其所有成员变成只读 (内置工具)
type IKunOptional = Readonly<IKun>

// 自己实现
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P]
}
export {}

```

### 2.9.4.Record

*  返回一个对象类型 对象类型的键是遍历K得到的  键的类型是T

```typescript
interface IKun {
  name: string
  age: number
  slogan?: string
}

type t1 = '上海' | '土耳其' | '徐庄'

// Record<K, T>  返回一个对象类型 对象类型的键是遍历K得到的  键的类型是T (内置工具)
type IKunOptional = MyRecord<t1, IKun>

// 自己实现
type keys = keyof IKun // 会返回一个由IKun所有key组成的联合类型  name|age|slogan
type res = keyof any // => string | number | symbol

type MyRecord<K extends keyof any, T> = {
  // 但是K必须满足条件 不是所有的类型都可以做对象的键  比如boolean类型就不可以
  [P in K]: T
}
export {}

```

### 2.9.5.Pick

* Pick<T, K> 从T中获取K属性  组成一个对象类型 

```typescript
interface IKun {
  name: string
  age: number
  slogan?: string
}

// Pick<T, K> 从T中获取K属性  组成一个对象类型 (内置工具)
type IKunOptional = MyPick<IKun, 'name'>
type IKunOptional = MyPick<IKun, 'name' | 'slogan'>

// 自己实现
type MyPick<T, K extends keyof T> = {
  // 遍历 K  然后从T中获取每个K 对应的值
  // 要限制K的值必须在T中都有
  [P in K]: T[P]
}

export {}

```

### 2.9.6.Omit

* Omit<T, K> 从T中去除K属性  组成一个对象类型 

```typescript
interface IKun {
  name: string
  age: number
  slogan?: string
}

// Omit<T, K> 从T中去除K属性  组成一个对象类型 (内置工具)
type IKunOptional = MyOmit<IKun, 'name'>

type t = 'name' extends keyof IKun ? never : false
// 自己实现
type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P]
}
```

### 2.9.7.Exclude

* Exclide<Union Type, ExcludedMembers>
* 从Union Type联合类型里排除了ExcludedMembers的类型

```typescript
type t1 = 'name' | 'age' | 'height'

// Exclude<T, K> 去除T中的K  (内置工具)
type IKunOptional = MyExclude<t1, 'name'>

// 自己实现
// 核心是内容分发
type MyExclude<T, U> = T extends U ? never : T
```

### 2.9.8.Extract

* Extract<T, U> 从T中提取U 

```typescript
type t1 = 'name' | 'age' | 'height'

// Extract<T, U> 从T中提取U (内置工具)
type IKunOptional = MyExtract<t1, 'name' | 'age'>

// 自己实现
// 核心是内容分发
type MyExtract<T, U> = T extends U ? T : never

export {}

```

### 2.9.9.MyNonNullable

* NonNullable<T> 去除T中的null或undefined

```typescript
type t1 = 'name' | 'age' | 'height' | null | undefined

// NonNullable<T> 去除T中的null或undefined (内置工具)
type IKunOptional = MyNonNullable<t1>

// 自己实现
// 核心是内容分发
type MyNonNullable<T> = T extends null | undefined ? never : T

export {}

```

### 2.9.10.required

```typescript

```

