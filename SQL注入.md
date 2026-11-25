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

## 语句快查表
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


#### 手工注入

## SQL注入学习要点
### (一) 掌握SQLmap的使用方法

#### 1 单引号 报错注入
`http://IP/Less-1/?id=1 ' or 1=1 --+`  (此处ip为靶场提供的内网地址)
利用单引号尝试回显
得到报错回显 `near ''1'' LIMIT 0,1' at line 1`
此处要为初学者解释报错回显的内容
1. 还原现场
原始 SQL 模板大概长这样

`SELECT * FROM users WHERE id='1' LIMIT 0,1`

因此输入后拼接得到的SQL语句为

`SELECT * FROM users WHERE id='1'` `' LIMIT 0,1`

                             └─┬─┘   └────┬────┘
                            你的输入    模板残留


### (二) 掌握手工注入的办法


## SQL数据库的补充知识
