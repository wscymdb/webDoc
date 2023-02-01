# 1.云数据库

## 1.1.查询数据

### 1.1.1.通过ID查询精确某一条数据

使用doc查询ID

```javascript
// 1.获取数据库
const db = wx.cloud.database()
// 2. 获取集合
const wzrzCol = db.collection("wzry")

wzrzCol.doc("d26f8b7363aba034001b3ceb2a7a784d").get().then(res => {
    console.log(res);
})
```



### 1.1.2.根据条件查询满足条件的某些数据

使用where语句

```javascript
wzrzCol.where({
    nickname: "李庚希gl"
}).get().then(res => {
    console.log(res.data);
})
```



### 1.1.3.通过指令过滤数据

使用db.command的指令

```javascript
const cmd = db.command
wzrzCol.where({
    rid: cmd.lte(500215)
}).get().then(res => {
    console.log(res.data);
})
```



### 1.1.4.通过正则表达式匹配数据

使用db.RegExp创建正则规则

```javascript
wzrzCol.where({
    nickname: db.RegExp({
        regexp: "李"
    })
}).get().then(res => {
    console.log(res.data);
})
```



### 1.1.5.获取整个集合的数据

直接调用get

小程序中单次只能查询20条数据，云函数中单次可以查询100条

```javascript
wzrzCol.get().then(res => {
    console.log(res.data);
})
```



### 1.1.6.过滤、分页、排序查询数据

使用field、skip、limit、orderBy

```javascript
// 分页查询 skip(等于offset)  limit(等于size)
// field 表示查出来的时候显示哪些字段
wzrzCol.field({
    rid: true
}).skip(0).limit(9).orderBy("rid", "desc").get().then(res => {
    console.log(res);
})
```



