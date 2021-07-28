# Use WebSockets

The Http protocol is stateless and can only be initiated by the client. The server responds passively. The server cannot actively push content to the client. Once the server responds, the link will be disconnected (see the notes section), so it cannot Real-time communication. The WebSocket protocol is a technology produced to solve the real-time communication between the client and the server. It is now supported by mainstream browsers, so it should be familiar to Web developers. Flutter also provides a special package to support the WebSocket protocol.

> Note: Although the keep-alive mechanism can be used in the Http protocol to keep the server connected for a period of time after the response ends, it will eventually be disconnected. The keep-alive mechanism is mainly used to avoid frequent requests for multiple resources on the same server Creating a link is essentially a technology that supports link reuse, not for real-time communication. Readers need to know the difference between the two.

The WebSocket protocol is essentially a TCP-based protocol. After a special http request is initiated through the HTTP protocol for handshake, if the server supports the WebSocket protocol, the protocol will be upgraded. WebSocket will use the tcp link created after the http protocol handshake. Unlike the http protocol, the tcp link of WebSocket is a long link (will not be disconnected), so the server and client can communicate in real time through this TCP connection. For the details of the WebSocket protocol, readers can read the RFC document. Let's focus on how to use WebSocket in Flutter.

In the following example, we will connect to [the test server provided](http://www.websocket.org/echo.html) by [websocket.org](http://www.websocket.org/echo.html) . The server will simply return the same message we sent to it!

### step

1.  Connect to the WebSocket server.
2.  Listen for messages from the server.
3.  Send data to the server.
4.  Close the WebSocket connection.

### 1. Connect to the WebSocket server

[The web_socket_channel](https://pub.dartlang.org/packages/web_socket_channel) package provides the tools we need to connect to the WebSocket server. This package provides a method that `WebSocketChannel`allows us to listen to messages from the server and send messages to the server.

In Flutter, we can create a `WebSocketChannel`connection to a server:

``` dart 
final channel = IOWebSocketChannel.connect('ws://echo.websocket.org');

```

### 2. Listen for messages from the server

Now that we have established a connection, we can listen for messages from the server, and after we send the message to the test server, it will return the same message.

How do we collect messages and display them? In this example, we will use a [`StreamBuilder`](https://docs.flutter.io/flutter/widgets/StreamBuilder-class.html)to listen for new messages and a Text to display them.

``` dart 
new StreamBuilder(
 stream: widget.channel.stream,
 builder: (context, snapshot) {
   return new Text(snapshot.hasData ? '${snapshot.data}' : '');
 },
);

```

#### working principle

`WebSocketChannel`Provides a message from the server `Stream`. This `Stream`class is `dart:async`a base class package. It provides a way to listen to asynchronous events from data sources. Unlike `Future`returning a single asynchronous response, a `Stream`class can deliver many events over time. This [`StreamBuilder`](https://docs.flutter.io/flutter/widgets/StreamBuilder-class.html)component will connect to one `Stream`and notify Flutter to rebuild the interface every time it receives a message.

### 3. Send data to the server

In order to send data to the server, we will send a `add`message to the `WebSocketChannel`provided sink.

``` dart 
channel.sink.add('Hello!');

```

#### working principle

`WebSocketChannel`One is provided [`StreamSink`](https://docs.flutter.io/flutter/dart-async/StreamSink-class.html)and it sends the message to the server.

`StreamSink`The class provides general methods for adding events to the data source synchronously or asynchronously.

### 4. Close the WebSocket connection

After we use `WebSocket`it, we need to close the connection:

``` dart 
channel.sink.close();

```

### Complete example

``` dart 
import 'package:flutter/material.dart';
import 'package:web_socket_channel/io.dart';

class WebSocketRoute extends StatefulWidget {
 @override
 _WebSocketRouteState createState() => new _WebSocketRouteState();
}

class _WebSocketRouteState extends State<WebSocketRoute> {
 TextEditingController _controller = new TextEditingController();
 IOWebSocketChannel channel;
 String _text = "";


 @override
 void initState() {
   //创建websocket连接
   channel = new IOWebSocketChannel.connect('ws://echo.websocket.org');
 }

 @override
 Widget build(BuildContext context) {
   return new Scaffold(
     appBar: new AppBar(
       title: new Text("WebSocket(内容回显)"),
     ),
     body: new Padding(
       padding: const EdgeInsets.all(20.0),
       child: new Column(
         crossAxisAlignment: CrossAxisAlignment.start,
         children: <Widget>[
           new Form(
             child: new TextFormField(
               controller: _controller,
               decoration: new InputDecoration(labelText: 'Send a message'),
             ),
           ),
           new StreamBuilder(
             stream: channel.stream,
             builder: (context, snapshot) {
               //网络不通会走到这
               if (snapshot.hasError) {
                 _text = "网络不通...";
               } else if (snapshot.hasData) {
                 _text = "echo: "+snapshot.data;
               }
               return new Padding(
                 padding: const EdgeInsets.symmetric(vertical: 24.0),
                 child: new Text(_text),
               );
             },
           )
         ],
       ),
     ),
     floatingActionButton: new FloatingActionButton(
       onPressed: _sendMessage,
       tooltip: 'Send message',
       child: new Icon(Icons.send),
     ),
   );
 }

 void _sendMessage() {
   if (_controller.text.isNotEmpty) {
     channel.sink.add(_controller.text);
   }
 }

 @override
 void dispose() {
   channel.sink.close();
   super.dispose();
 }
}

```

The above example is relatively simple, so I won't repeat it. We now think about a question, if we want to transmit binary data via WebSocket (for example, to receive a picture from the server)? We found that both `StreamBuilder`and `Stream`did not specify the parameters of the receiving type, and there was no corresponding configuration when creating the WebSocket link. It seems that there is no way... In fact, it is very simple. To receive binary data `StreamBuilder`, it is still used , because all the data sent in WebSocket uses frame The frame is sent in a fixed format, and the data type of each frame can be specified by the Opcode field. It can specify whether the current frame is a text type or a binary type (and other types), so the client will already know its data type, so the flutter can fully resolve the correct type after receiving the data, so developers do not need to care about, when the data server is designated as a binary transmission time, `StreamBuilder`the `snapshot.data`type that is text, Then .`List<int>``String`