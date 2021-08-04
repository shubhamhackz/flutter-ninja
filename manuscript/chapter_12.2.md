# 12.2 Plug-in development: introduction to platform channels

The platform in "platform specific" or "specific platform" refers to the platform on which the Flutter application runs, such as Android or IOS. We know that a complete Flutter application actually includes two parts: native code and Flutter code. Since Flutter itself is only a UI system, it cannot provide some system capabilities, such as using Bluetooth, camera, GPS, etc., so in order to call these capabilities in the Flutter APP, it must communicate with the native platform. To this end, a platform channel is provided in Flutter for communication between Flutter and the native platform. The platform channel is the communication bridge between Flutter and native, and it is also the underlying infrastructure of Flutter plugins.

Flutter uses a flexible system that allows you to call platform-specific APIs, whether in Java or Kotlin code on Android, or in ObjectiveC or Swift code on iOS.

The communication between Flutter and Native relies on flexible message passing methods:

-   The Flutter part of the application sends messages to the host (iOS or Android) application (native application) where the application is located through the platform channel.
-   The host monitors the platform channel and receives the message. It then calls the platform's API and sends the response back to the client, the Flutter part of the application.

### Platform channel

Use platform channels to pass messages between Flutter (client) and native (host), as shown in the following figure:

![Platform channel](../resources/imgs/12-3.png)

When calling a native method in Flutter, the calling information is passed to the native through the platform channel, and the native can perform the specified operation after receiving the calling information. If the data needs to be returned, the native will pass the data to Flutter through the platform channel. It is worth noting that message delivery is asynchronous, which ensures that the user interface will not be suspended during message delivery.

On the client side, the [MethodChannel API](https://docs.flutter.io/flutter/services/MethodChannel-class.html) can send messages corresponding to method calls. On the host platform, `MethodChannel`the [Android API](https://docs.flutter.io/javadoc/io/flutter/plugin/common/MethodChannel.html) and [FlutterMethodChannel iOS API](https://docs.flutter.io/objcdoc/Classes/FlutterMethodChannel.html) can receive method calls and return results. These classes can help us develop platform plug-ins with very little code.

> **Note** : If necessary, the method call (messaging) can be reversed, that is, the host calls the API implemented in Dart as a client. [`quick_actions`](https://pub.dartlang.org/packages/quick_actions)Plug-ins are a concrete example.

### Platform channel data type support

The platform channel uses a standard message encoder/decoder to encode and decode messages, which can efficiently perform binary serialization and deserialization of messages. Due to the differences in data types between Dart and native platforms, below we list the mapping relationships between data types.
| Dart                      | Android              | IOS                                            |
| ------------------------- | -------------------- | ---------------------------------------------- |
| null                      | null                 | nil (NSNull when nested)                       |
| bool                      | java.lang.Boolean    | NSNumber numberWithBool:                       |
| int                       | java.lang.Integer    | NSNumber numberWithInt:                        |
| int, if less than 32 bits | java.lang.Long       | NSNumber numberWithLong:                       |
| int, if less than 64 bits | java.math.BigInteger | FlutterStandardBigInteger                      |
| double                    | java.lang.Double     | NSNumber numberWithDouble:                     |
| String                    | java.lang.String     | NSString                                       |
| Uint8List                 | byte[]               | FlutterStandardTypedData typedDataWithBytes:   |
| Int32List                 | int[]                | FlutterStandardTypedData typedDataWithInt32:   |
| Int64List                 | long[]               | FlutterStandardTypedData typedDataWithInt64:   |
| Float64List               | double[]             | FlutterStandardTypedData typedDataWithFloat64: |
| List                      | java.util.ArrayList  | NSArray                                        |
| Map                       | java.util.HashMap    | NSDictionary                                   |


When sending and receiving values, the serialization and deserialization of these values ​​in the message will proceed automatically.

### Custom codec

In addition to the above mentioned `MethodChannel`, it can also be used [`BasicMessageChannel`](https://docs.flutter.io/flutter/services/BasicMessageChannel-class.html), it supports the use of custom message codecs for basic asynchronous messaging. In addition, you can use specialized [`BinaryCodec`](https://docs.flutter.io/flutter/services/BinaryCodec-class.html), [`StringCodec`](https://docs.flutter.io/flutter/services/StringCodec-class.html)and [`JSONMessageCodec`](https://docs.flutter.io/flutter/services/JSONMessageCodec-class.html)class, or create your own codec.

### How to obtain platform information

Flutter provides a global variable `defaultTargetPlatform`to obtain the platform information of the current application, which is `defaultTargetPlatform`defined in "platform.dart". Its type is `TargetPlatform`, which is an enumeration class, defined as follows:

``` dart 
enum TargetPlatform {
 android,
 fuchsia,
 iOS,
}

```

It can be seen that Flutter currently only supports these three platforms. We can judge the platform by the following code:

``` dart 
if(defaultTargetPlatform==TargetPlatform.android){
 // 是安卓系统，do something
 ...
}
...

```

Because different platforms have their own interaction specifications, some components in the Flutter Material library are adapted to the corresponding platforms, such as the routing component `MaterialPageRoute`, which applies the switching animation of the respective platform specifications in android and ios. What if we want our APP to behave consistently on all platforms, for example, if we want to switch animations on all platforms to follow the same left and right sliding switching style of the ios platform? Flutter provides a mechanism to override the default platform. We can `debugDefaultTargetPlatformOverride`specify the application platform by explicitly specifying the value of the global variable. such as:

``` dart 
debugDefaultTargetPlatformOverride=TargetPlatform.iOS;
print(defaultTargetPlatform); // 会输出TargetPlatform.iOS

```

After the above code runs in Android, the Flutter APP will think that the current system is iOS, and all the component interaction methods in the Material component library will be aligned with `defaultTargetPlatform`the iOS platform, and the value will also change `TargetPlatform.iOS`.