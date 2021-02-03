# 12.3 Develop Flutter plugin

Below we introduce the development process of the Flutter plug-in through a plug-in for obtaining battery power. In this plug-in, we `getBatteryLevel`call Android `BatteryManager`API and iOS `device.batteryLevel`API in Dart .

### Create a new application project

First create a new application:

-   Run in the terminal:`flutter create batterylevel`

By default, the template supports writing Android code in Java or writing iOS code in Objective-C. To use Kotlin or Swift, use the -i and/or -a flags:

-   Run in terminal: `flutter create -i swift -a kotlin batterylevel`

### Create Flutter platform client

The application `State`class has the current application state. We need to extend this to maintain the current charge

First, we build the channel. We use to `MethodChannel`call a method to return the battery level.

The client and host of the channel are connected by the channel name passed in the channel constructor. All channel names used in a single application must be unique; we recommend adding a unique "domain name prefix" before the channel name, for example `samples.flutter.io/battery`.

```
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
...
class _MyHomePageState extends State<MyHomePage> {
  static const platform = const MethodChannel('samples.flutter.io/battery');

  // Get battery level.
}

```

Next, we call the method on the channel, specifying the method to be called via the string identifier `getBatteryLevel`. The call may fail (the platform does not support the platform API, such as when running in the simulator), so we wrap the invokeMethod call in a try-catch statement.

We use the returned results `setState`to update the user interface status in `batteryLevel`.

```
  // Get battery level.
  String _batteryLevel = 'Unknown battery level.';

  Future<Null> _getBatteryLevel() async {
    String batteryLevel;
    try {
      final int result = await platform.invokeMethod('getBatteryLevel');
      batteryLevel = 'Battery level at $result % .';
    } on PlatformException catch (e) {
      batteryLevel = "Failed to get battery level: '${e.message}'.";
    }

    setState(() {
      _batteryLevel = batteryLevel;
    });
  }

```

Finally, we create a user interface in the build that contains a small font to display the battery status and a button to refresh the value.

```
@override
Widget build(BuildContext context) {
  return new Material(
    child: new Center(
      child: new Column(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: [
          new RaisedButton(
            child: new Text('Get Battery Level'),
            onPressed: _getBatteryLevel,
          ),
          new Text(_batteryLevel),
        ],
      ),
    ),
  );
}

```

So far the Flutter part of the test code is written. Next, we need to implement the APIs under the Android and iOS platforms. Because the platform API implementation part is relatively large, we will introduce the Android and iOS APIs in the next two sections. achieve.