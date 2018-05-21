---
layout:       post
title:        "同源策略与跨域问题"
subtitle:     "JavaScript, 网络, 基础知识"
date:         2018-05-21
author:       "Chou"
header-img:   "img/post-bg-cross-domain.jpg"
header-mask:  0.3
catalog:      true
multilingual: false
tags:
    - 前端开发
    - JavaScript
    - 网络 
    - 知识总结

---

## 同源策略

什么是同源策略： 协议相同，域名相同，端口相同（IE不对端口进行限制）

受限行为：

`*`  Cookie，LocalStorage，IndexDB无法读取

`*`  无法获取DOM

`*` Ajax请求无法发送



## 解决跨域问题

### Cookie

通过设置document.domain共享Cookie

### window.postMessage方法

#### 父窗口http://aaa.com向子窗口http://bbb.com发送消息：

```javascript
var popup = window.open('http://bbb.com', 'title');
popup.postMessage('Hello', 'http://bbb.com');
```

 postMessage方法第一个参数是信息内容，第二个参数是接受消息的窗口的源。

#### 子窗口向父窗口发送消息

```javascript
window.open.postMessage('Nice to see you', 'http://aaa.com');
// 子窗口通过event.source属性引用父窗口，发送消息。
window.addEventListener('message', receiveMessage)
function receiveMessage(event) {
	event.source.postMessage('Nice to see you!', '*')
}
// event属性过滤不是发给本窗口的消息。
window.addEventListener('message', receiveMessage)
function receievMessage(event) {
	if (event.origin !== 'http://aaa.com') return
    if (event.data === 'Hello World') {
    	event.source.postMessage('Hello', event.origin)
    } else {
    	console.log(event.data)
    }
}
```



postMessage方法读取其他窗口的LocalStorage

```javascript
// 子窗口将父窗口发来的消息写入自己的LocalStorage
window.onmessage = function(e) {
	if (e.origin !== 'http://bbb.com') {
		return
	}
	var payload = JSON.parse(e.data)
	localStorage.setItem(payload.key, JSON.stringify(payload.data))
}
// 父窗口发送消息的代码
var win = document.getElementsByTagName('iframe')[0].contentWindow
var obj = { name: 'Jack'}
win.postMessage(JSON.stringify({key: 'storage', data: obj}), 'http://bbb.com')
```


加强版子窗口接受消息代码

```javascript
window.onmessage = function(e) {
	if (e.origin !== 'http://bbb.com') return
	var payload = JSON.parse(e.data)
    switch (payload.method) {
    	case 'set':
    	  localStorage.setItem(payload.key, JSON.stringify(payload.data))
    	  break
    	case 'get':
    	  var parent = window.parent
    	  var data = localStorage.getItem(payload.key)
    	  parent.postMessage(data, 'http://aaa.com')
    	  break
    	case 'remove':
    	  localStorage.removeItem(payload.key)
    	  break
    }
}
```


加强版父窗口发送消息代码

```javascript
var win = document.getElementsByTagName('iframe')[0].contentWindow
var obj = { name: 'Jack'}
// 存入对象
win.postMessage(JSON.stringify({key: 'storage', method: 'set', data: obj}), 'http://bbb.com')
// 读取对象
win.postMessage(JSON.stringify({key: 'storage', method: 'get'}), "*")
window.onmessage = function(e) {
	if (e.origin !== 'http://aaa.com') return
	// "Jack"
    console.log(JSON.parse(e.data).name)
}
```

### JSONP

 网页通过添加``` <script> ```元素，向服务器请求JSON数据。服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。

简单，兼容性好，但是只能发送GET请求。

```javascript
// 网页动态插入<script>标签，由它向跨域网址发出请求
function addScriptTag(src) {
    var script = document.createElement('script')
    script.setAttribute("type", "text/javascript")
	script.src = src
	document.body.appendChild(script)
}
// 查询字符串有一个callback参数，用来指定回调函数的名字
window.onload = function () {
	addScriptTag('http://example.com/ip?callback=foo')
}
function foo(data) {
	console.log('Your public IP address is: ' + data.ip)
}
// 服务器收到这个请求后，会将数据放在回调函数的参数位置返回。
foo({
	"ip": "8.8.8.8"
})// 作为参数的JSON数据被视为JavaScript对象，而不是字符串，避免了使用JSON.parse的步骤。
```



### WebSocket

 WebSocket是一种通讯协议，不实行同源政策，只要服务器支持就可以进行跨源通信。



### CORS 

跨源资源分享(Cross-Origin Resource Sharing)，允许任何类型的请求。

只要服务器实现了CORS接口，就可以进行跨域通信。

CORS请求分为两种：简单请求和非简单请求。

简单请求：

（1）请求是GET、POST、HEAD之一。

（2）HTTP头信息字段在下面的几种之间。

> ・Accept
>
> ・Accept-Language
>
> ・Content-Language
>
> ・Last-Event-ID
>
> ・Content-Type

不满足上述情况的请求都属于复杂请求。

#### 简单请求通信过程

`*` **添字段**

浏览器发现此次请求是简单请求之后，在HTTP头信息中添加``` Origin``` 字段。

```javascript
GET /cors HTTP/1.1
Origin: http://api.bob.com  /* 指定请求的来源 */
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0
```

`*`  **查看白名单**

服务器根据``` Origin``` 字段内容，判断该域名是否在白名单内。如果在，就会返回响应，并添加几个字段，如下：

```javascript
Access-Control-Allow-Origin: http://api.bob.com
// 必须 表示请求的源，如果该字段是*，表示接受任何源的请求。
Access-Control-Allow-Credentials: true
// 可选 是否发送Cookie
// 而且要在Ajax请求中打开withCredentials属性
/*
var xhr = new XMLHttpRequest()
xhr.withCredentials = true
*/
Access-Control-Expose-Headers: FooBar
// 可选 指定想拿到的字段。XMLHttpRequest对象的getResponseHeader()只能拿到Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma
Content-Type: text-html; charset=utf-8
```

如果源域名不在白名单之内，服务器会返回响应，这个响应里不包含``` Access-Control-Allow-Origin```字段。

浏览器发现这个问题，抛出错误，被XMLHttpRequest的onerror函数捕获。



#### 非简单请求通信过程

当CORS请求的请求方法是PUT、DELETE，或者Content-Type的类型是```application/json```。

与简单请求的区别是，在通信之前增加了一次HTTP请求，成为预检请求。

该请求会先询问服务器，当前域名是否在服务器的白名单之中，以及可以使用的请求方法和字段有哪些。在得到服务器的答复之后，浏览器才会发送真正的XMLHttpRequest请求。

`*` 浏览器根据请求方法和信息头判定该请求为非简单请求。

```javascript
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpReauest();
xhr.open('PUT', url, true); // 请求方法：PUT
xhr.setRequestHeader('X-Custom-Header', value); // 自定义信息头
xhr.send();
```



`*` 发送预检请求，该请求的HTTP头信息如下：

```javascript
OPTIONS /cors HTTP/1.1 // 'OPTIONS'表示该请求是预检请求
Origin: http://api.bob.com
Access-Control-Request-Method: PUT // 必须 说明接下来的CORS请求的请求方法。
Access-Control-Request-Headers: X-Custom-Header // CORS请求的头信息
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

`*` 服务器的回应

同意跨域请求的回应：

```javascript
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

不同意跨域请求：

浏览器检测到服务器不同意预检请求时，会触发错误处理，由XMLHttpRequest的onerror函数捕获。



`*`  服务器通过预检请求之后，浏览器的每次请求都会被视为简单请求。



#### 应用场景

`#` 由XMLHttpRequest或者fetch发起的跨域请求。

`#` Web 字体 (CSS 中通过` @font-face `使用跨域字体资源) 。

`#`  [WebGL 贴图](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial/Using_textures_in_WebGL)

`#`  使用 `drawImage` 将 Images/video 画面绘制到 canvas

`#` 样式表（使用 [CSSOM](https://developer.mozilla.org/en-US/docs/Web/CSS/CSSOM_View)） 

`#` Scripts(未处理的异常) 



### JONP与CORS的比较

#### 浏览器支持：

浏览器对于JSONP的支持较好。

主流浏览器支持CORS，但是IE10以下不支持。



#### 请求类型

相对于JSONP，支持所有类型的HTTP请求，JSONP只支持GET请求。



#### 错误处理

JSONP没法进行错误处理。

CORS的错误会被onerror函数捕获，能在浏览器看到报错信息。



#### JSONP的安全问题



##### JSON劫持：

属于CSRF(Cross-site request forgery 跨站请求伪造 )：攻击者伪造JSONP调用页面，获取用户的隐私数据。



解决方法：

`*` 方法一：

验证JSON文件调用来源（正则匹配）：在网站输出JSON数据时，判断Referer是不是在白名单内。

这种方法可以解决问题，但是不严谨。

如果我们通过验证某个关键词，比如``` google```，对http://www.google.com/login/php?callback=cb进行Referer过滤，但是攻击者依然可以伪造成http://www.attack.com/attack.htm?google.com或者http://www.google.attack.com/attack.htm绕过验证进行攻击。

还有，一般情况下浏览器访问URL是不带Referer的，请求头没有Referer。



`*` 方法二

添加随机token



##### 由callback引发的安全问题

由于callback可以自定义，如果没严格定义好Content-Type，没有对callback进行过滤，就会导致XSS漏洞。



解决方法：

`*` 将Content-Type定义为application/json

`*`  过滤callback以及JSON数据输出

对输出结果进行转码处理，指定Content-Type的编码为utf-8，即Content-Type: application/json; utf-8
