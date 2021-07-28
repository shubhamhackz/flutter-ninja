# 11.3 Http request-Dio http library

Through the introduction in the previous section, we can find that it is more troublesome to directly use HttpClient to initiate a network request. Many things have to be handled manually. If it involves file upload/download, Cookie management, etc., it will be very cumbersome. Fortunately, the Dart community has some third-party http request libraries. It will be much simpler to use them to initiate http requests. In this section, we introduce the currently popular [dio](https://github.com/flutterchina/dio) libraries.

> dio is a powerful Dart Http request library that supports Restful API, FormData, interceptor, request cancellation, cookie management, file upload/download, timeout, etc. The usage of dio may change with its version upgrade. If there is a difference between the content described in this section and the official dio, please refer to the official dio document.

### Introduce

Introduce dio:

``` dart 
dependencies:
 dio: ^x.x.x #请使用pub上的最新版本

```

Import and create a dio instance:

``` dart 
import 'package:dio/dio.dart';
Dio dio =  Dio();

```

Next, you can initiate network requests through the dio instance. Note that one dio instance can initiate multiple http requests. Generally speaking, when the APP has only one http data source, the dio should use the singleton mode.

### Example

Initiate `GET`requests:

``` dart 
Response response;
response=await dio.get("/test?id=12&name=wendu")
print(response.data.toString());

```

For `GET`requests, we can pass the query parameters through the object. The above code is equivalent to:

``` dart 
response=await dio.get("/test",queryParameters:{"id":12,"name":"wendu"})
print(response);

```

Initiate a `POST`request:

``` dart 
response=await dio.post("/test",data:{"id":12,"name":"wendu"})

```

Initiate multiple concurrent requests:

``` dart 
response= await Future.wait([dio.post("/info"),dio.get("/token")]);

```

download file:

``` dart 
response=await dio.download("https://www.google.com/",_savePath);

```

Send FormData:

``` dart 
FormData formData = new FormData.from({
  "name": "wendux",
  "age": 25,
});
response = await dio.post("/info", data: formData)

```

If the data sent is FormData, dio will set the request header `contentType`to "multipart/form-data".

Upload multiple files via FormData:

``` dart 
FormData formData = new FormData.from({
  "name": "wendux",
  "age": 25,
  "file1": new UploadFileInfo(new File("./upload.txt"), "upload1.txt"),
  "file2": new UploadFileInfo(new File("./upload.txt"), "upload2.txt"),
    // 支持文件数组上传
  "files": [
     new UploadFileInfo(new File("./example/upload.txt"), "upload.txt"),
     new UploadFileInfo(new File("./example/upload.txt"), "upload.txt")
   ]
});
response = await dio.post("/info", data: formData)

```

It is worth mentioning that dio still uses requests initiated by HttpClient internally, so the proxy, request authentication, certificate verification, etc. are the same as HttpClient, and we can `onHttpClientCreate`set it in the callback, for example:

``` dart 
(dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate = (client) {
   //设置代理 
   client.findProxy = (uri) {
     return "PROXY 192.168.1.2:8888";
   };
   //校验证书
   httpClient.badCertificateCallback=(X509Certificate cert, String host, int port){
     if(cert.pem==PEM){
     return true; //证书一致，则允许发送数据
    }
    return false;
   };   
 };

```

Note that it `onHttpClientCreate`will be called when HttpClient needs to be created inside the current dio instance, so configuring HttpClient through this callback will take effect for the entire dio instance. If you want to request a separate proxy or certificate verification strategy for an application, you can create a new dio Examples can be.

How? Is it very simple? In addition to these basic usages, dio also supports request configuration, interceptors, etc. The official information is more detailed, so I won’t go into details in this book. For details, please refer to the dio homepage: [https://github.com/ flutterchina/dio](https://github.com/flutterchina/dio) . In the next section we will use dio to implement a block downloader.

### Instance

We request all public open source projects under the flutterchina organization through the Github open API to achieve:

1.  Loading pops up during the request phase
2.  After the request is over, if the request fails, an error message will be displayed; if it succeeds, a list of project names will be displayed.

code show as below:

``` dart 
class _FutureBuilderRouteState extends State<FutureBuilderRoute> {
 Dio _dio = new Dio();

 @override
 Widget build(BuildContext context) {

   return new Container(
     alignment: Alignment.center,
     child: FutureBuilder(
         future: _dio.get("https://api.github.com/orgs/flutterchina/repos"),
         builder: (BuildContext context, AsyncSnapshot snapshot) {
           //请求完成
           if (snapshot.connectionState == ConnectionState.done) {
             Response response = snapshot.data;
             //发生错误
             if (snapshot.hasError) {
               return Text(snapshot.error.toString());
             }
             //请求成功，通过项目信息构建用于显示项目名称的ListView
             return ListView(
               children: response.data.map<Widget>((e) =>
                   ListTile(title: Text(e["full_name"]))
               ).toList(),
             );
           }
           //请求未完成时弹出loading
           return CircularProgressIndicator();
         }
     ),
   );
 }
}

```
