# WEB-CTF
新手入门的复盘整理
//基于先辈指引与自身热爱，投身网安事业

## MOeCTF2024 垫刀之路 复盘

### 垫刀关卡1-7(2026-10-8:垫刀6与php反序列化相关，暂时跳过)

#### 垫刀1 RCE
##### 题目【网页中】
根据提示已经获取了目标权限，给了一个命令框
于是想到使用基于linux的命令行指令-> ls ，成功遍历当前目录下文件
于是尝试返回根目录-> ls / ，
发现flag文件-> cat /flag 提示查看环境变量
于是输入 env，在末尾找到flag

#### Image Cloud前置
##### 题目【url后面怎么有个?url=, 啧啧啧，这貌似是一个很经典的漏洞, flag在/etc/passwd里，嗯？这是一个什么文件声明：题目环境出不了网，无法访问http资源，但这并不影响做题，您可以拿着源码本地测试】

给出输入框让输入Image URL
试着输入一个x，跳转一个网页，观察url发现后缀出现?url=x
联系flag在路径/etc/passwd中，这是一个ssrf漏洞，用到文件协议（类似http协议）
于是通过?url=file:///etc/passwd,在末尾找到flag

#### ProveYourLove
##### 题目【都七夕了，怎么还是单身狗丫？快拿起勇气向你 crush 表白叭，300份才能证明你的爱！】

进入网页有几个输入框，300份数量庞大，于是想到使用bp的爆破(Intruder)模块，利用ai生成一个字典，值得提的是爆破的方法，在抓取数据包后导入爆破模块，选中要替换的'部分'点击添加payload，设置中导入字典，启动爆破，找到flag

#### ez_http
##### 题目(吐槽，对新生而言并非ez)【】
`1 Please use POST method`
打开F12 hackbar(注意是hackbar不是hack`er`bar)load加载 启动POST EXECUTE发包
`2 Please POST the parameter imoau=sb`
在POST的Body中输入imoau=sb,发包
`3 Please GET the parameter xt=大帅b`
在URL中输入?xt=大帅b，但是发现返回了400，观察发现该网页不支持utf-8(笔者建议，以后的题目也尽量使用URL编码，有些题目中可能会禁用诸如等号'='空格' '等字符)于是打开CyberChef将字符转换为url编码,输入发包
`4 The source must be https://www.xidian.edu.cn/`
在http中，由Referer记录你来时的网站，添加这个参数，发包
`5 Please set cookie:user=admin`
有手就行，发包
`6 Please use MoeDedicatedBrowser`
这里值得思考，http中的参数还有谁呢？ 观察发现使用......Browser user-agent参数就是用来记录你使用什么浏览器的，于是添加该参数,发包
`7 Local access only`
哇去，这是什么玩意,想到了LocalHost和X-Forwarded-For，输入127.0.0.1，找到flag

#### 垫刀2 一句话木马
##### 题目【映入眼帘的是一个文件上传的按钮。看来只要上传点木马什么的，就可以控制机器了吧。】

显然（bushi）可以想到上传一份一句话木马文件+中国蚁剑寻找flag
但是进入目录中没有找到，想到#垫刀1，于是启动了虚拟终端查找环境变量env
在倒数几行处找到了flag

#### 垫刀3 一句话木马+
##### 题目【为了保证服务器的安全，Sxrhhh 把文件上传的类型进行了限制，现在终于只能上传图片了。但是百密必有一疏，相信你能找到成功把你木马上传上去的方法的。】

把木马文件改为php后缀，放入网页中，打开F12，上传图片后在网络中找到刚刚的请求，编辑(更改png后缀为php)重新发送，重复垫刀2的方法获得旗子

#### 垫刀4 目录穿越
##### 题目【Sxrhhh 做了一个文件浏览器，塞了很多东西进去。不知道你能不能从这一堆乱七八糟的文件里面，翻出你想要的 flag 呢？注意：题目中有一些 readme 文件，我觉得你不应该错过。再注：本题与 jail-lv1 考点无关，只是致敬 （cue） 一下 flag 位置】

打开题目观察readme文件，似乎没有作用，但是在进入/src下目录时，观察到url出现了?path= ，猜测可以目录穿越(经验之谈？),尝试?path=/../../../../etc/passwd
找到了一些目录，同样方法观察bin文件，在排除bin，dev，etc这些'系统文件'后，在tmp下找到flag

#### 垫刀5 sql注入漏洞
##### 题目【这是一个登陆页面。听说管理员叫 admin123 ，而且只要登陆成功，就会显示 flag 。可是，听管理员自己说，它自己的密码在密码强度检查器网站上，需要上百年才能被破译。那么，我们应该怎么登陆进去呢？】

登录界面，首先考虑sql注入漏洞，尝试admin123' or 1=1# 成功登录找到flag

#### 垫刀6 php反序列化
##### 题目【】
//没时间学习php反序列化了，把代码交给ai获取一个payload先

#### 垫刀7 前后端结合，python中os库的使用
##### 题目【Sxrhhh 正在使用 Flask 编写网站服务器，不慎泄漏了 PIN 码, 是时候给他一个乱用调试模式的教训了。】

一个前后端结合的案例，观察网络请求发现，其与python密切相关，尝试利用webshell打开/console或/_debugger
输入/console,输入题目给的pin码，进入python后端控制台，这里笔者将补充一些os库的知识，以便于获取flag
##### 关os库
  import os       #必须的步骤
  WEB相关的重点用法先行列出
 #os.system("")  #在本题中尝试发现执行任何命令，只会返回一个0
  例如os.system("pwd")
       os.system("ls")
       os.system("whoami")
  让系统执行pwd，ls，查看user;相当于在py中让系统执行Linux命令行
 #os.popen("").read() #本题解法
  这相当于一个管道函数,把让系统执行cat flag后的返回值给到.read()并以python字符串形式返回

再给出一些其他用法，也有其用武之地
  os.listdir()    #列出当前目录^
  os.getcwd()     #查看当前目录^
  os.environ.get()#获取环境变量^
  os.chdir()      #切换目录  
  os.mkdir()      #新建目录
  os.rmdir()      #删除空目录，非空会报错
  os.path.exists()#判断当前路径下是否有目标文件

  通过os.listdir()发现了flag文件，并通过os.popen("cat flag").read()读取
  ^于是我们获得了flag
