# 7.5 Asynchronous UI update (FutureBuilder, StreamBuilder)

Many times we rely on some asynchronous data to dynamically update the UI. For example, when opening a page, we need to obtain data from the Internet first. During the process of obtaining data, we display a loading box, and we will render the page when the data is obtained; For another example, we want to show the progress of Stream (such as file stream, Internet data receiving stream). Of course, through StatefulWidget we can fully realize these functions. However, since this scenario of relying on asynchronous data to update the UI is very common in actual development, Flutter specifically provides `FutureBuilder`and `StreamBuilder`two components to quickly implement this function.

## 7.5.1 FutureBuilder

`FutureBuilder`It will rely on one `Future`, and it will `Future`dynamically build itself based on the state it depends on . Let's take a look at the `FutureBuilder`constructor:

``` dart 
FutureBuilder({
 this.future,
 this.initialData,
 @required this.builder,
})

```

-   `future`: `FutureBuilder`Dependent `Future`, usually an asynchronous time-consuming task.
   
-   `initialData`: Initial data, default data set by the user.
   
-   `builder`: Widget builder; this builder will `Future`be called multiple times at different stages of execution, and the builder signature is as follows:
   
``` dart 
   Function (BuildContext context, AsyncSnapshot snapshot)
   
```
   
   `snapshot`It will contain the status information and result information of the current asynchronous task. For example, we can `snapshot.connectionState`obtain the status information of the `snapshot.hasError`asynchronous task , judge whether the asynchronous task has errors, etc. The reader can view the `AsyncSnapshot`class definition for the complete definition.
   
   In addition, `FutureBuilder`the `builder`function signature `StreamBuilder`of `builder`is the same as.
   

### Example

We implement a route. When the route is opened, we get data from the Internet, and a loading box pops up when the data is obtained. When the data is obtained, the obtained data will be displayed if it succeeds, and an error will be displayed if it fails. Since we haven't introduced how to initiate a network request in flutter, we don't really go to the network to request data here, but simulate this process, and return a string after 3 seconds:

``` dart 
Future<String> mockNetworkData() async {
 return Future.delayed(Duration(seconds: 2), () => "我是从互联网上获取的数据");
}

```

`FutureBuilder`The code used is as follows:

``` dart 
...
Widget build(BuildContext context) {
 return Center(
   child: FutureBuilder<String>(
     future: mockNetworkData(),
     builder: (BuildContext context, AsyncSnapshot snapshot) {
       // 请求已结束
       if (snapshot.connectionState == ConnectionState.done) {
         if (snapshot.hasError) {
           // 请求失败，显示错误
           return Text("Error: ${snapshot.error}");
         } else {
           // 请求成功，显示数据
           return Text("Contents: ${snapshot.data}");
         }
       } else {
         // 请求未结束，显示loading
         return CircularProgressIndicator();
       }
     },
   ),
 );
}

```

The running results are shown in Figures 7-8 and 7-9:

![Figure 7-8](https://pcdn.flutterchina.club/imgs/7-8.png)![Figure 7-9](https://pcdn.flutterchina.club/imgs/7-9.png)

In the above code, we return different widgets `builder`according to the current asynchronous task status `ConnectionState`. `ConnectionState`Is an enumeration class, defined as follows:

``` dart 
enum ConnectionState {
 /// 当前没有异步任务，比如[FutureBuilder]的[future]为null时
 none,

 /// 异步任务处于等待状态
 waiting,

 /// Stream处于激活状态（流上已经有数据传递了），对于FutureBuilder没有该状态。
 active,

 /// 异步任务已经终止.
 done,
}

```

Note that it `ConnectionState.active`only `StreamBuilder`appears in.

## 7.5.2 StreamBuilder

We know that Dart is `Stream`also used to receive asynchronous event data. The `Future`difference is that it can receive the results of multiple asynchronous operations. It is often used in asynchronous task scenarios that read data multiple times, such as network content downloading, file reading Write etc. `StreamBuilder`It is `Stream`the UI component that is used to cooperate to show the event (data) changes on the stream. Let's take `StreamBuilder`a look at the default constructor:

``` dart 
StreamBuilder({
 Key key,
 this.initialData,
 Stream<T> stream,
 @required this.builder,
})

```

You can see that `FutureBuilder`the constructors of and are only different: the former requires one `future`, and the latter requires one `stream`.

### Example

Let's create an example of a timer: every 1 second, the count increases by 1. Here, we use `Stream`to generate a number every second:

``` dart 
Stream<int> counter() {
 return Stream.periodic(Duration(seconds: 1), (i) {
   return i;
 });
}

```

`StreamBuilder`The code used is as follows:

``` dart 

Widget build(BuildContext context) {
   return StreamBuilder<int>(
     stream: counter(), //
     //initialData: ,// a Stream<int> or null
     builder: (BuildContext context, AsyncSnapshot<int> snapshot) {
       if (snapshot.hasError)
         return Text('Error: ${snapshot.error}');
       switch (snapshot.connectionState) {
         case ConnectionState.none:
           return Text('没有Stream');
         case ConnectionState.waiting:
           return Text('等待数据...');
         case ConnectionState.active:
           return Text('active: ${snapshot.data}');
         case ConnectionState.done:
           return Text('Stream已关闭');
       }
       return null; // unreachable
     },
   );
}

```

Readers can run this example by themselves to view the results. Note that this example is only for demonstration purposes `StreamBuilder`. In actual combat, it can be used in any scene where the UI changes depending on multiple asynchronous data `StreamBuilder`.