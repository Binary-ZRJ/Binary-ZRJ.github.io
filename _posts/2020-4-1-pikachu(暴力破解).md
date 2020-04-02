#### 暴力破解

**--- --- 基于表单的暴力破解**

没做任何的防护措施，直接抓包爆破就可以了

--- --- 验证码绕过（on server）

这种情况是服务器端生成并验证验证码，但是由于开发人员的疏忽，验证完毕后没有销毁验证码，导致验证码一直有效（只要用户不刷新验证码），所以上一种破解没什么区别，每次多提交一个相同的验证码就可以了

**--- --- 验证码绕过（on client）**

这种验证是通过js验证的好像没什么用，只能提示用户输入验证码错误，根本没有验证的效果。直接抓包爆破，验证码都可以不提交，因为服务端根本没有办法验证用户提交的验证码是个什么玩意儿

**--- --- token防爆破？**

这个相当于服务器为每一次用户的请求都发了一个token，用户每次登陆都要提交这个token，如果token和服务端发放的不一样，就验证失败。这个也是可以爆破的，我们只要把每次爆破的参数里加上服务端发放的token参数就可以了（每次token值的获取，可以通过抓包获取）

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/2.png?raw=true)

截获数据包，配置参数（这里假设知道用户名，要爆破的位置是密码和token，因为每一个token只可以用一次，所以模式选择pitchfork再好不过，简单废话两句这种模式的工作方式，假设两个字典dict_A,dict_B,dict_A里有三条记录1，2，3，dict_B里也有五条记录，1，2，3，4，5，那么payload的组合是[1，1],[2,2],[3,3]，看到是两位字典的一一对应，payload数目以短字典记录条数的为准，所以想想这种情况这种模式是不是再好不过了呢？）

然后是设置payload，密码字典加载不是问题，重点是token的payload去哪找呢？不急，burpsuit是非常周到的。

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/3.png?raw=true)

看到recursive grep了吗？这种方式代表接收来自burpsuit  grep（英语不好不知道怎么翻译）出来的数据作为payload，好的下一步怎么grep出想要的数据呐。

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/4.png?raw=true)

![](https://github.com/Binary-ZRJ/Binary-ZRJ.github.io/blob/master/blog_photos/pika_2/5.png?raw=true)

只要找到想要的数据值，burpsuit就自动帮我们生成，相应的正则表达式洛。

还等什么开始爆破吧（对了记得设置单线程，因为recursive grep只支持单线程哈）。