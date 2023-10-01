## weak_auth

攻防世界的一道弱密码爆破题目，主要是熟悉一下怎么用burpsuite去爆破弱密码，之前一直都没用过bp。

首先是进入网址是一个登录表单，简单地用账号111和密码111去测试，登录后页面无显示，查看网页源代码发现有提示please login as admin以及maybe you need a dictionary；那基本就是账号是admin，然后密码用字典去爆破。

通过google的SwitchyOmega插件设置代理127.0.0.1:8080，然后开启burpsuite打开拦截，页面输入账号为admin，密码任意，bp拦截请求后利用bp来进行密码的爆破，步骤截图如下：

![weak-auth01](https://github.com/2aurora2/CTF-WriteUp/blob/main/攻防世界/image/weak-auth01.png)

![weak-auth02](https://github.com/2aurora2/CTF-WriteUp/blob/main/攻防世界/image/weak-auth02.png)

![weak-auth03](https://github.com/2aurora2/CTF-WriteUp/blob/main/攻防世界/image/weak-auth03.png)

![weak-auth04](https://github.com/2aurora2/CTF-WriteUp/blob/main/攻防世界/image/weak-auth04.png)