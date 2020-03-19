小白的笔记，大神请绕路。记录一下我可以理解的一些模块的功能，有的模块的功能还用不到就不记录了（还有两个模块scanner，spider我的burpsuit里没有...寻思着找其他的工具代替一下）

## --- proxy

代理模块，相当于主机于服务器之间的一个中间人，可以拦截，修改，发送截获的数据包。

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/1.png?raw=true)

他下面有四个功能

### --- --- intercept

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/2.png?raw=true)

截断功能，这个部分可以查看截获的数据包的内容。

Forward的功能是放出截获的数据包

Dorp的作用是丢弃数据包

Intercept on/off 决定代理是否开启

Action 可以将截获的数据包交给其他模块处理（右键也可以）

### --- --- HTTP history

这个部分记录了代理截获的数据的相关日志

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/3.png?raw=true)

### --- --- websocket history 

和上面一个部分功能一样...

### --- --- Options

这个部分是关于proxy工作的一下参数配置，内容有点多一个一个记录一下

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/4.png?raw=true)

Proxy listener 是配置proxy拦截谁的请求的，比如ip填写127.0.0.1就去拦截本机和远程服务器之间的数据包，当然如果局域网的其他主机把装有burpsuit的机器设为代理，那么burpsuit就可以拦截其他机器的数据包

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/5.png?raw=true)

Intercept client request是用来筛选拦截客户端请求数据包的一些规则配置，and表示前面的规则不满足则不执行这一条规则，or是前面的一个规则满足则改规则也会被执行。后面的一些参数是一些匹配的规则（我也没有过...）

Intercept server response 和上面一样只是筛选规则的内容是服务端的响应

Intercept websockets messages 这个先默认开着吧（不太清楚其作用...）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/6.png?raw=true)

Response modification这个部分相当有用，可以对服务端的返回做一些修改，比如剔除js验证，把https转为http，删除cookie的secure flag，删除表单输入长度限制等功能...

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/7.png?raw=true)

当想要批量的修改数据包的内容时 match and replace 就很有用，他可以根据我们输入的规则去匹配相关的内容并完成替换

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/8.png?raw=true)

Ssl pass through burpsuit 通过ssl直接连接到目标服务器（不知道是干嘛的...）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/9.png?raw=true)

Miscellaneous 一些杂项配置

 

## 一个小插曲

当用proxy代理http请求时,那是一个行云流水，但是代理https时就会报错，这时就需要在浏览器里添加burpsuit的证书

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/10.png?raw=true)

推荐下这篇文章（自己试了一遍可以用，作者写的很好我就不去再写一遍了，重点是作者简述了代理https的原理，当然不同的浏览器证书的存放位置可能不同，需要自己找一下）：

https://www.cnblogs.com/lsdb/p/6824416.html

 

## 一个小栗子 --- 用burpsuit抓手机的数据流量包

先设置burpsuit代理的网卡和端口

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/exp1.png?raw=true)

然后再手机端设置一下代理就可以了

抓包结果（打开淘宝app）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/exp2.png?raw=true) 

 

## --- Target

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/11.png?raw=true)

Target 可以认为是和proxy一起工作的（我是这么认为的），因为proxy拦截的数据都会被target记录下来只是方式有些不同

### --- --- scope

这个是配置作用域的地方，就两个坑，上面的填写要接受的作用域的数据，下面的填写不接受的作用域数据

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/12.png?raw=true)

### --- --- site map 

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/13.png?raw=true)

Emmm...最左边就像一个站点的树形结构，中间显示的是具体的内容，最右边显示的是一个某些主机可能存在的漏洞

### --- --- issue definitions

这个部分提供了安全的漏洞的相关信息供我们参考，比如sql注入的简介，危险级别什么的...

 

## --- Decoder

这个模块是用来做一些简单的加解密的模块,使用方式也很简单，还有一个smart decode，但是我就没有smart decode出过东西...不太会用这个模块，还是用在线的加解密吧...

 

## --- intruder

这个模块是我个人比较喜欢的，做暴力破解很给力( ^ _^)

### --- --- target 

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/14.png?raw=true)

这是用来设置目标ip和端口的位置，我的靶场（DVWA）就在本机所以填写了127.0.0.1

### --- ---positions

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/15.png?raw=true)

用于设置要爆破的位置和爆破的方式

具体有以下四种方式

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/16.png?raw=true)

Sniper 用不同的payload（字典里的值）对一个点进行爆破

Battering ram 用相同的payload对多个位置进行爆破，比如，用户名和密码相同的情况

Pitchfork 使用多个payload对多个位置进行爆破，字典的组合方式有点奇怪...我也没怎么搞清楚

Cluster bomb 也是使用多个payload爆破多个位置，但字典的组合方式是笛卡尔积

### --- --- payloads

设置payload的相关参数的地方

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/17.png?raw=true)

Payload sets 设置字典的数目和加载方式

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/18.png?raw=true)

Paylaod processing 对生成的payload做一些处理，规则有用户决定，比如进行编码，加密，替换什么的

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/19.png?raw=true)

Payload encoding 对payload的一些字符进行url编码，安全传输

 

### --- ---Options

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/20.png?raw=true)

Requests header 就两个选项都勾上吧，第一个会选项会更新content-length header，保证完成一次正常的请求，第二个完成请求后会关闭连接，可以加快爆破的速度

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/21.png?raw=true)

Request engine 设置爆破的线程数，重传次数，重传时间，爆破的间隔（最好啊设置一下，避免目标察觉，或因为负载过大而挂掉），开始时间，可以是立刻爆破也可以，隔一段时间再爆破

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/22.png?raw=true)

Attack results 筛选显示攻击的结果，use denial-service mode(no results),是用于拒绝服务攻击的，他不会返回结果

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/23.png?raw=true)

grep-match 比配满足用户设置的规则的的响应内容，匹配方式可以是字符串或是正则表达式。

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/24.png?raw=true)

Grep-extract 提取响应里有用的内容

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/25.png?raw=true)

Grep-payload 我说不清楚这个选项的功能...看使用说明也是非常的懵逼，先放着吧

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/26.png?raw=true)

Redirections 这个模块再爆破的时候如何处理重定向

 

### 一个小栗子（拿DVWA试试）

Simple

抓包并交给intruder

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/exp3.png?raw=true)

设置爆破位置和模式（两个位置，爆破方式我选择pitchfork）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/exp4.png?raw=true)

设置payload

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/exp5.png?raw=true)

设置完成后开始爆破，并查看结果

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/exp6.png?raw=true)

 

## --- repeater

这个模块在我看来就是一个可以不断重放数据包的部分，再重放之前，可以对数据包的内容进行修改，修改完成后通过go按钮即可送，在reponse部分可以看到返回的内容。

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/burp_photos/27.png?raw=true)

 