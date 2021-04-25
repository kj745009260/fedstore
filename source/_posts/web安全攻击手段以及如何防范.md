---
title: web安全攻击手段以及如何防范
date: 2021-03-11 10:50:38
tags: web安全
categories:
  - 浏览器特性
---

对于常规的Web攻击手段，如XSS、CRSF、SQL注入、（常规的不包括文件上传漏洞、DDoS攻击）等

比如XSS的防范需要转义掉输入的尖括号; 防止CRSF攻击需要将cookie设置为httponly，以及增加session相关的Hash token码; SQL注入的防范需要将分号等字符转义，

## __xss(cross site scripting) 跨站脚本攻击__

定义: 指攻击者在网页嵌入脚本，用户浏览网页触发恶意脚本执行

> XSS攻击分为3类：存储型（持久型）、反射型（非持久型）、基于DOM

如何防范:

> + 设置HttpOnly以避免cookie劫持的危险 
> + 过滤，对诸如script、img、a等标签进行过滤 
> + 编码，像一些常见的符号，如<>在输入的时候要对其进行转换编码 
> + 限制，对于一些可以预期的输入可以通过限制长度强制截断来进行防御

## __CSRF(cross site request forgery) 跨站请求伪造(CSRF 或者 XSRF)__

CSRF攻击的全称是跨站请求伪造（ cross site request forgery)，是一种对网站的恶意利用，尽管听起来跟XSS跨站脚本攻击有点相似，但事实上CSRF与XSS差别很大，XSS利用的是站点内的信任用户，而CSRF则是通过伪装来自受信任用户的请求来利用受信任的网站。

### __CSRF攻击原理__

> 1. 首先用户C浏览并登录了受信任站点A；
> 2. 登录信息验证通过以后，站点A会在返回给浏览器的信息中带上已登录的cookie，cookie信息会在浏览器端保存一定时间（根据服务端设置而定）；
> 3. 完成这一步以后，用户在没有登出（清除站点A的cookie）站点A的情况下，访问恶意站点B；
> 4. 这时恶意站点 B的某个页面向站点A发起请求，而这个请求会带上浏览器端所保存的站点A的cookie；
> 5. 站点A根据请求所带的cookie，判断此请求为用户C所发送的。

因此，站点A会报据用户C的权限来处理恶意站点B所发起的请求，而这个请求可能以用户C的身份发送 邮件、短信、消息，以及进行转账支付等操作，这样恶意站点B就达到了伪造用户C请求站点 A的目的。

受害者只需要做下面两件事情，攻击者就能够完成CSRF攻击：

> + 登录受信任站点 A，并在本地生成cookie；
> + 在不登出站点A（清除站点A的cookie）的情况下，访问恶意站点B。

### __CSRF的防御__

1、尽量使用POST，限制GET

GET接口太容易被拿来做CSRF攻击，看上面示例就知道，只要构造一个img标签，而img标签又是不能过滤的数据。接口最好限制为POST使用，GET则无效，降低攻击风险。

当然POST并不是万无一失，攻击者只要构造一个form就可以，但需要在第三方页面做，这样就增加暴露的可能性。

2、将cookie设置为HttpOnly

CRSF攻击很大程度上是利用了浏览器的cookie，为了防止站内的XSS漏洞盗取cookie,需要在cookie中设置“HttpOnly”属性，这样通过程序（如JavaScript脚本、Applet等）就无法读取到cookie信息，避免了攻击者伪造cookie的情况出现。
在Java的Servlet的API中设置cookie为HttpOnly的代码如下：

```javascript
ctx.cookies.set('cookiename', 'cookievalue', { httpOnly: true })
```

3、增加token

CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于cookie中，因此攻击者可以在不知道用户验证信息的情况下直接利用用户的cookie来通过安全验证。

由此可知，抵御CSRF攻击的关键在于：在请求中放入攻击者所不能伪造的信息，并且该信总不存在于cookie之中。鉴于此，系统开发人员可以在HTTP请求中以参数的形式加入一个随机产生的token，并在服务端进行token校验，如果请求中没有token或者token内容不正确，则认为是CSRF攻击而拒绝该请求。

假设请求通过POST方式提交，则可以在相应的表单中增加一个隐藏域：

```html
<input type="hidden" name="_toicen" value="tokenvalue"/>
```
token的值通过服务端生成，表单提交后token的值通过POST请求与参数一同带到服务端，每次会话可以使用相同的token，会话过期，则token失效，攻击者因无法获取到token，也就无法伪造请求。

在session中添加token的实现代码：

```java
  HttpSession session = request.getSession();
  Object token = session.getAttribute("_token");
  if(token == null I I "".equals(token)) {
      session.setAttribute("_token", UUID.randomUUIDO .toString());
  }
```

4、通过Referer识别

根据HTTP协议，在HTTP头中有一个字段叫Referer，它记录了该HTTP请求的来源地址。

在通常情况下，访问一个安全受限的页面的请求都来自于同一个网站。比如某银行的转账是通过用户访问http://www.xxx.com/transfer.do页面完成的，用户必须先登录www.xxx.com，然后通过单击页面上的提交按钮来触发转账事件。

当用户提交请求时，该转账请求的Referer值就会是提交按钮所在页面的URL（本例为www.xxx. com/transfer.do）。

如果攻击者要对银行网站实施CSRF攻击，他只能在其他网站构造请求，当用户通过其他网站发送请求到银行时，该请求的Referer的值是其他网站的地址，而不是银行转账页面的地址。

因此，要防御CSRF攻击，银行网站只需要对于每一个转账请求验证其Referer值即可，如果是以www.xx.om域名开头的地址，则说明该请求是来自银行网站自己的请求，是合法的；

如果Referer是其他网站，就有可能是CSRF攻击，则拒绝该请求。

取得HTTP请求Referer：

```java
  ctx.headers["Referer"]
```


## __sql注入(SQL injection)__

定义: 在未授权情况下，非法访问数据库信息

如何防范:

> + 杜绝用户提交的参数入库并且执行 
> + 在代码层，不准出现sql语句 
> + 在web输入参数处，对所有的参数做sql转义 
> + 上线测试，需要使用sql自动注入工具进行所有的页面sql注入测试