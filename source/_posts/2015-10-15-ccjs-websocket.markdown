---
layout: post
title: "WebSocket数据帧"
date: 2015-10-15 17:35:06 +0800
comments: true
categories: 
---

WebSocket通过onmessage函数接收服务器返回的数据,其中evt.data的数据格式可能为String、ArrayBuffer或Blob三者之一。这个数据格式是由服务器返回数据里的数据帧的opcode决定的。

所以,即便服务器返回的数据是byte或者char,如果opcode的值仍是文本类型,客户端还是会收到一个字符串。


- <https://w3c.github.io/websockets/#websocket>
- <http://www.cnblogs.com/fengyunlishi/archive/2013/05/10/3071893.html>


#代码:

<https://github.com/sbxfc/CocosJSWebSocket>