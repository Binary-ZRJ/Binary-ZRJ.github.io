#### CSRF

**--- --- CSRF（get）**

这种情况就是程序没有对用户的敏感操作（修改密码...）做验证造成的，同时通过get方式提交修改请求，这样就很容易造成CSRF攻击（当然也得用户登陆在目标站点上才行，因为我们要用用户的权限去做相关内容的修改），用pikachu平台模拟一下

受害者：192.168.147.144

攻击者：192.168.1.2

Pikchu：192.168.147.145

攻击者先自己登陆一下pikachu平台并且做一些修改用户参数的操作，并抓包，发现同过get请求并且没用任何的验证

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika3/%E5%9B%BE%E7%89%87%201.png?raw=true)

现在就可以直接伪造求情啦，诱使受害者（此时在线）点击就可以了

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika3/%E5%9B%BE%E7%89%87%203.png?raw=true)

假设受害者上当了点击了攻击者发的连接，发现自己的信息被修改了

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika3/%E5%9B%BE%E7%89%87%204.png?raw=true)

**--- --- CSRF（post）**

好的通过post传值了，也就意味着不能构造url了，咋办呢(-_-?)，还记得xss里可以构造一个页面帮我们完成一系列的操作，这个也可以这么干呀

构造一个页面先（我没学过前端，拿着靶场的xss部分里的post.html结合w3school照葫芦画了一个瓢，把我的这个瓢（文件）放在攻击者的网站上，受害者只要点击访问这个页面，攻击就完成了，当然受害者得处于在线状态...）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika3/%E5%9B%BE%E7%89%87%205.png?raw=true)

受害者访问攻击者构造得页面，发现信息被修改了

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika3/%E5%9B%BE%E7%89%87%206.png?raw=true)

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika3/%E5%9B%BE%E7%89%87%207.png?raw=true)

伪造的参数感觉还是隐藏起来比较好，比如input标签里的’type=hidden’ ……

**--- --- CSRF TOkEN**

Emmm... 我以为我可以，搞了好久，最后看教学视频得出结论...好吧我不可以