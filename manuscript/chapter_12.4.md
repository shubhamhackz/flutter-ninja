# 12.4 Plug-in development: Android-side API implementation

In this section, we follow the example of the "Get battery power" plugin in the previous section to complete the implementation of the Android API. The following steps are examples of using Java. If you prefer Kotlin, you can skip directly to the Kotlin section below.

First open the Android part of your Flutter app in Android Studio:

1.  Launch Android Studio
2.  Choose File> Open...
3.  Flutter app to locate your directory, then select the inside of `android`the folder, click OK
4.  `java`Open in the directory`MainActivity.java`

Next, `onCreate`create a MethodChannel and set one in it `MethodCallHandler`. Make sure to use the same name as the channel name used in the Flutter client.

```
import io.flutter.app.FlutterActivity;
import io.flutter.plugin.common.MethodCall;
import io.flutter.plugin.common.MethodChannel;
import io.flutter.plugin.common.MethodChannel.MethodCallHandler;
import io.flutter.plugin.common.MethodChannel.Result;

public class MainActivity extends FlutterActivity {
    private static final String CHANNEL = "samples.flutter.io/battery";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new MethodChannel(getFlutterView(), CHANNEL).setMethodCallHandler(
          new MethodCallHandler() {
             @Override
             public void onMethodCall(MethodCall call, Result result) {
                 // TODO
             }
          });
    }
}

```

Next, we add Java code to use the Android battery API to get the battery level. This code is exactly the same as the code written in the native Android application.

First, add the dependencies that need to be imported.

```
import android.content.ContextWrapper;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.BatteryManager;
import android.os.Build.VERSION;
import android.os.Build.VERSION_CODES;
import android.os.Bundle;

```

Then, add the following new method to the activity class, located below the onCreate method:

```
private int getBatteryLevel() {
  int batteryLevel = -1;
  if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP) {
    BatteryManager batteryManager = (BatteryManager) getSystemService(BATTERY_SERVICE);
    batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY);
  } else {
    Intent intent = new ContextWrapper(getApplicationContext()).
        registerReceiver(null, new IntentFilter(Intent.ACTION_BATTERY_CHANGED));
    batteryLevel = (intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) * 100) /
        intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1);
  }

  return batteryLevel;
}

```

Finally, we complete the `onMethodCall`method we added earlier . We need to process `getBatteryLevel`the call message of the platform method name , so we need to determine whether the called method is the call parameter first `getBatteryLevel`. The implementation of this platform method only needs to call the Android code we wrote in the previous step, and return the response information for success or error through the result parameter. If an undefined API is called, we will also notify the return:

```
@Override
public void onMethodCall(MethodCall call, Result result) {
    if (call.method.equals("getBatteryLevel")) {
        int batteryLevel = getBatteryLevel();

        if (batteryLevel != -1) {
            result.success(batteryLevel);
        } else {
            result.error("UNAVAILABLE", "Battery level not available.", null);
        }
    } else {
        result.notImplemented();
    }
}

```

You can now run the application on Android. If you are using an Android emulator, you can access the battery level in the Extended Controls panel through the "..." button in the toolbar.

### Use Kotlin to add Android platform specific implementation

The steps for using Kotlin and Java are similar. First, open the Android part of your Flutter application in Android Studio:

1.  Launch Android Studio.
2.  选择 the menu item "File > Open…"。
3.  Locate the Flutter app directory, and then select the inside of `android`the folder, click OK.
4.  `kotlin`Open in the catalog `MainActivity.kt`.

Next, `onCreate`create a MethodChannel and set one in it `MethodCallHandler`. Make sure to use the same channel name as used on the Flutter client.

```
import android.os.Bundle
import io.flutter.app.FlutterActivity
import io.flutter.plugin.common.MethodChannel
import io.flutter.plugins.GeneratedPluginRegistrant

class MainActivity() : FlutterActivity() {
  private val CHANNEL = "samples.flutter.io/battery"

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    GeneratedPluginRegistrant.registerWith(this)

    MethodChannel(flutterView, CHANNEL).setMethodCallHandler { call, result ->
      // TODO
    }
  }
}

```

Next, we add Kotlin code and use the Android battery API to get battery power, which is the same as native development.

First, add the dependencies that need to be imported.

```
import android.content.Context
import android.content.ContextWrapper
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager
import android.os.Build.VERSION
import android.os.Build.VERSION_CODES

```

Then, add the following new method to the activity class, located below the onCreate method:

```
  private fun getBatteryLevel(): Int {
    val batteryLevel: Int
    if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP) {
      val batteryManager = getSystemService(Context.BATTERY_SERVICE) as BatteryManager
      batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
    } else {
      val intent = ContextWrapper(applicationContext).registerReceiver(null, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
      batteryLevel = intent!!.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) * 100 / intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
    }

    return batteryLevel
  }

```

Finally, we complete the `onMethodCall`method we added earlier . We need to process `getBatteryLevel`the call message of the platform method name , so we need to determine whether the called method is the call parameter first `getBatteryLevel`. The implementation of this platform method only needs to call the Android code we wrote in the previous step, and return the response information for success or error through the result parameter. If an undefined API is called, we will also notify the return: ​

```
MethodChannel(flutterView, CHANNEL).setMethodCallHandler { call, result ->
  if (call.method == "getBatteryLevel") {
     val batteryLevel = getBatteryLevel()
     if (batteryLevel != -1) {
       result.success(batteryLevel)
     } else {
       result.error("UNAVAILABLE", "Battery level not available.", null)
     }
  } else {
      result.notImplemented()
  }
}

```

You can now run the application on Android. If you are using an Android emulator, you can access the battery level in the Extended Controls panel through the "..." button in the toolbar.