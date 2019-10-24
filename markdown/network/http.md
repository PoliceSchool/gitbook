Cookie和Session的由来：源于http协议无状态的特点，为了记住http的状态，所以Cookie和Session应运而生，为了保存http的状态。

Cookie保存在客户端，cookie是第一次web端输入登陆信息，在server端生成保存并生成cookie,后通过response的setCookie方法，将登陆信息放到响应头中。浏览器得到响应后，如果有cookie，会将cookie信息保存到本地，带下次http-request请求时会将这个cookie放到request的Header中。后台就可以根据cookie中有没有登陆信息，或者登陆信息是否超时来做进一步操作。

原文链接：https://blog.csdn.net/zhanglf02/article/details/86607424————————————————

而Session保存在服务器

什么是Http无状态呢？举个栗子，你访问一个网站需要登录，然后你传了账号名和密码，服务端通过了验证，然后返回给你信息，但如果此时没有对你的登录状态进行相应的缓存，那么你下次访问这个网站其他页面的时候还是需要再传一次账号密码进行验证，因为http协议不会保存这些信息？为什么不会保存呢？[看这篇文章](https://blog.csdn.net/tennysonsky/article/details/44562435);这就是http无状态的例子，它不会保存你的历史信息。所以为了解决http无状态所带来的问题，就需要把需要保存的东西保存起来。保存在哪里呢？显而易见，可以在客户端也可以在服务端，如果在客户端则叫做Cookie，在服务端就叫做Session



Sessions常用Cookie来实现，具体来说就是服务端生成了Sessions，但是会把唯一标识当前Session的SessionId放在Cookies里面，然后再传到客户端，接下来客户端每次都会把Cookies里面的SessionId传到服务端，服务端通过客户端中的SessionId找到对应的Session：<https://web.stanford.edu/~ouster/cgi-bin/cs142-fall10/lecture.php?topic=cookie>