# 单点登录（SSO）

## 单机系统登录

因为http是无状态的，所以在单机系统中，我们通过`session`和`cookie`保存用户状态。

### 使用cookie保存会话状态

用户登录系统后，服务端可以在`response`中调用`addCookie(Cookie cookie)`方法保存用户标识，浏览器收到后以后的请求都会在`request Headers`中c带上这个`cookie`，服务端收到请求后解析`cookie`就可以得到用户标识，这样就能判断当前会话属于哪个用户。

### 使用session保存会话状态

用户登录系统后，在服务端可以通过`setAttribute(String key，Object value)`方法设置`session`，将用户信息保存在服务端，浏览器收到`response`之后，在`response Header`中就可以看到一条类似`Set-Cookie: JSESSIONID=A1D9FBF86E779B5DE28BB20EAD19F9B5; Path=/; HttpOnly`的记录，这是我们设置`session`时服务端自动生成的，下次再请求服务端时就会看到`request Headers`就会多了一条`Cookie: JSESSIONID=8260F1E79DDBF2EA75D8C73E04BFEE0F`，因为每次都带上了session id ，服务器（这里是指Tomcat或其他容器）自动帮我们区分了不同的会话，所以就直接可以用`getAttribute(String name)`获取`session`的值，并且保证张三不会获取到李四的值。

## 多系统登录

当我们有多个系统时，比如淘宝`https://www.tmall.com/`和天猫`https://www.tmall.com/`是两个系统，我们希望登录了淘宝之后，再打开天猫就直接是登录状态，一处登录，处处可用这就是单点登录。

> **单点登录**（英语：Single sign-on，缩写为 SSO），又译为**单一签入**，一种对于许多相互关连，但是又是各自独立的软件系统，提供访问控制的属性。当拥有这项属性时，当用户登录时，就可以获取所有系统的访问权限，不用对每个单一系统都逐一登录

因为cookie不能跨域，所以淘宝和天猫不能共享cookie。session是依赖当前系统的tomcat，使用不同的容器会产生不同的seession，session也不能共享，这两个系统不能共享会话。

**但是，如果我引入第三个系统，这个系统只负责登录，登录后用户的信息就可以存在session中，淘宝要登录时，可以重定向到这个系统，并且把淘宝的url拼在参数中，登录系统把session中的信息取出来，拼接在淘宝的url后边，那么淘宝的系统就可以获取到用户的信息了，也就可以实现登录了**



但是换个角度想，因为sesion在不同的系统中，所以不能共享，如果把session放到同一个系统里边