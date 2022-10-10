# 1.stringify

## 1.1. replace参数

* JSON.stringify的第二个参数

```js
      let obj = {
        name: "zs",
        age: 12,
      };

      let stringify = JSON.stringify(obj, (key, val) => {
        if (key === "name") return "sss";
        return val;
      });
      console.log(stringify);
      // {"name":"sss","age":12}
```

## 1.2. space参数

* 这个参数是JSON.stringify的第三个参数，
* 目的是打印出的json串便于阅读
* 值是数字，表示以几个数字进行缩进

```js
      let stringify = JSON.stringify(
        obj,
        (key, val) => {
          if (key === "name") return "sss";
          return val;
        },
        2
      );
      console.log(stringify);
    //  {
    //   "name": "sss",
    //   "age": 12
    // }
```

## 1.3.toJSON方法

* 如果序列化的对象有toJSON方法，
* 调用JSON.stringify的时候，直接调用的就是toJSON这个方法

```javascript
   // 利用这个特性我们可以自定义序列化的方法
   // 可以直接再原型中添加这个方法，也可以再当前对象实列中添加
	Object.prototype.toJSON = () => {
        return 123;
      };

      let obj = {
        name: "zs",
        age: 12,
      };

      let stringify = JSON.stringify(
        obj,
        (key, val) => {
          if (key === "name") return "sss";
          return val;
        },
        2
      );
      console.log(stringify);
      // 123
```

# 2.parse

# 2.1. 回调参数

* 这个参数是JSON.stringify的第二个参数

```javascript
      let obj = {
        name: "zs",
        age: 12,
      };

      let stringify = JSON.parse(obj, (key, val) => {
        if (key === "age") return val * 2;
        return val;
      });
      console.log(stringify);
      // {"name":"sss","age":24}
```

