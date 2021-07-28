# 11.1 File operations

Dart's IO library contains related classes for file reading and writing. It is part of the Dart syntax standard. Therefore, through the Dart IO library, whether it is a script under the Dart VM or Flutter, the file is manipulated through the Dart IO library, but and Compared with Dart VM, Flutter has an important difference in file system paths. This is because Dart VM runs on PC or server operating systems, while Flutter runs on mobile operating systems, so their file systems will have some differences.

#### APP directory

Android and iOS have different application storage directories. The [`PathProvider`](https://pub.dartlang.org/packages/path_provider)plugin provides a platform-transparent way to access common locations on the device file system. This class currently supports access to two file system locations:

-   **Temporary directory:** it can be used `getTemporaryDirectory()`to obtain a temporary directory; system can always remove the temporary directory (cache). On iOS, this corresponds [`NSTemporaryDirectory()`](https://developer.apple.com/reference/foundation/1409211-nstemporarydirectory)to the value returned. On Android, this is [`getCacheDir()`](https://developer.android.com/reference/android/content/Context.html#getCacheDir()the value returned by).
-   **Document directory:** You can use it `getApplicationDocumentsDirectory()`to get the document directory of the application, which is used to store files that only you can access. Only when the application is uninstalled, the system will clear the directory. On iOS, this corresponds to `NSDocumentDirectory`. On Android, this is the `AppData`directory.
-   **External storage directory** : It can be used `getExternalStorageDirectory()`to obtain an external storage directory, such as an SD card; because iOS does not support external directories, calling this method under iOS will throw `UnsupportedError`an exception, while under Android the result is `getExternalStorageDirectory`the return value in the android SDK .

Once your Flutter application has a file location reference, you can use the [dart:io](https://api.dartlang.org/stable/dart-io/dart-io-library.html) API to perform read/write operations on the file system. For details on using Dart to process files and directories, you can refer to the Dart language documentation. Let's look at a simple example.

#### Example

Let's take the counter as an example to realize that the number of clicks can be restored after the application exits and restarts. Here, we use files to save data:

1.  Introduce the PathProvider plug-in; `pubspec.yaml`add the following statement in the file:
   
``` dart 
   path_provider: ^0.4.1
   
```
   
   After adding, execute to `flutter packages get`get it, the version number may change over time, readers can use the latest version.
   
2.  achieve:
   
``` dart 
   import 'dart:io';
   import 'dart:async';
   import 'package:flutter/material.dart';
   import 'package:path_provider/path_provider.dart';
   
   class FileOperationRoute extends StatefulWidget {
     FileOperationRoute({Key key}) : super(key: key);
   
     @override
     _FileOperationRouteState createState() => new _FileOperationRouteState();
   }
   
   class _FileOperationRouteState extends State<FileOperationRoute> {
     int _counter;
   
     @override
     void initState() {
       super.initState();
       //从文件读取点击次数
       _readCounter().then((int value) {
         setState(() {
           _counter = value;
         });
       });
     }
   
     Future<File> _getLocalFile() async {
       // 获取应用目录
       String dir = (await getApplicationDocumentsDirectory()).path;
       return new File('$dir/counter.txt');
     }
   
     Future<int> _readCounter() async {
       try {
         File file = await _getLocalFile();
         // 读取点击次数（以字符串）
         String contents = await file.readAsString();
         return int.parse(contents);
       } on FileSystemException {
         return 0;
       }
     }
   
     Future<Null> _incrementCounter() async {
       setState(() {
         _counter++;
       });
       // 将点击次数以字符串类型写到文件中
       await (await _getLocalFile()).writeAsString('$_counter');
     }
   
     @override
     Widget build(BuildContext context) {
       return new Scaffold(
         appBar: new AppBar(title: new Text('文件操作')),
         body: new Center(
           child: new Text('点击了 $_counter 次'),
         ),
         floatingActionButton: new FloatingActionButton(
           onPressed: _incrementCounter,
           tooltip: 'Increment',
           child: new Icon(Icons.add),
         ),
       );
     }
   }
   
```
   
   The above code is relatively simple and will not be repeated. It should be noted that this example is only for demonstrating file reading and writing. In actual development, if you want to store some simple data, it will be easier to use the shared_preferences plugin.
   
   > Note that the API of the Dart IO library to manipulate files is very rich, but this book is not an introduction to the Dart language, so it will not be explained in detail. Readers can learn by themselves if they want.