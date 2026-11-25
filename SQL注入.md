# SQL注入
SQL注入是学习过程中第一个接触的漏洞，也是应用比较广泛的漏洞

本笔记基于厦门大学网络安全实训靶场提供的相关工具和靶场进行学习

在笔记的最后会补充一些关于SQL数据库的基础知识

## 原理
用户输入的数据被SQL解释器执行，通过猜测后台的SQL语句，进行恶意的SQL语句查询

正确的sql语句一般为

select * from table where id = 'input'

select * from table where id = "input"

select * from table where id = ("input")

select * from table where id = ('input')

select * from table where id = ("'input'"")


## SQL注入学习要点
### 一 掌握sqlmap的使用方法
#### 1 单引号 报错注入

### 二 掌握手工注入的办法


## SQL数据库的补充知识
