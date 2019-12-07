#### ------login1(SKCTF)

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login1.png?raw=true) 

Sql 约束攻击：

做个小实验

创建数据表：（用户名的长度设为25个字符）

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login2.png?raw=true) 

插入数据，管理员用户名（admin）和密码（admin）

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login3.png?raw=true) 

查询，一个不存在的用户，确定用户不存在，再把该用户插入

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login4.png?raw=true) 

由于用户名最长设置为25，而插入的数据长度超过了25，所以后面的  ‘1’  被截断，这时再查询表里的内容，发现有两个 ’admin‘ ，但是第二个 ’admin‘ 其实是 ‘admin                           ’  

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login5.png?raw=true) 

但使用select 语句时，查询内容（admin）后面的空格会被删除，所以会返回两条查询结果

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login6.png?raw=true) 

有了这个漏洞，开始做题

思路：

向数据库插入（通过注册功能）一个和已存在用户同名的用户名，再用插入的用户名和密码登陆得到想要的东西。

猜已经存在的用户名，输入admin时得想要的东西

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login7.png?raw=true) 

 注册，不知道后端限制长度是多少，注册时多输入结果空格

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login8.png?raw=true) 

 用 ‘admin’ 和注册的密码登陆

![img](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/login9.png?raw=true) 

 

 
