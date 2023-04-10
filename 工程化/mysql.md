# 1.连接方式

## 1.1.命令方式

```sql
# 输入以下命令
mysql uroot -p
# 之后出入密码即可进入
```

# 2.mysql默认数据库

* MySQL一共有**4个默认数据库**
* `information_schema:`信息数据库，其中包括MySQL在维护的其他数据库、表、列、访问权限等信息
* `performance_schema:`性能数据库，记录着MySQL_Server数据库引擎在运行过程中的一些资源消耗相关的信息
* `mysql:`用于存储数据库管理者的用户信息、权限信息以及一些日志信息等
* `sys:`相当于一个简易版的performance_schema，将性能数据库中的数据汇总成更容易理解的形势

# 3.SQL语句

## 3.1.SQL语句常用的规范

* 通常关键字使用`大写`
* 一条语句结束后需要以`;`结尾
* 如果使用关键字命名表，需要使用`反引号`包裹

## 3.2.SQL语句的分类

**常见的SQL语句可以分成四类**



* **DDL(Data Definition  Language):**数据定义语句

  * 可以通过DDL语句对数据库或者表进行：创建、删除、修改等操作

* **DML（Data Manipulation Language）:**数据操作语言

  * 可以通过DML语句对`表中的数据`进行：添加、删除、修改等操作

* **DQL（Data Query Language）:**数据查询语言

  * 可以通过DQL从数据库中查询记录

* **DCL（Data Control Language）：**数据控制语言

  * 对数据库、表格的权限进行相关访问控制操作

  

## 3.3.DDL

### 3.3.1.数据库操作

#### 3.3.1.1.**查看**

```sql
# 查看所有数据库
SHOW DATABASES;

# 使用某个数据库
USE xxx;

# 查看当前正在使用的数据库
SELECT DATABASE()
```

#### 3.3.1.2.**创建**

* `mysql中如果创建已存在的数据库会报错`

```sql
# 创建数据库
CREATE DATABASE xxx;

# 创建xxx数据库如果不存在的话
CREATE DATABASE IF NOT EXISTS xxx;

# 创建数据库并设置编码格式（一般使用默认即可）
CREATE DATABASE IF NOT  EXISTS xxx
			 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```

#### 3.3.1.3.删除

* `如果删除不存在的数据库也会报错`

```sql
# 删除数据库
DROP DATABASE xxx;
# 删除数据库如果存在的话
DROP DATABASE IF EXISTS xxx;
```

#### 3.3.1.4.修改

```sql
# 修改数据库的字符集和排序规则 一般不用修改
ALTER DATABASE xxx CHARACTER SET = utf8 COLLATE = utf8_unicode_ci
```

### 3.3.2.表操作

#### 3.3.2.1.**查看**

```sql
# 查看当前数据库中的所有表
SHOW TABLES;

# 查看表结构
DESC xxx;

```

#### 3.3.2.2.**创建**

* `mysql中如果创建已存在的表会报错`

```sql
# 创建表
CREATE TABLE xxx (字段1 类型 列约束1 列约束n ... , 字段2 类型 列约束, ...);
# eg:  CREATE TABLE xxx (name varchar(10) UNIQUE NOT NULL , age int NOT NULL, ...);

# 创建表如果不存在的话
CREATE TABLE IF NOT EXISTS xxx (字段1 类型 列约束, 字段2 类型  列约束, ...);
```

#### 3.3.2.3.删除

* `如果删除不存在的表也会报错`

```sql
# 删除表如果存在的话
DROP TABLE IF EXISTS xxx;
```

#### 3.3.2.4.修改

* 如果是非关键字的比如表名、字段名可以用``包裹，方便区分

```sql
# 修改表名
ALTER TABLE xxx RENAME TO xxx;
# eg: ALTER TABLE `user1` RENAME `user`;

# 添加一个新的列
ALTER TABLE xxx ADD 列名 列类型 列约束(可选);
# eg: ALTER TABLE `user` ADD `score` TINYINT NOT NULL;

# 删除列
ALTER TABLE xxx DROP 列名;
# eg: ALTER TABLE `user` DROP `score`;

# 修改列名称
ALTER TABLE xxx CHANGE 原列名 新列名 新列类型 新列约束(可选);
# eg: ALTER TABLE `user` CHANGE `score` `score1` INT NOT NULL;

# 修改列的数据类型(约束)
ALTER TABLE xxx MODIFY 列名 列类型 列约束(可选);
#eg: ALTER TABLE `user` MODIFY `score1` BIGINT NOT NULL;
```

## 3.4.SQL的数据类型

* MySQL支持的数据类型有：`数字类型`、`日期和时间类型`、`字符串`（字节和字符）、空间类型（坐标系（x,y,z））、JSON类型
* Boolean类型一般使用数字类型进行存储

### 3.4.1.数字类型

* MySQL支持的数字类似有很多
* **整数数字类型**：`INT、SMALLINT、TINYINT、MEDIUMINT、BIGINT`
* **浮点数字类型**：`FLOAT、DOUBLE`（FLOAT是4个字节、DOUBLE是8个字节）
* **精确数字类型**：`DECIMAL、NUERIC`（DECIMAL是NUMERIC的实现形式）
  * 所以开发中用DECIMAL表示这个数字的小数位有几位
* 有符号表示 可以有复数，无符号表示只能是正数

| Type      | 字节 | 最小值(有符号) | 最大值(有符号) | 最小值(无符号) | 最大值(无符号) |
| --------- | ---- | -------------- | -------------- | -------------- | -------------- |
| TINYINT   | 1    | -128           | 127            | 0              | 255            |
| SAMLLINT  | 2    | -32768         | 32767          | 0              | 65535          |
| MEDIUMINT | 3    | -8388608       | 8388607        | 0              | 16777215       |
| INT       | 4    | -2147483648    | 2147483647     | 0              | 4294967295     |
| BIGINT    | 8    | -2的63次方     | 2的63次方-1    | 0              | 2的64次方-1    |

### 3.4.2.日期类型

* **YEAR类型**：以YYYY格式显示值
  * 范围1901到2155。和0000
* **DATE类型**：用于有日期部分但是没有时间部分的值
  * DATE以格式YYYY- MM- DD显示值
  * 支持的范围是1000-01-01到9999-12-31
* **DATETIME类型**：用于包含日期和时间部分的值
  * DATETIME格式 `YYYY-MM-DD       hh:mm:ss`
  * 支持的范围是1000-01-01 00:00:00到9999-12-31 23:59:59
* **TIMESTAMP类型**：用于包含日期和时间部分的值（`开发中一般用这种`）
  * TIMESTAMP格式 `YYYY-MM-DD       hh:mm:ss`
  * 范围是UTC的时间范围：1970-01-01 00:00:01 到 2038-01-19 03:14:07

### 3.4.3.字符串类型

* **CHAR类型**：定长字符串
  * 长度是0-255个字符
  * 在查询时，如果结果后面有空格，空格将会被删除
  * eg:char(10)
    * 如果存入的是‘你我他’，那么还剩下7个字符没用，mysql会补全剩下的字符（用空值补全）
* **VARCHAR类型**：变长字符串
  * 长度是0到65535个字符
  * 在查询时，如果结果后面有空格，不会删除
* **TEXT类型**：用于存储大的字符串类型
* BINARY和VARBINARY
  * 用于存储二进制字符串，存储的是字节字符串
* BLOB类型：用于存储大的二进制类型

## 3.5.表约束

### 3.5.1.主键 PRIMARY KEY

* 一张表中，为了`区分每一条记录的唯一性`，必须有一个字段是永远不会重复，并且`不会位空的`，这个字段通常将他设为`主键`
* 主键是表中**唯一的索引**
* 并且必须是NOT NULL的，如果`没有设置NOT NULL`，那么MySQL也`会隐式`的`设置为NOT NULL`
* 主键也可以是多列索引，PRIMARY KEY（key_pary,...），一般称为`联合主键`(了解)
* 建议：开发中主键字段应该是和业务无关的，尽量不要使用业务字段来作为主键

### 3.5.2.唯一约束 UNIQUE

* `某些字段`在开发中`希望是唯一的`，`不会重复的`，比如手机号码，身份证，这些字段可以使用UNIQUE来进行约束
* `使用UNIQUE约束的字段在表中必须是不同的（不能重复）`
* UNIQUE索引`允许多个NULL存在`
  * 比如name设置了UNIQUE约束，那么name这一列可以有多个null存在

### 3.5.3.非空约束 NOT NULL

* 用此约束的字段不能为空

### 3.5.4.默认值 DEFAULT

* 可以给某个列设置默认值，插入数据时，如果此列没有内容就会用默认值填充

* ```sql
  CREATE TABLE
      user (
          id INT PRIMARY KEY AUTO_INCREMENT,
          level INT DEFAULT 0,
      );
  ```

* 

### 3.5.5. 自动递增 AUTO_INCREMENT

* 某些字段我们不希望设置值时可以进行递增，比如用户的id，
* 默认情况下初始值为1

### 3.5.6.外键约束

## 3.6.DML

### 3.6.1.插入

```sql
# 单个插入数据
INSERT INTO xxx VALUES (value1,value2...);
# eg: INSERT INTO `student` VALUES (null, 'jack', 18, 69); 

# 多个插入
INSERT INTO xxx VALUES (value1,value2...),(value1,value2...),(value1,value2...)...;

# 对某个字段进行插入
INSERT INTO xxx(字段1,字段2) VALUES (value1,value2);
# eg: INSERT INTO `student`(name) VALUES ('jack'); 
```

### 3.6.2.修改

```sql
# 修改全部数据
UPDATE xxx SET 修改内容;
# eg: UPDATE `student` SET score = 100;

# 按条件进行修改
UPDATE xxx SET 修改内容 WHERE 修改条件;
# eg: UPDATE `student` SET score = 100 WHERE id = 2;

# 修改内容后显示修改的时间  
# 本质：添加一个updateTime字段 设置默认值是当前时间 并且当当前表有字段更新自动更新updateTime字段的时间
ALTER TABLE `student`
ADD
    `update_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;
```

### 3.6.3.删除

```sql
# 删除全部数据
DELETE FROM xxx;

# 按条件删除
DELETE FROM xxx WHERE 删除条件;
# eg: DELETE FROM `student` WHERE id=1;
```

## 3.7.DQL

### 3.7.1.基本查询

```sql

-- 查询所有数据的所有字段
SELECT * FROM product;

-- 查询所有数据的制定字段
SELECT brand,score,price FROM product;

-- 查询字段后给字段重命名 使用as关键字 （as关键字可以省略）
SELECT brand as '手机品牌',price '手机价格' FROM product;
```

### 3.7.2.条件查询

**WHERE的比较运算符**

* `>、<、>=、=<、=、!=`

```sql
-- 查询价格小于1000的手机
SELECT * FROM product WHERE price < 1000;

-- 查询价格不等于3399的手机
SELECT * FROM product WHERE price != 3399;

-- 查询手机品牌是华为的手机
SELECT * FROM product WHERE brand = '华为';

-- 查询手机品牌是华为的手机
SELECT * FROM product WHERE brand != '华为';
```

**WHERE的逻辑运算符**

* `&&、||、AND、BETWEEN...AND...、IN`
* BETWEEN a AND b `包括a和`

```sql
-- 查询品牌是华为并且价格小于2000的手机
SELECT * FROM product WHERE brand = '华为' AND price <= 2000;
SELECT * FROM product WHERE brand = '华为' && price <= 2000;

-- 查询手机品牌是华为或价格小于2000的手机
SELECT * FROM product WHERE brand = '华为' || price < 2000;
SELECT * FROM product WHERE brand = '华为' OR price < 2000;

-- 查询价格是1000-2000（包含1000和2000）之间的手机
SELECT * FROM product WHERE price>=1000 && price <= 2000;
SELECT * FROM product WHERE price BETWEEN 1000 AND 2000;

-- 查看多个结果中的一个
SELECT * FROM product WHERE brand in('华为','苹果');
```

**模糊查询**

* 模糊查询使用的是`LIKE`关键字，结合两个符号
  * `%`表示匹配`任意个`的`任意字符`
  * `_`表示匹配`一个`的`任意字符`

```sql
-- 查询所有以v开头的title （v%表示 v后面可以跟任意多个的任意字符）
SELECT * FROM product WHERE title LIKE 'v%';

-- 查询所有title中都带有a的手机 ('%a%:表示a前面可以跟任意多个任意字符，a后面也可以跟任意多个任意字符)
SELECT * FROM product WHERE title LIKE '%a%';

-- 查询title中的第三个字符是M的所有手机 (__M%表示 M前面两个任意字符，M后面可以有任意多个任意字符)
SELECT * FROM product WHERE title LIKE '__M%';
```

### 3.7.3.查询结果排序

* 查询结果排序借助`ORDER BY`关键字，有两个常用值
* `ASC`：升序（ascending）
* `DESC`：降序（descending）

```sql
-- 查询所有价格小于1000的手机，并且按照手机的评分降序排列
SELECT * FROM product WHERE price < 1000 ORDER BY score DESC;
```

### 3.7.4.分页查询

* 分页查询借助`LIMIT、OFFSET`关键字
* LIMIT：查询量
* OFFSET：偏移量

```sql
-- 分页查询（查询5条） 不偏移 从第一条数据查询5条数据出来
SELECT * FROM product LIMIT 5;

-- 分页查询（查询5条） 偏移1条数据 会从第二条数据开始查询5条数据出来
SELECT * FROM product LIMIT 5 OFFSET 1;

-- 分页查询的另一种写法 LIMIT 1,5  （1表示偏移5表示查询几条）
SELECT * FROM product LIMIT 1,5;
```

## 3.8.常见的聚合函数

* 聚合函数的本质其实就是，`先将查询的结果放在一个集合`，然后`用函数对这个集合进行操作`
* **注意**：`对于不知道有多少结果 或者没有直接结果的查询` 不可以`查询多个字段(mysql5以上会报错)
  * 比如求所有手机价格的平均值，还要查看哪一款手机 这是一个错误的逻辑
  * 再比如 查看评分最高的分数，但是还想知道是哪一款手机 SELECT AVG(price),title xxx; 这样是不对的因为你不确定评分最高的有几个，可以用`子查询`来查哪一款手机
* **AVG()、MAX()、MIN()、SUM()、COUNT()**

```sql

-- 计算华为手机的平均价格

-- 查询的所有价格的评价值
SELECT AVG(price) FROM product WHERE brand = '华为';

-- 计算小米手机的平均分数
SELECT AVG(score) '小米手机avg' FROM product WHERE brand ='小米';

-- 选择手机中评分最高的一款的手机
SELECT MAX(score) FROM product;

-- 选择手机中评分最低的一款的手机
SELECT MIN(score) FROM product;

-- 计算所有的手机一共多少价格
SELECT SUM(price) FROM product;

-- 计算一共有多少个商品 COUNT() 参数可以是字段 但是如果某个字段的数据是null那么获取的数量就会不准确，一般用*表示 所有的都会被计算
SELECT COUNT(*) FROM product;
```

## 3.9.Group By

* 事实上聚合函数相当于`默认将所有的数据分成了一组`
  * 上面的案例中使用avg还是max等，都是将所有的结果看出是一组来计算的
  * 那么`如果希望划分多个组`：比如苹果、华为、小米等手机分别的平均价格怎么办呢
  * 这个时候`可以用GROUP BY`
* **GROUP BY通常和聚合函数一起使用**
  * 表示我们对数据`先进行分组`，`再`对每一组数据`进行聚合函数的计算`

 

```sql
-- group by 对品牌进行分组，然后求出每个品牌的平均价格、最高、最低价格
-- ROUND() 四舍五入函数 2表示保留两位小数
SELECT
    brand,MAX(price),MIN(price),ROUND(AVG(price), 2) 
FROM `product` GROUP BY brand;
```

### 3.9.1Group By的约束条件

* 如果希望给GROUP BY查询的结果添加一些约束，可以使用`HAVING`关键字

```sql
-- group by 对品牌进行分组，然后求出最高价格小于8000的每个品牌的平均价格、最高、最低价格
-- HAVING后面的添加 字段可以使用起的别名(maxPrice)也可以使用原始的(MAX(price))

SELECT
    brand,
    MAX(price) maxPrice,
    MIN(price) minPrice,
    ROUND(AVG(price), 2) avgPrice
FROM `product`
GROUP BY brand
HAVING maxPrice < 8000;
```

## 3.10.多表相关

### 3.10.1多表的应用

* 现在有一张歌曲表，这个表存放着歌曲名称、歌手、歌手介绍等等
* 每个歌手不止有一首音乐，所以这个表中有很多的周杰伦的音乐
* 那么每首周杰伦的音乐都有一个他的介绍，这样就造成了数据的冗余，浪费空间
* 所以可以再创建一张歌手表，里面存放着歌手的名称、介绍等等
* 那么歌曲表中就不用存放歌手相关的东西（名称、介绍等）
* 歌曲表只用存放歌手对应的id（歌手表会有一个id字段）即可

### 3.10.2.外键

* 上面说了多表的应用

* 那么在歌曲表中歌手的id可以随便添加吗，比如歌手表中只有id是1和id是2的两个id

* 那么在歌曲表中的歌手id肯定要符合歌手表的ID，不然查询的时候查不到了

* 这时候就可以使用`外键`对歌曲表的歌手id做约束了

  * 当然也可以不使用 那么表中的数据量一大就会很混乱，难以维护

    

* 结合以上的案例总结一下：`外键约束就是一张表的某个字段与另一张表的某个字段有关联，为了让这张表的某个字段符合另一张表的某个字段的范围，而对这张表的进行约束`

#### 3.10.2.1.创建

* **一张表的字段被另一张表的字段约束，那么该表称为从表(子表)，另一张表称为主表(父表)**
* **初次创建表时候添加外键方式**：(外键名可选属性)
  * `[CONSTRAINT  外键名称 ] FOREIGN KEY (子表字段) REFERENCES 父表名 (父表字段)`
* **表已经存在了，额外添加外键方式**：
  * `ALTER TABLE 子表名 ADD [CONSTRAINT  外键名称 ] FOREIGN KEY(子表字段) REFERENCES 父表名(父表字段);`

```sql
-- 创建表的时候添加外键
CREATE TABLE
    `t_songs`(
        singer_id INT,
        FOREIGN KEY (singer_id) REFERENCES `t_singer`(id)
    );
-- 在已有表中添加外键
ALTER TABLE t_songs ADD FOREIGN KEY(singer_id) REFERENCES t_singer(id);

-- 创建带外键名称的外键
CREATE TABLE
    `t_songs`(
        singer_id INT,
        CONSTRAINT `外键名称` FOREIGN KEY(singer_id) REFERENCES `t_singer`(id);
    );
```

#### 3.10.2.2.删除

* **外键不能被修改，只能删除后重新创建**

* **外键名!==字段名**  

```sql
-- 通过上面两种方式创建的外键是没有名称的，创建的时候系统会默认给一个名字

-- 可以使用查看默认名称
SHOW CREATE TABLE 表名;

-- 删除外键
ALTER TABLE DROP FOREIGN KEY 外键名;
```



### 3.10.3.**外键存在时的操作**

* `其实外键不仅约束了子表同时也约束了父表`
  * 不可对父表中外键引用的字段进行删除、更新
  * 原因是当更新或删除某个记录时，会触发 on delete 或 on update来检查该记录是否有关联的外键记录
* 如果确实想要操作父表的被引用字段的数据，需要修改on delete或on update的值
* 有以下几个值
  * `RESTRICT(默认属性)`：当更新或删除某个记录时，会检查该记录是否有关联的外键记录，有的话就报错，不允许更新或删除；（mysql中定义的标准）
  * `NO ACTION`：和RESTRICT是一致的，是在SQL标准中定义的
  * `CASCADE`：当更新或删除某个记录时，会检查该记录是否有关联的记录外键，有的话
    * 更新：会更新对应的记录
    * 删除：那么关联的记录会被一起删掉
  * `SET NULL`：当更新或删除某个记录时，会检查该记录是否有关联的记录外键，有的话 将对应的值设置为NULL
* **每个字段再创建的时候都可以设置on update和on delete**

```sql
-- 如果是初次创建表的话
CREATE TABLE
    `t_songs`(
        singer_id INT,
        FOREIGN KEY (singer_id) REFERENCES `t_singer`(id) ON UPDATE CASCADE ON DELETE CASCADE
    );
    
-- 如果是表已经存在的话 需要先删除表，因为mysql中不允许直接修改外键，在添加外键并设置更新时的操作
-- 删除外键
ALTER TABLE t_songs DROP FOREIGN KEY t_songs_ibfk_1;

-- 添加外键 并设置 on update 和on delete时的操作
ALTER TABLE t_songs
ADD
    FOREIGN KEY(singer_id) REFERENCES t_singer(id) ON UPDATE CASCADE ON DELETE CASCADE;
```

### 3.10.4.多表查询

**默认查询结果**

* 如果直接查询两张表，数据会有很多没有的数据

* 因为默认多表查询会用第一张表格的每一条数据与第二张表格的全部数据匹配一次

* 这个结果称之为`笛卡尔乘积`，也称为`直积`，表示位`X*Y`

* ```sql
  SELECT * FROM t_songs, t_singer;
  ```

**用where条件筛选**

* 那么如果查询完毕使用where条件进行筛选呢

* 我们会发现查询的结果并不正确

* 这是因为不能确保第一张表的每条数据的外键（可能为null）都能与第二张表中匹配

* ```sql
  SELECT * FROM t_songs, t_singer WHERE t_songs.singer_id = t_singer.id;
  ```

#### 3.10.4.1.多表之间的连接

* 上面两种方式的效果并不是我们想要的效果，这时候可以使用SQL JOIN(连接)操作
* **左连接、右连接、内连接、全连接**



**左连接**

* 如果我们希望`获取到的是左边的所有数据（以左表为准）`
* 这个时候就表示无论左边的表是否有对应的singer_id的值对应右边表的id，左边的数据都会被查询出来
* 这个也是开发中使用最多的情况
* **LEFT [OUTER] JOIN  OUTER可以省略**

```sql
-- 左连接
SELECT * FROM 左表 LEFT JOIN 右表 ON 连接条件;
-- eg:SELECT * FROM t_songs LEFT JOIN t_singer ON t_songs.singer_id = t_singer.id;
```

* 还有两种左连接的查询（用的不多）

```sql
-- 左连接,查询左边和右边没有交集的数据
SELECT * FROM t_songs LEFT JOIN t_singer ON t_songs.singer_id = t_singer.id WHERE t_singer.id IS NULL;

-- 左连接,查询左边和右边有交集的数据
SELECT * FROM t_songs LEFT JOIN t_singer ON t_songs.singer_id = t_singer.id WHERE t_singer.id IS NOT NULL;
```

**右连接**

* 如果我们希望`获取到的是右边的所有数据（以右表为准）`
* 这个时候就表示无论右边的表是否有对应的id的值对应左边表的singer_id，右边的数据都会被查询出来
* **RIGHT [OUTER] JOIN  OUTER可以省略**

```sql
-- 右连接
SELECT * FROM 左表 RIGHT JOIN 右表 ON 连接条件;
-- eg:SELECT * FROM t_songs RIGHT JOIN t_singer ON t_songs.singer_id = t_singer.id;
```

* 还有两种右连接的查询（用的不多）

```sql
-- 右连接,查询左边和右边没有交集的数据
SELECT * FROM t_songs RIGHT JOIN t_singer ON t_songs.singer_id = t_singer.id WHERE t_songs.singer_id IS NULL;

-- 右连接,查询左边和右边有交集的数据 where之后的条件不一定是下面案例的 灵活应变即可
SELECT * FROM t_songs RIGHT JOIN t_singer ON t_songs.singer_id = t_singer.id WHERE t_songs.singer_id IS NOT NULL;
```

**内连接**

* 内连接表示左边的表和右边的表都有对应的数据关联
* **写法：CORSS JOIN 或者 INNER JOIN 或者 JOIN 都行**

```sql
-- 内连接
SELECT * FROM t_songs JOIN t_singer ON t_songs.singer_id = t_singer.id;
```

**全连接**

* 虽然SQL规范中有全连接但是MySQL中没有实现全连接
* MySQL需要借助`UNION`来实现全连接
* 本质上就是让左连接和右连接联合在一起

```sql
-- 全连接
(SELECT * FROM t_songs LEFT JOIN t_singer ON t_songs.singer_id = t_singer.id) UNION (SELECT * FROM t_songs RIGHT JOIN t_singer ON t_songs.singer_id = t_singer.id)
```

### 3.10.5.多对多表的查询

* 现有学生表student和选课表course
* 每个学生可以选择多个课程，每个课程也可以被多个学生所选择
* 基于这种情况，通常会`创建一个关系表来存放学生选课和课程的关系`
* **那么查询的时候就会用到多对对表的查询了**

**表的创建**

```sql
-- 创建学生表
CREATE TABLE
    IF NOT EXISTS student(
        id INT PRIMARY KEY AUTO_INCREMENT,
        name VARCHAR(15) NOT NULL,
        age INT DEFAULT 18,
        update_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    );

INSERT INTO student(name) VALUES ('小明'), ('小红'), ('小张'), ('小丽'), ('小李'), ('小赵');

-- 创建课程表
CREATE TABLE
    IF NOT EXISTS course(
        id INT PRIMARY KEY AUTO_INCREMENT,
        name VARCHAR(20) NOT NULL,
        score DOUBLE NOT NULL,
        update_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    );

INSERT INTO
    course(name, score)
VALUES ('篮球', 2), ('足球', 2), ('排球', 1), ('太极', 1), ('羽毛球', 2), ('游泳', 1);

-- 创建学生选择课程的关系表
CREATE TABLE
    IF NOT EXISTS student_select_course(
        id INT PRIMARY KEY AUTO_INCREMENT,
        student_id INT NOT NULL,
        course_id INT NOT NULL,
        Foreign Key (student_id) REFERENCES student(id) ON UPDATE CASCADE ON DELETE CASCADE,
        Foreign Key (course_id) REFERENCES course(id) ON UPDATE CASCADE ON DELETE CASCADE
    );

-- 选课的过程
INSERT INTO
    student_select_course(student_id, course_id)
VALUES (1, 1), (1, 3), (3, 2), (3, 3), (3, 4);
```

**查询所有的学生的课程**

```sql
-- 查询所有选课的学生选择的所有课程--内连接 (只查询了有选课的学生和选课的对应关系，内连接的特点)
-- 下面用别名的方式 简化代码
SELECT
    stu.name stuName,
    stu.age stuAge,
    cs.name couName,
    cs.score csScore
FROM student AS stu
    JOIN student_select_course AS ssc ON stu.id = ssc.student_id
    JOIN course AS cs ON cs.id = ssc.course_id;

-- 查询所有的学生选择的所有课程--左连接 (即使没有选课的学生也会被查询)
SELECT
    stu.name stuName,
    stu.age stuAge,
    cs.name couName,
    cs.score csScore
FROM student AS stu
    LEFT JOIN student_select_course AS ssc ON stu.id = ssc.student_id
    LEFT JOIN course AS cs ON cs.id = ssc.course_id;
```

**查询单个学生的课程**

* 查询的时候`最好用左连接`，因为查询的是学生选课情况，即使有的学生没有选课也会被查出来

```sql
-- 查询小明(id是1)选择了哪些课程
SELECT
    stu.name stuName,
    stu.age stuAge,
    cs.name courseName,
    cs.score coureScore
FROM student stu
    JOIN student_select_course ssc ON stu.id = ssc.student_id
    JOIN course cs ON cs.id = ssc.course_id
WHERE stu.id = 1;

-- 查询小赵(id是6)选择了哪些课程，这里必须用左连接，小赵没有选课，内连接查不出来(内连接的特性决定)
SELECT
    stu.name stuName,
    stu.age stuAge,
    cs.name courseName,
    cs.score coureScore
FROM student stu
    LEFT JOIN student_select_course ssc ON stu.id = ssc.student_id
    LEFT JOIN course cs ON cs.id = ssc.course_id
WHERE stu.id = 6;
```

**查询哪些学生和哪些课程没被选择**

```sql
-- 查询哪些学生没有选择课程
-- 使用的是左连接 那么student一定都会被查出来，那么只要判断course.id是null就带表这些学生一定没有选择课程

SELECT
    stu.name stuName,
    stu.age stuAge,
    cs.name courseName,
    cs.score coureScore
FROM student stu
    LEFT JOIN student_select_course ssc ON stu.id = ssc.student_id
    LEFT JOIN course cs ON cs.id = ssc.course_id
WHERE cs.id IS NULL;

-- 查询哪些课程没有被选过
-- 因为查询的主体是课程，那么要确保课程都被查出来，这里使用右连接，所以判断student.id是null(课程没有对应的学生)就代表这些课程没有被选择过

SELECT
    stu.name stuName,
    stu.age stuAge,
    cs.name courseName,
    cs.score coureScore
FROM student stu
    RIGHT JOIN student_select_course ssc ON stu.id = ssc.student_id
    RIGHT JOIN course cs ON cs.id = ssc.course_id
WHERE stu.id IS NULL;
```

