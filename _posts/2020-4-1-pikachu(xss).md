#### Cross-Site Scripting

**--- --- 反射型xss（get）**

这种方式的利用相对简单，只需让目标点击构造的url就可以了（JavaScript不太会，要补补课了……不过也可以利用大神们搭建的xss平台）

（测试思路，找到目标的输入接口，如留言板，反馈，评论...，先简单测试，目标是否对一些特殊字符做处理，查看源码里的返回可以判断……）

**--- --- 反射性xss（post）**

这种情况不再使用url提交，也就意味着不能直接使用url构造欺骗了。该怎么办呢？这时候我们可以搞一个页面，里面添加一些我们精心构造的代码（比如先获取用户的cookie，然后再重定向到一个可信任的网站……）

**--- --- 存储型xss**

这种类型的xss会永久的存储再服务端，当用户的操作触发了恶意代码（比如浏览一篇文章，点击某个按钮）攻击就完成了。通常可以在留言板，用户反馈这些地方去找突破点

**--- --- DOM型xss**

通过闭合js语言的标签来实现，如果js从url处读取数据（window.location.search），那么就可以构造url发给目标，完成相应的攻击。

**--- --- xss之盲打**

这是我们也不知究竟有没有xss漏洞，通过提交恶意代码带后台，如果存在漏洞，且管理员触发了恶意代码，那么可能就会受到意外惊喜(^_^)

**--- --- xss之过滤**

大小写，拼凑，添加注释标签（<!---->）,编码绕过（了解前端可以识别的编码，编码必须前端可以识别） ……

**--- --- htmlspecialchars**

该方法默认是不对单引号进行转义的，所以有可能通过闭合单引号的方式去构造playload

**--- --- js输出和href输出**

如果输出给了js，可以通过构造script标签的闭合来实现xss攻击，如果输出给了href，如果没有定义网络协议（http,https），那么可以通过javascrpit协议执行恶意代码 

#### ---小栗子

**--- --- 盗取用户cookie**

Xss站点（虚拟机）：192.168.147.145

受害者（虚拟机）： 192.168.147.144

攻击这（宿主机）： 192.168.1.2

（先进行联通性测试，保证各个机器之间可以正常的交互...确保初始化数据库，不然会出现许多莫名其妙的问题，我搞了好久...）

确保受害者机器可以正常访问目标站点（关于登陆的地方去登陆一下看看行不行，不然要出幺蛾子）

下面就以靶场上post的xss进行实验。首先受害者会有个登陆操作（这个过程也就获得了个cookie）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/2_2.png?raw=true)

我们在攻击上运行pkxss的后台来记录用户的cookie（也就是，平台最下面的管理工具）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/7.png?raw=true)

我们知道受害者登陆成功后的页面的输入框时存在xss漏洞的

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/3_2.png?raw=true)

顺便看看受害者获得cookie值是多少

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/4_2.png?raw=true)

攻击者只需把带有恶意代码的页面（攻击者的站点的地址，诱使得受害者点击...）发个受害者,如果受害者上钩也就获得了cookie（pkxss里帮我们准备了有个post.html的页面可以直接拿来使用，注意修改一下相应的IP，和文件的路径），假设受害者上当了访问了攻击者的站点

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/5_2.png?raw=true)

这时攻击者在后台就拿到了cookie，甚至还有用户名和密码（md5）欧（^_^）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/6.png?raw=true)

**--- --- 钓鱼攻击**

这个原理更简单，只需要做一个让受害者输入账号和密码的页面，诱使受害者上当就行了（虽然不太可能），pkxss也为我们写好了有个fish.php文件，访问这个文件会弹出有个输入账号和密码的认证框。如果受害者输入账号和密码就会被传到攻击者的后台（-_-?）,不过我实验的时候后台一直收不到数据，也不没找原因...（记录一下，日后再回头看看）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/7.png?raw=true)

**--- --- 键盘记录**

这里有一个重要的策略（同源策略），就是不同域的javascript代码不能互相对对方的资源进行操作，但是记录用户的键盘输入需要用到javascript的代码，不用着急(^.^)，有的属性是可以无视同源策略的比如src...再说了攻击者可以关闭同源策略的限制，设置站点运行跨域请求。

额...可以将再存储型的xss里利用这个，比如插入恶意代码`<script>恶意js地址</script>`，这样就可以一直记录受害者数据了（虽然也不太可能...）