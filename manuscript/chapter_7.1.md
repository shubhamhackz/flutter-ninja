# 7.1 Navigation back interception (WillPopScope)

In order to prevent the user from accidentally touching the back button and causing the APP to exit, in many APPs, the user's click on the back button is blocked, and then some anti-error judgments are made. For example, when the user clicks twice in a certain period of time, it will Think that the user wants to log out (not touch by mistake). Flutter can be `WillPopScope`used to achieve the return button interception, let's take a look at `WillPopScope`the default constructor:

``` dart 
const WillPopScope({
 ...
 @required WillPopCallback onWillPop,
 @required Widget child
})

```

`onWillPop`It is a callback function that is called when the user clicks the return button (including the navigation return button and the Android physical return button). The callback needs to return an `Future`object. If the `Future`final value returned is `false`time, the current route will not pop out of the stack (no return); `true`when the final value is time, the current route will exit the stack. We need to provide this callback to decide whether to exit.

### Example

In order to prevent the user from accidentally touching the return key to exit, we intercept the return event. When the user clicks the return button twice within 1 second, it will exit; if the interval exceeds 1 second, it will not exit and the time will be recorded again. code show as below:

``` dart 
import 'package:flutter/material.dart';

class WillPopScopeTestRoute extends StatefulWidget {
 @override
 WillPopScopeTestRouteState createState() {
   return new WillPopScopeTestRouteState();
 }
}

class WillPopScopeTestRouteState extends State<WillPopScopeTestRoute> {
 DateTime _lastPressedAt; //上次点击时间

 @override
 Widget build(BuildContext context) {
   return new WillPopScope(
       onWillPop: () async {
         if (_lastPressedAt == null ||
             DateTime.now().difference(_lastPressedAt) > Duration(seconds: 1)) {
           //两次点击间隔超过1秒则重新计时
           _lastPressedAt = DateTime.now();
           return false;
         }
         return true;
       },
       child: Container(
         alignment: Alignment.center,
         child: Text("1秒内连续按两次返回键退出"),
       )
   );
 }
}

```

Readers can run the example to see the effect.