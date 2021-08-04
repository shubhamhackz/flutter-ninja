# 11.2 Initiate HTTP request through HttpClient

The Dart IO library provides some classes for initiating Http requests, which we can use directly `HttpClient`to initiate requests. There `HttpClient`are five steps to use to initiate a request:

1.  Create one `HttpClient`:
   
``` dart 
    HttpClient httpClient = new HttpClient();
   
```
   
2.  Open the Http connection and set the request header:
   
``` dart 
   HttpClientRequest request = await httpClient.getUrl(uri);
   
```
   
   Any Http Method can be used in this step, such as `httpClient.post(...)`, `httpClient.delete(...)`etc. If the query parameter is included, it can be added when constructing the uri, such as:
   
``` dart 
   Uri uri=Uri(scheme: "https", host: "flutterchina.club", queryParameters: {
       "xx":"xx",
       "yy":"dd"
     });
   
```
   
   By `HttpClientRequest`be provided request header, such as:
   
``` dart 
   request.headers.add("user-agent", "test");
   
```
   
   If it is a post or put method that can carry the request body, the request body can be sent through the HttpClientRequest object, such as:
   
``` dart 
   String payload="...";
   request.add(utf8.encode(payload)); 
   //request.addStream(_inputStream); //可以直接添加输入流
   
```
   
3.  Waiting to connect to the server:
   
``` dart 
   HttpClientResponse response = await request.close();
   
```
   
   After this step is completed, the request information has been sent to the server, and an `HttpClientResponse`object is returned , which contains the response header (header) and the response stream (Stream of the response body), and then the response content can be obtained by reading the response stream.
   
4.  Read response content:
   
``` dart 
   String responseBody = await response.transform(utf8.decoder).join();
   
```
   
   We get the data returned by the server by reading the response stream. We can set the encoding format when reading, here is utf8.
   
5.  End of request, close `HttpClient`:
   
``` dart 
   httpClient.close();
   
```
   
   After closing the client, all requests initiated through the client will be aborted.
   

#### Example

We implement an example of obtaining Baidu homepage html, the example effect is shown in Figure 11-1:

​  ![Figure 11-1](../resources/imgs/11-1.png)

After clicking the "Get Baidu Homepage" button, it will request the Baidu homepage. After the request is successful, we will display the returned content and print the response header on the console. The code is as follows:

``` dart 
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';

class HttpTestRoute extends StatefulWidget {
 @override
 _HttpTestRouteState createState() => new _HttpTestRouteState();
}

class _HttpTestRouteState extends State<HttpTestRoute> {
 bool _loading = false;
 String _text = "";

 @override
 Widget build(BuildContext context) {
   return ConstrainedBox(
     constraints: BoxConstraints.expand(),
     child: SingleChildScrollView(
       child: Column(
         children: <Widget>[
           RaisedButton(
               child: Text("获取百度首页"),
               onPressed: _loading ? null : () async {
                 setState(() {
                   _loading = true;
                   _text = "正在请求...";
                 });
                 try {
                   //创建一个HttpClient
                   HttpClient httpClient = new HttpClient();
                   //打开Http连接
                   HttpClientRequest request = await httpClient.getUrl(
                       Uri.parse("https://www.baidu.com"));
                   //使用iPhone的UA
                   request.headers.add("user-agent", "Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_1 like Mac OS X) AppleWebKit/603.1.30 (KHTML, like Gecko) Version/10.0 Mobile/14E304 Safari/602.1");
                   //等待连接服务器（会将请求信息发送给服务器）
                   HttpClientResponse response = await request.close();
                   //读取响应内容
                   _text = await response.transform(utf8.decoder).join();
                   //输出响应头
                   print(response.headers);

                   //关闭client后，通过该client发起的所有请求都会中止。
                   httpClient.close();

                 } catch (e) {
                   _text = "请求失败：$e";
                 } finally {
                   setState(() {
                     _loading = false;
                   });
                 }
               }
           ),
           Container(
               width: MediaQuery.of(context).size.width-50.0,
               child: Text(_text.replaceAll(new RegExp(r"\s"), ""))
           )
         ],
       ),
     ),
   );
 }
}

```

Console output:

``` dart 
I/flutter (18545): connection: Keep-Alive
I/flutter (18545): cache-control: no-cache
I/flutter (18545): set-cookie: ....  //有多个，省略...
I/flutter (18545): transfer-encoding: chunked
I/flutter (18545): date: Tue, 30 Oct 2018 10:00:52 GMT
I/flutter (18545): content-encoding: gzip
I/flutter (18545): vary: Accept-Encoding
I/flutter (18545): strict-transport-security: max-age=172800
I/flutter (18545): content-type: text/html;charset=utf-8
I/flutter (18545): tracecode: 00525262401065761290103018, 00522983

```

#### HttpClient configuration

`HttpClient`There are many attributes that can be configured. The list of commonly used attributes is as follows:

| Attributes            | Meaning                                                                                                                                                                                                                                                                           |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| idleTimeout           | Corresponding to the keep-alive field value in the request header, in order to avoid frequent connection establishment, httpClient will keep the connection for a period of time after the request ends, and the connection will be closed only after this threshold is exceeded. |
| connectionTimeout     | The timeout for establishing a connection with the server, if it exceeds this value, a SocketException will be thrown.                                                                                                                                                            |
| maxConnectionsPerHost | The maximum number of connections allowed for the same host at the same time.                                                                                                                                                                                                     |
| autoUncompress        | Corresponding to the Content-Encoding in the request header, if set to true, the value of the Content-Encoding in the request header is the list of compression algorithms supported by the current HttpClient, currently only "gzip"                                             |
| userAgent             | Corresponds to the User-Agent field in the request header.                                                                                                                                                                                                                        |



It can be found that some attributes are just for more convenient setting of the request header. For these attributes, you can `HttpClientRequest`set the header directly. The difference is that the `HttpClient`settings `httpClient`are effective for the whole , and the `HttpClientRequest`settings are only effective for the current request.

#### HTTP request authentication

The authentication mechanism of the Http protocol can be used to protect non-public resources. If authentication is enabled on the Http server, the user needs to carry user credentials when making a request. If you access a resource with Basic authentication enabled in the browser, a login box will pop up when browsing, such as:

![image-20181031114207514](https://cdn.jsdelivr.net/gh/flutterchina/flutter-in-action@1.0/docs/imgs/image-20181031114207514.png)

Let's first look at the basic process of Basic authentication:

1.  The client sends an http request to the server, and the server verifies whether the user has been logged in and verified. If not, the server will return a 401 Unauthozied to the client and add a "WWW-Authenticate" field to the response header, for example:
   
``` dart 
   WWW-Authenticate: Basic realm="admin"
   
```
   
   Among them, "Basic" is the authentication method, and realm is the grouping of user roles, which can be added in the background.
   
2.  After the client gets the response code, base64 encode the username and password (the format is username:password), set the request header Authorization, and continue to visit:
   
``` dart 
   Authorization: Basic YXXFISDJFISJFGIJIJG
   
```
   
   The server verifies the user credentials, and returns the resource content if passed.
   

Note that in addition to Basic authentication, Http methods include Digest authentication, Client authentication, Form Based authentication, etc. Currently Flutter's HttpClient only supports Basic and Digest authentication methods. The biggest difference between these two authentication methods is sending user credentials. For the content of user credentials, the former is simply encoded by Base64 (reversible), while the latter is hashed, which is relatively safer, but for the sake of security, **whether it is Basic authentication or Digest authentication, it should be Under the Https protocol** , this can prevent packet capture and man-in-the-middle attacks.

`HttpClient`About Http authentication methods and attributes:

1.  `addCredentials(Uri url, String realm, HttpClientCredentials credentials)`
   
   This method is used to add user credentials, such as:
   
``` dart 
   httpClient.addCredentials(_uri,
    "admin", 
     new HttpClientBasicCredentials("username","password"), //Basic认证凭据
   );
   
```
   
   If it is Digest authentication, you can create Digest authentication credentials:
   
``` dart 
   HttpClientDigestCredentials("username","password")
   
```
   
2.  `authenticate(Future<bool>  f(Uri url, String scheme, String realm))`
   
   This is a setter, and the type is a callback. When the server requires user credentials and the user credentials have not been added, httpClient will call this callback. In this callback, it is usually called `addCredential()`to dynamically add user credentials, for example:
   
``` dart 
   httpClient.authenticate=(Uri url, String scheme, String realm) async{
     if(url.host=="xx.com" && realm=="admin"){
       httpClient.addCredentials(url,
         "admin",
         new HttpClientBasicCredentials("username","pwd"), 
       );
       return true;
     }
     return false;
   };
   
```
   
   One suggestion is that if all requests require authentication, it should be called when HttpClient is initialized `addCredentials()`to add global credentials instead of dynamically adding them.
   

#### proxy

You can `findProxy`set the proxy policy through, for example, we want to send all requests through the proxy server (192.168.1.2:8888):

``` dart 
 client.findProxy = (uri) {
   // 如果需要过滤uri，可以手动判断
   return "PROXY 192.168.1.2:8888";
};

```

`findProxy` The callback return value is a string that follows the PAC script format of the browser. For details, please refer to the API documentation. If no proxy is required, just return "DIRECT".

In APP development, many times we need to capture packets for debugging, and the packet capture software (such as Charles) is an agent, then we can send the request to our packet capture software, we can see in the packet capture software To the requested data.

Sometimes the proxy server also enables authentication, which is similar to http protocol authentication. HttpClient provides the corresponding proxy authentication methods and attributes:

``` dart 
set authenticateProxy(
   Future<bool> f(String host, int port, String scheme, String realm));
void addProxyCredentials(
   String host, int port, String realm, HttpClientCredentials credentials);

```

The above method and their use "HTTP Authentication Request" described in section `addCredentials`and `authenticate`the same, it will not be repeated.

#### Certificate verification

In order to prevent man-in-the-middle attacks initiated by forged certificates in Https, the client should verify self-signed or non-CA certificates. `HttpClient`The logic of certificate verification is as follows:

1.  If the requested Https certificate is issued by a trusted CA, and the access host is included in the domain list of the certificate (or conforms to the wildcard rule) and the certificate has not expired, the verification is passed.
2.  If the first step verification fails, but the certificate has been added to the certificate trust chain through SecurityContext when the HttpClient is created, then if the certificate returned by the server is in the trust chain, the verification passes.
3.  If the verification of 1 and 2 fails, if the user provides a `badCertificateCallback`callback, it will be called. If the callback returns `true`, the link is allowed to continue, and if it returns `false`, the link is terminated.

To sum up, our certificate verification is actually to provide a `badCertificateCallback`callback, which is illustrated by an example below.

##### Example

Assuming that our background service uses a self-signed certificate, the certificate format is PEM format, and we save the content of the certificate in a local string, then our verification logic is as follows:

``` dart 
String PEM="XXXXX";//可以从文件读取
...
httpClient.badCertificateCallback=(X509Certificate cert, String host, int port){
 if(cert.pem==PEM){
   return true; //证书一致，则允许发送数据
 }
 return false;
};

```

`X509Certificate`It is the standard format of the certificate, which contains all the information of the certificate except the private key. Readers can consult the document by themselves. In addition, the above example does not verify the host, because as long as the content of the certificate returned by the server is consistent with the local storage, it can be proved to be our server (not the middleman). The host verification is usually to prevent the certificate and the domain name from mismatching.

For a self-signed certificate, we can also add it to the local certificate trust chain, so that the certificate will automatically pass when it is verified, instead of going to the `badCertificateCallback`callback:

``` dart 
SecurityContext sc=new SecurityContext();
//file为证书路径
sc.setTrustedCertificates(file);
//创建一个HttpClient
HttpClient httpClient = new HttpClient(context: sc);

```

Note that `setTrustedCertificates()`the certificate format set must be PEM or PKCS12. If the certificate format is PKCS12, the certificate password needs to be passed in. This will expose the certificate password in the code. Therefore, it is not recommended to use PKCS12 format certificates for client certificate verification. .

#### to sum up

It is worth noting that `HttpClient`these provided attributes and methods will eventually be used in the request header. We can do this by setting the header manually. The reason why these methods are provided is only for the convenience of developers. In addition, the Http protocol is a very important and most used network protocol. Every developer should be very familiar with the http protocol.