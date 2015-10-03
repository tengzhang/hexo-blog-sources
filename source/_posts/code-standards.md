title: code-standards
date: 2014-07-19 22:42:52
tags:
---
# 后端
---
## 工程结构
分为`service`和`web`

## 注释
1. 类头要写明该类的职责是什么。
for example:
``` java
/**
 * 职责
 * @Description
 *
 * @author zhangteng
 * @email zhangt@pandawork.net
 * @time: 2014/7/9 14:25
 */
```
<!-- more -->

2. 方法要写明该方法的功能是什么。
``` java
/**
 * 获取分页列表
 *
 * @param curPage
 * @param code
 * @param name
 * @param startTime
 * @param endTime
 * @return
 */
```
3. 方法里面要有适当的注释。不能写了一个方法100行，一个注释都没有。
4. 数据库表要有一个总的声明，每个字段有相关说明。
5. 实体或者业务修改，也要修改相关数据表中表对应的字段说明。保持一致。

## 后端规范
### 类名称
1. 以大写字母开头。
2. 以驼峰方式命名。

### 属性名、方法名
1. 以小写字母开头。
2. 以驼峰方式命名。

### 常量命名规范
1. 所有字母大写
2. 单词之间用’_’分隔

### service,mapper方法名
方法名，根据不同的功能，按照不同的前缀进行设计。如下表：

| 功能说明 | 前缀 |
| -------- | ---- |
| 计算数量 | count|
| 获取列表 | list |
| 查询功能 | query |
| 创建     | add |
| 删除     | del |
| 修改     | update |

### controller命名规范
| 功能说明 | 前缀 |
| -------- | ---- |
| 计算数量 | count|
| 获取列表 | list |
| 查询功能 | query |
| 创建     | add |
| 删除     | del |
| 修改     | update |
如果是ajax，则需要在此前加上ajax。例如：
要通过ajax按照ID删除一个产品，方法名为ajaxDelProductById

### URL规范
#### URL结构
下面，我们以用户为例。
* CREATE
/user/new        GET              这个Url负责跳转到提交表单的页面，用Get方法
/user            POST       	  负责接收表单提交的数据，用POST方法
* DELETE
/user/{id}/del   GET              跳转到提交删除请求的页面，用GET方法（可选）
/user/{id}       DELETE           负责接收要删除的用户的数据，用DELETE方法
* UPDATE
/user/{id}/update     GET         跳转到用户修改的页面，用GET方法
/user/{id}       PUT              负责接收表单提交的数据，修改用户，用PUT方法
* READ
/user            GET              跳转到查看用户列表的页面，用GET方法
/user/{id}       GET              跳转到查看某个用户的页面，用GET方法
* OTHER
/user/{id}/lock  PUT       		 有些时候，为了表示某一动作对用户产生的变化，我们这样命名，比如用户的锁定
* 分页
/user/{curPage} GET              大家可以用这个来进行获取分页列表

### mapper层SQL语句书写规范
书写SQL语句时，SQL中的关键字全部采用大写，如下所示
``` sql
SELECT * FROM `t_quality_check_item`
```
SELECT、FROM、INSERT INTO、UPDATE、DELETE、AND、OR、LIKE、LIMIT、ORDER等关键字，在mapper中书写SQL语句时均采用大写。

### 后端开发注意事项
1. 职责单一化
要确保所写的程序职责单一，例如，有ProductService，和一个DiscountService，两者的区别在于一个是对产品的操作，另外一个是折扣的业务，在ProductService中，出现直接调去DiscountMapper的方法。
如上例，一个方法可以，多了，整个工程会出现轮乱局面，正确做法是走discountService进行相关的折扣计算。
2. 重复方法
此类问题最为严重，禁止在同一个Service中，或在mapper中出现多个名称不一样，但功能一样的方法。

## 数据库
### 数据库表名命名规则
1. 表名由前缀和实际名字组成，前缀和实际名字之间使用“_”连接。前缀使用“t”，实际名字都使用小写单词，多个单词之间使用“_”连接。例如：t_student_info
2. 如果要进行分表时，表的命名直接在表名的后面加上分表的标志。例如，t_user001,t_user002,t_user003
3. 表名的构成有t_模块名称_自己的表名。

### 字段命名规则
1. 每一个表都将有一个自动id作为主健,逻辑上的主健作为第一组候选主健来定义,如果是数据库自动生成的编码，统一命名为：id;如果是自定义的逻辑上的编码则用缩写 加“id”的方法命名。
2. 采用前缀命名。给每个表的列名都采用统一的前缀，后面的单词或则缩写都以大写字母开头

### 数据表的存储引擎和索引约定规则
1. 现在所有表的存储引擎都要选择InnoDB，如果需要设为其他存储引擎的时，要特别说明。
2. 在建立查询时候，务必要建立好数据库的索引，在确保查询语句使用正确高效可靠以后，养成马上建立对于索引的习惯。
3. 索引名称格式如下：
	* index\_索引类型\_表名\_按照索引顺序编写字段名，以_为分隔符。
	* index\_t\_books\_isbn（普通索引）
	* index\_unique\_t\_books\_isbn(构建唯一索引)  

索引的类型一般有三种类型，分别是普通索引（数据库中的index）, 唯一索引（数据库中的index unique),全文索引，这个基本上我们不会使用。

``` js
url:{
    index:{
        login:{
            check: '${website}index/login/check'
        }
    }
}

```