# 11.6 Using Socket API

The Http protocol and WebSocket protocol that we introduced earlier are both application layer protocols. In addition to them, there are many application layer protocols such as SMTP, FTP, etc. The implementation of these application layer protocols is achieved through Socket API. In fact, the native network request API provided in the operating system is standard. In the Socket library of the C language, it mainly provides the basic API for establishing a link and sending data from end to end, while the Socket library in high-level programming languages ​​is actually A wrapper to the socket API of the operating system. Therefore, if we need to customize the protocol or want to directly control and manage the network connection, or we feel that the built-in HttpClient is not easy to use and want to reimplement one, then we need to use Socket. Flutter's Socket API is in the dart:io package. Let's look at an example of using Socket to implement a simple http request. Take the Baidu homepage as an example:

``` dart 
_request() async{
 //建立连接
 var socket=await Socket.connect("baidu.com", 80);
 //根据http协议，发送请求头
 socket.writeln("GET / HTTP/1.1");
 socket.writeln("Host:baidu.com");
 socket.writeln("Connection:close");
 socket.writeln();
 await socket.flush(); //发送
 //读取返回内容
 _response =await socket.transform(utf8.decoder).join();
 await socket.close();
}

```

As you can see, using Socket requires us to implement the Http protocol ourselves (the communication process with the server needs to be implemented by ourselves). This example is just a simple example, and does not handle redirection, cookies, etc. Refer to the demo demo for the complete code of this example, the effect after running is shown in Figure 11-2:

![Figure 11-2](https://pcdn.flutterchina.club/imgs/11-2.png)

You can see that the response content is divided into two parts, the first part is the response header, and the second part is the response body. The server can dynamically output the response body according to the request information. Since the request header in this example is relatively simple, the response body will be different from the one accessed in the browser. Readers can add some request headers (such as user-agent) to see the output changes.