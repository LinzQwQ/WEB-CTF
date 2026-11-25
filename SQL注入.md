# SQL注入
SQL注入是学习过程中第一个接触的漏洞，也是应用比较广泛的漏洞

## 原理
用户输入的数据被SQL解释器执行，通过猜测后台的SQL语句，进行恶意的SQL语句查询  

正确的sql语句一般为
select * from table where id = 'input'
select * from table where id = "input"
select * from table where id = ("input")
select * from table where id = ('input')
select * from table where id = ("'input'"")

## 学习要点
### 1 掌握sqlmap的使用方法


### 2 掌握手工注入的办法

