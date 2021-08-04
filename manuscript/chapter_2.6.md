# 2.6 Flutter exception capture

Before introducing Flutter exception capture, we must first understand the Dart single-threaded model. Only by understanding Dart's code execution flow can we know where to catch exceptions.

## 2.6.1 Dart single-threaded model

In Java and Objective-C (hereinafter referred to as "OC"), if an exception occurs in the program and it is not caught, the program will terminate, but this is not true in Dart or JavaScript! Investigate the reason, this has something to do with their operating mechanism. Both Java and OC are programming languages ​​with a multithreaded model. When any thread triggers an exception and the exception is not caught, it will cause the entire process to exit. But Dart and JavaScript do not. They are both single-threaded models, and their operating mechanisms are very similar (but there are differences). Let's take a look at the general operating principle of Dart through a diagram provided by Dart:

![Figure 2-12](../resources/imgs/2-12.png)

Dart operates in a single-threaded message loop mechanism, which contains two task queues, one is the " **microtask queue** " **microtask queue** , and the other is called the "event queue" **event queue** . It can be found from the figure that the execution priority of the micro task queue is higher than that of the event queue.

Now let's introduce the Dart thread running process, as shown in the figure above, after the entry function main() is executed, the message loop mechanism is started. First, the tasks in the microtask queue will be executed one by one in a first-in, first-out order. After the event task is executed, the program will exit. However, new microtasks and event tasks can also be inserted during the execution of the event task. In this case, the execution process of the entire thread is always looping and will not exit, while in Flutter, the execution process of the main thread is like this and never terminates.

In Dart, all external event tasks are in the event queue, such as IO, timer, click, and drawing events. Microtasks usually come from within Dart, and there are very few microtasks. This is because micro The priority of the task queue is high. If there are too many microtasks, the longer the total execution time will be, and the longer the delay of the event queue task. The most intuitive performance for GUI applications is the comparison card, so it must be ensured that the microtask queue will not be too long. long. It is worth noting that we can `Future.microtask(…)`insert a task into the micro task queue through the method.

In the event loop, when an exception occurs in a task and is not caught, the program will not exit, and the direct result is **that** the subsequent code of the **current task** will not be executed, which means that the exception in a task is Will not affect the execution of other tasks.

## 2.6.2 Flutter exception capture

Dart can be `try/catch/finally`used to capture code block exceptions. This is similar to other programming languages. If the reader is not clear, you can check the Dart language documentation and not repeat them. Let's take a look at exception capture in Flutter.

### Flutter framework exception capture

The Flutter framework captures exceptions in many key methods for us. Here is an example. When our layout is out of bounds or out of specification, Flutter will automatically pop up an error interface. This is because Flutter has added exception capture when executing the build method. The final source code is as follows:

``` dart 
@override
void performRebuild() {
...
 try {
   //执行build方法  
   built = build();
 } catch (e, stack) {
   // 有异常时则弹出错误提示  
   built = ErrorWidget.builder(_debugReportException('building $this', e, stack));
 } 
 ...
}

```

As you can see, when an exception occurs, Flutter's default processing method is to pop an ErrorWidget, but what should we do if we want to catch the exception and report it to the alarm platform? Let's `_debugReportException()`take a look at the method:

``` dart 
FlutterErrorDetails _debugReportException(
 String context,
 dynamic exception,
 StackTrace stack, {
 InformationCollector informationCollector
}) {
 //构建错误详情对象  
 final FlutterErrorDetails details = FlutterErrorDetails(
   exception: exception,
   stack: stack,
   library: 'widgets library',
   context: context,
   informationCollector: informationCollector,
 );
 //报告错误 
 FlutterError.reportError(details);
 return details;
}

```

We found that the error was `FlutterError.reportError`reported through the method, continue to track:

``` dart 

static void reportError(FlutterErrorDetails details) {
 ...
 if (onError != null)
   onError(details); //调用了onError回调
}

```

We found that it `onError`is `FlutterError`a static property, which has a default processing method `dumpErrorToConsole`, which is clear here. If we want to report exceptions ourselves, we only need to provide a custom error handling callback, such as:

``` dart 
void main() {
 FlutterError.onError = (FlutterErrorDetails details) {
   reportError(details);
 };
...
}

```

So that we can handle the exceptions that Flutter caught for us, let's see how to catch other exceptions.

### Other exception capture and log collection

In Flutter, there are some exceptions that Flutter did not catch for us, such as calling empty object method exceptions and exceptions in Future. In Dart, there are two types of exceptions: synchronous exceptions and asynchronous exceptions. Synchronous exceptions can be `try/catch`caught, while asynchronous exceptions are more troublesome. For example, the following code cannot catch `Future`exceptions:

``` dart 
try{
   Future.delayed(Duration(seconds: 1)).then((e) => Future.error("xxx"));
}catch (e){
   print(e)
}

```

There is a `runZoned(...)`method in Dart that can assign a Zone to the execution object. Zone represents the scope of a code execution environment. For ease of understanding, readers can compare Zone to a code execution sandbox. Different sandboxes are isolated. The sandbox can capture, intercept or modify some code behaviors, such as in Zone. The behavior of log output, Timer creation, and micro task scheduling can be captured, and Zone can also capture all unhandled exceptions. Let's look at the `runZoned(...)`method definition below :

``` dart 
R runZoned<R>(R body(), {
   Map zoneValues, 
   ZoneSpecification zoneSpecification,
   Function onError,
})

```

-   `zoneValues`: Zone's private data can be obtained through examples `zone[key]`, which can be understood as the private data of each "sandbox".
   
-   `zoneSpecification`: Some configuration of Zone, you can customize some code behaviors, such as intercepting log output behaviors, for example:
   
   The following is `print`the behavior of intercepting all call output logs in the application .
   
``` dart 
   main() {
     runZoned(() => runApp(MyApp()), zoneSpecification: new ZoneSpecification(
         print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
           parent.print(zone, "Intercepted: $line");
         }),
     );
   }
   
```
   
   In this way, all `print`the behaviors of calling methods in our APP to output logs will be intercepted. In this way, we can also record logs in the application. When the application triggers an uncaught exception, the exception information and log will be reported uniformly. ZoneSpecification can also customize some other behaviors, and readers can view the API documentation.
   
-   `onError`: Uncaught exception handling callback in Zone. If the developer provides an onError callback or `ZoneSpecification.handleUncaughtError`specifies an error handling callback, then this zone will become an error-zone. An uncaught exception occurs in the error-zone (regardless of synchronous or asynchronous ) Will call the callback provided by the developer, such as:
   
``` dart 
   runZoned(() {
       runApp(MyApp());
   }, onError: (Object obj, StackTrace stack) {
       var details=makeDetails(obj,stack);
       reportError(details);
   });
   
```
   
   In this way, combined with the above, `FlutterError.onError`we can catch all the errors in our Flutter application! It should be noted that errors that occur within the error-zone will not cross the boundary of the current error-zone. If you want to capture exceptions across the error-zone boundary, you can capture them through a common "source" zone, such as:
   
``` dart 
   var future = new Future.value(499);
   runZoned(() {
       var future2 = future.then((_) { throw "error in first error-zone"; });
       runZoned(() {
           var future3 = future2.catchError((e) { print("Never reached!"); });
       }, onError: (e) { print("unused error handler"); });
   }, onError: (e) { print("catches error of first error-zone."); });
   
```
   

### to sum up

Our final exception capture and reporting code is roughly as follows:

``` dart 
void collectLog(String line){
   ... //收集日志
}
void reportErrorAndLog(FlutterErrorDetails details){
   ... //上报错误和日志逻辑
}

FlutterErrorDetails makeDetails(Object obj, StackTrace stack){
   ...// 构建错误信息
}

void main() {
 FlutterError.onError = (FlutterErrorDetails details) {
   reportErrorAndLog(details);
 };
 runZoned(
   () => runApp(MyApp()),
   zoneSpecification: ZoneSpecification(
     print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
       collectLog(line); // 收集日志
     },
   ),
   onError: (Object obj, StackTrace stack) {
     var details = makeDetails(obj, stack);
     reportErrorAndLog(details);
   },
 );
}

```
