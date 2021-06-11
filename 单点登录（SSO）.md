# 单点登录（SSO）

## 单机系统登录

因为http是无状态的，所以在单机系统中，我们通过`session`和`cookie`保存用户状态。

### 使用cookie保存会话状态

用户登录系统后，服务端可以在`response`中调用`addCookie(Cookie cookie)`方法保存用户标识，浏览器收到后以后的请求都会在`request Headers`中带上这个`cookie`，服务端收到请求后解析`cookie`就可以得到用户标识，这样就能判断当前会话属于哪个用户。

### 使用session保存会话状态

用户登录系统后，在服务端可以通过`setAttribute(String key，Object value)`方法设置`session`，将用户信息保存在服务端，浏览器收到`response`之后，在`response Header`中就可以看到一条类似`Set-Cookie: JSESSIONID=A1D9FBF86E779B5DE28BB20EAD19F9B5; Path=/; HttpOnly`的记录，这是我们设置`session`时服务端自动生成的，下次再请求服务端时就会看到`request Headers`就会多了一条`Cookie: JSESSIONID=8260F1E79DDBF2EA75D8C73E04BFEE0F`，因为每次都带上了session id ，服务器（这里是指Tomcat或其他容器）自动帮我们区分了不同的会话，所以就直接可以用`getAttribute(String name)`获取`session`的值，并且保证张三不会获取到李四的值。

## 多系统登录

当我们有多个系统时，比如淘宝`https://www.tmall.com/`和天猫`https://www.tmall.com/`是两个系统，我们希望登录了淘宝之后，再打开天猫就直接是登录状态，一处登录，处处可用这就是单点登录。

> **单点登录**（英语：Single sign-on，缩写为 SSO），又译为**单一签入**，一种对于许多相互关连，但是又是各自独立的软件系统，提供访问控制的属性。当拥有这项属性时，当用户登录时，就可以获取所有系统的访问权限，不用对每个单一系统都逐一登录

因为cookie不能跨域，所以淘宝和天猫不能共享cookie。session是依赖当前系统的tomcat，使用不同的容器会产生不同的session，session也不能共享，这两个系统不能共享会话。

**但是，如果我引入第三个系统，这个系统只负责登录，淘宝登录这个系统之后，保存一份用户信息，天猫要登录时发现这个系统已经有了用户信息，那么取到用户信息也就完成登录了。** 这就是单点登录的大致思想。

## 登录流程

从这样图应该可以清晰看到整个登录流程

![sso](http://saulimg.zsly.xyz/img/sso.png)

1. 用户要访问系统a，检验系统a的cookie，cookie不存在，说明用户没登录。则携带系统a的url到sso系统登录，形如`http://hellosso.com:8089/login?returnUrl=http://client1.com:8088`
2. sso系统验证发现sso系统不是登录状态，那么就在sso系统输入用户名，密码进行登录
3. 登录成功之后创建**用户会话**保存用户信息；创建**sso票据**，可以存入sso系统的cookie，用来表示sso系统是登录状态；创建**临时票据**，把临时票据传给系统a，之后系统a再带着临时票据到sso系统校验临时票据是否有效
4. sso系统校验临时票据成功后就可以将用户信息传给系统a，系统a在cooke中记下用户信息，表示系统a是登录状态，至此系统a完成了登录
5. 之后系统b登录时，系统b要携带系统b的url到sso系统，因为sso系统已经是登录状态，所以sso系统发给系统b一个临时票据，之后系统b再带着临时票据到sso校验，成功之后，sso系统将用户信息传给系统b，系统b在cookie中记下，表示系统b是登录状态，系统b也完成了登录。

## 退出登录

1. 系统a要退出登录时，重定向到sso系统，去掉sso系统的cookie，再去掉用户会话，系统a再去掉本地cookie，系统a就退出登录了。
2. 系统b再访问的时候因sso系统没有cookie，并且没有用户会话，那么也就可以让系统b退出登录了

## 示例演示

源代码已上传GitHub，https://github.com/Saul-Zhang/hello-sso

其中用户会话等信息我使用了redis来存储。

redis中存储的信息：

|          | key        | value           |
| -------- | ---------- | --------------- |
| 用户会话 | 用户id     | 用户详细信息    |
| sso票据  | sso_ticket | 用户id          |
| 临时票据 | tmp_ticket | tmp_ticket的md5 |

### 示例截图

![image-20210308000127431](http://saulimg.zsly.xyz/img/image-20210308000127431.png)

![image-20210308000825251](http://saulimg.zsly.xyz/img/image-20210308000825251.png)

![image-20210308000844529](http://saulimg.zsly.xyz/img/image-20210308000844529.png)

# 参考

https://apereo.github.io/cas/4.2.x/images/cas_flow_diagram.png