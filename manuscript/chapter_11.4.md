# 11.4 Example: Http download in blocks

This section will demonstrate the specific usage of dio through an example of "Http block download".

### principle

The Http protocol defines the response header field for block transmission, but whether it supports it depends on the implementation of the Server. We can specify the "range" field of the request header to verify whether the server supports block transmission. For example, we can use the curl command to verify:

``` dart 
bogon:~ duwen$ curl -H "Range: bytes=0-10" http://download.dcloud.net.cn/HBuilder.9.0.2.macosx_64.dmg -v
# 请求头
> GET /HBuilder.9.0.2.macosx_64.dmg HTTP/1.1
> Host: download.dcloud.net.cn
> User-Agent: curl/7.54.0
> Accept: */*
> Range: bytes=0-10
# 响应头
< HTTP/1.1 206 Partial Content
< Content-Type: application/octet-stream
< Content-Length: 11
< Connection: keep-alive
< Date: Thu, 21 Feb 2019 06:25:15 GMT
< Content-Range: bytes 0-10/233295878

```

The function of adding "Range: bytes=0-10" to the request header is to tell the server that we only want to get the content of the file 0-10 (including 10, 11 bytes in total) in this request. If the server supports block transmission, the response status code is 206, which means "partial content", and the response header contains the "Content-Range" field. If it does not support it, it will not include it. Let's take a look at the content of "Content-Range" above:

``` dart 
Content-Range: bytes 0-10/233295878

```

0-10 means the block returned this time, 233295878 means the total length of the file, the unit is byte, which means that the file is a little more than 233M.

Based on this, we can design a simple multi-threaded file block downloader, the idea of ​​realization is:

1.  First check whether it supports block transmission, if not, download it directly; if it does, download the remaining content in blocks.
2.  When each part is downloaded, it is saved to its own temporary file, and the temporary files are merged after all parts are downloaded.
3.  Delete temporary files.

### achieve

The following is the overall process:

``` dart 
// 通过第一个分块请求检测服务器是否支持分块传输  
Response response = await downloadChunk(url, 0, firstChunkSize, 0);
if (response.statusCode == 206) {    //如果支持
   //解析文件总长度，进而算出剩余长度
   total = int.parse(
       response.headers.value(HttpHeaders.contentRangeHeader).split("/").last);
   int reserved = total -
       int.parse(response.headers.value(HttpHeaders.contentLengthHeader));
   //文件的总块数(包括第一块)
   int chunk = (reserved / firstChunkSize).ceil() + 1;
   if (chunk > 1) {
       int chunkSize = firstChunkSize;
       if (chunk > maxChunk + 1) {
           chunk = maxChunk + 1;
           chunkSize = (reserved / maxChunk).ceil();
       }
       var futures = <Future>[];
       for (int i = 0; i < maxChunk; ++i) {
           int start = firstChunkSize + i * chunkSize;
           //分块下载剩余文件  
           futures.add(downloadChunk(url, start, start + chunkSize, i + 1));
       }
       //等待所有分块全部下载完成
       await Future.wait(futures);
   }
   //合并文件文件  
   await mergeTempFiles(chunk);
}

```

Below we use dio's `download`API implementation `downloadChunk`:

``` dart 
//start 代表当前块的起始位置，end代表结束位置
//no 代表当前是第几块
Future<Response> downloadChunk(url, start, end, no) async {
 progress.add(0); //progress记录每一块已接收数据的长度
 --end;
 return dio.download(
   url,
   savePath + "temp$no", //临时文件按照块的序号命名，方便最后合并
   onReceiveProgress: createCallback(no), // 创建进度回调，后面实现
   options: Options(
     headers: {"range": "bytes=$start-$end"}, //指定请求的内容区间
   ),
 );
}

```

Next to achieve `mergeTempFiles`:

``` dart 
Future mergeTempFiles(chunk) async {
 File f = File(savePath + "temp0");
 IOSink ioSink= f.openWrite(mode: FileMode.writeOnlyAppend);
 //合并临时文件  
 for (int i = 1; i < chunk; ++i) {
   File _f = File(savePath + "temp$i");
   await ioSink.addStream(_f.openRead());
   await _f.delete(); //删除临时文件
 }
 await ioSink.close();
 await f.rename(savePath); //合并后的文件重命名为真正的名称
}

```

Let's take a look at the complete implementation:

``` dart 
/// Downloading by spiting as file in chunks
Future downloadWithChunks(
 url,
 savePath, {
 ProgressCallback onReceiveProgress,
}) async {
 const firstChunkSize = 102;
 const maxChunk = 3;

 int total = 0;
 var dio = Dio();
 var progress = <int>[];

 createCallback(no) {
   return (int received, _) {
     progress[no] = received;
     if (onReceiveProgress != null && total != 0) {
       onReceiveProgress(progress.reduce((a, b) => a + b), total);
     }
   };
 }

 Future<Response> downloadChunk(url, start, end, no) async {
   progress.add(0);
   --end;
   return dio.download(
     url,
     savePath + "temp$no",
     onReceiveProgress: createCallback(no),
     options: Options(
       headers: {"range": "bytes=$start-$end"},
     ),
   );
 }

 Future mergeTempFiles(chunk) async {
   File f = File(savePath + "temp0");
   IOSink ioSink= f.openWrite(mode: FileMode.writeOnlyAppend);
   for (int i = 1; i < chunk; ++i) {
     File _f = File(savePath + "temp$i");
     await ioSink.addStream(_f.openRead());
     await _f.delete();
   }
   await ioSink.close();
   await f.rename(savePath);
 }

 Response response = await downloadChunk(url, 0, firstChunkSize, 0);
 if (response.statusCode == 206) {
   total = int.parse(
       response.headers.value(HttpHeaders.contentRangeHeader).split("/").last);
   int reserved = total -
       int.parse(response.headers.value(HttpHeaders.contentLengthHeader));
   int chunk = (reserved / firstChunkSize).ceil() + 1;
   if (chunk > 1) {
     int chunkSize = firstChunkSize;
     if (chunk > maxChunk + 1) {
       chunk = maxChunk + 1;
       chunkSize = (reserved / maxChunk).ceil();
     }
     var futures = <Future>[];
     for (int i = 0; i < maxChunk; ++i) {
       int start = firstChunkSize + i * chunkSize;
       futures.add(downloadChunk(url, start, start + chunkSize, i + 1));
     }
     await Future.wait(futures);
   }
   await mergeTempFiles(chunk);
 }
}

```

Now you can download in chunks:

``` dart 
main() async {
 var url = "http://download.dcloud.net.cn/HBuilder.9.0.2.macosx_64.dmg";
 var savePath = "./example/HBuilder.9.0.2.macosx_64.dmg";
 await downloadWithChunks(url, savePath, onReceiveProgress: (received, total) {
   if (total != -1) {
     print("${(received / total * 100).floor()}%");
   }
 });
}

```

### Thinking

1.  Can downloading in blocks really increase the download speed?
   
   In fact, the main bottleneck of the download speed depends on the network speed and the export speed of the server. If it is the same data source, the block download is of little significance, because the server is the same, and the export speed is determined, which mainly depends on the network speed. And the above example is officially same-source block download, readers can compare the block and non-block download speed. If there are multiple download sources, and the export bandwidth of each download source is limited, then the block download may be faster. The reason why it is said "possible" is because it is not certain, for example, there are three sources , The export bandwidth of the three sources is 1G/s, and the peak value of the network connected to our equipment is assumed to be only 800M/s, then the bottleneck is our network. Even if the bandwidth of our device is greater than any source, the download speed is still not necessarily faster than single-source single-line download. Imagine that there are two sources A and B. The speed of A source is 3 times that of B source. If you use block download If the two sources download half of each, readers can calculate the download time required, and then calculate the time required to download only from source A to see which is faster.
   
   The final speed of the block download is affected by many factors such as the network bandwidth where the device is located, the source/export speed, the size of each block, and the number of blocks. It is difficult to ensure the optimal speed in the actual process. In actual development, readers can test and compare before deciding whether to use it.
   
2.  Is there any practical use for downloading in chunks?
   
   There is also a more commonly used scenario for block downloading. The file can be divided into several blocks, and then a download status file can be maintained to record the status of each block, so that it can be restored even after the network is interrupted. The state before the interruption, the reader can try it out for the specific implementation, or there are some details that need special attention, such as the appropriate block size? How to deal with the half-downloaded block? Do you want to maintain a task queue?