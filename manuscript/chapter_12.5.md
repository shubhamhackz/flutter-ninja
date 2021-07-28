# 12.5 Plug-in development: iOS API implementation

In this section, we follow the example of the "obtain battery power" plug-in to complete the implementation of the iOS API. The following steps use Objective-C. If you prefer Swift, you can skip directly to the following Swift part.

First open the iOS part of the Flutter app in Xcode:

1.  Start Xcode
2.  Choose File> Open...
3.  Flutter app to locate your directory, then select the inside of `iOS`the folder, click OK
4.  Make sure that the Xcode project is built without errors.
5.  Select Runner> Runner, open`AppDelegate.m`

Next, `application didFinishLaunchingWithOptions:`create one inside the method `FlutterMethodChannel`and add a processing method. Make sure it is the same as the channel name used on the Flutter client.

``` dart 
#import <Flutter/Flutter.h>

@implementation AppDelegate
- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
 FlutterViewController* controller = (FlutterViewController*)self.window.rootViewController;

 FlutterMethodChannel* batteryChannel = [FlutterMethodChannel
                                         methodChannelWithName:@"samples.flutter.io/battery"
                                         binaryMessenger:controller];

 [batteryChannel setMethodCallHandler:^(FlutterMethodCall* call, FlutterResult result) {
   // TODO
 }];

 return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

```

Next, we add Objective-C code and use the iOS battery API to get battery power, which is the same as native.

In `AppDelegate`adding the following new methods in the class:

``` dart 
- (int)getBatteryLevel {
 UIDevice* device = UIDevice.currentDevice;
 device.batteryMonitoringEnabled = YES;
 if (device.batteryState == UIDeviceBatteryStateUnknown) {
   return -1;
 } else {
   return (int)(device.batteryLevel * 100);
 }
}

```

Finally, we complete the `setMethodCallHandler`method we added earlier . The platform method we need to handle is named `getBatteryLevel`, so we need to determine whether it is in the call parameter `getBatteryLevel`. The implementation of this platform method only needs to call the iOS code we wrote in the previous step, and use the result parameter to return a success or error response. If an undefined API is called, we will also notify the return:

``` dart 
[batteryChannel setMethodCallHandler:^(FlutterMethodCall* call, FlutterResult result) {
 if ([@"getBatteryLevel" isEqualToString:call.method]) {
   int batteryLevel = [self getBatteryLevel];

   if (batteryLevel == -1) {
     result([FlutterError errorWithCode:@"UNAVAILABLE"
                                message:@"电池信息不可用"
                                details:nil]);
   } else {
     result(@(batteryLevel));
   }
 } else {
   result(FlutterMethodNotImplemented);
 }
}];

```

Now you can run the application on iOS. If you are using the iOS simulator, please note that it does not support the battery API, so the application will display "Battery information is not available".

### Implement iOS API using Swift

The following steps are similar to using Objective-C above. First, open the iOS part of the Flutter application in Xcode:

1.  Start Xcode
2.  Choose File> Open...
3.  Flutter app to locate your directory, then select the inside of `ios`the folder, click OK
4.  Make sure that the Xcode project is built without errors.
5.  Select Runner> Runner, and then open`AppDelegate.swift`

Next, override the application method and create a `FlutterMethodChannel`binding channel name `samples.flutter.io/battery`:

``` dart 
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
 override func application(
   _ application: UIApplication,
   didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
   GeneratedPluginRegistrant.register(with: self);

   let controller : FlutterViewController = window?.rootViewController as! FlutterViewController;
   let batteryChannel = FlutterMethodChannel.init(name: "samples.flutter.io/battery",
                                                  binaryMessenger: controller);
   batteryChannel.setMethodCallHandler({
     (call: FlutterMethodCall, result: FlutterResult) -> Void in
     // Handle battery messages.
   });

   return super.application(application, didFinishLaunchingWithOptions: launchOptions);
 }
}

```

Next, we add Swift code and use iOS battery API to get battery power, which is the same as native development.

Add the following new methods to the `AppDelegate.swift`bottom:

``` dart 
private func receiveBatteryLevel(result: FlutterResult) {
 let device = UIDevice.current;
 device.isBatteryMonitoringEnabled = true;
 if (device.batteryState == UIDeviceBatteryState.unknown) {
   result(FlutterError.init(code: "UNAVAILABLE",
                            message: "电池信息不可用",
                            details: nil));
 } else {
   result(Int(device.batteryLevel * 100));
 }
}

```

Finally, we complete the `setMethodCallHandler`method we added earlier . The platform method we need to handle is named `getBatteryLevel`, so we need to determine whether it is in the call parameter `getBatteryLevel`. The implementation of this platform method only needs to call the iOS code we wrote in the previous step, and use the result parameter to return a success or error response. If an undefined API is called, we will also notify the return:

``` dart 
batteryChannel.setMethodCallHandler({
 (call: FlutterMethodCall, result: FlutterResult) -> Void in
 if ("getBatteryLevel" == call.method) {
   receiveBatteryLevel(result: result);
 } else {
   result(FlutterMethodNotImplemented);
 }
});

```

Now you can run the application on iOS. If you are using the iOS simulator, please note that it does not support the battery API, so the application will display "Battery information is not available".