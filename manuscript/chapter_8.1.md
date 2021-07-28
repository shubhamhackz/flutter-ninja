# 8.1 Raw pointer event handling

This section first introduces the original pointer event (Pointer Event, usually touch events on mobile devices), and then introduces gesture processing in the next section.

On the mobile side, the original pointer event model of each platform or UI system is basically the same, that is: a complete event is divided into three stages: finger press, finger movement, and finger lift, and higher-level gestures ( Such as clicking, double-clicking, dragging, etc.) are based on these primitive events.

When the pointer is pressed, Flutter will perform a **hit test (Hit Test)** on the application to determine which components (widgets) exist where the pointer contacts the screen. The pointer down event (and subsequent events of the pointer) are then distributed to The innermost component found by the hit test, and from there, events will bubble up in the component tree. These events will be distributed from the innermost component to all components on the path of the component tree root. This is similar to Web development. The **event bubbling** mechanism of the browser in Flutter is similar, but there is no mechanism in Flutter to cancel or stop the "bubbling" process, and the bubbling of the browser can be stopped. Note that only components that pass the hit test can trigger events.

Flutter can be used `Listener`to monitor raw touch events. According to the classification of components in this book, it `Listener`is also a functional component. The following is `Listener`the constructor definition:

``` dart 
Listener({
 Key key,
 this.onPointerDown, //手指按下回调
 this.onPointerMove, //手指移动回调
 this.onPointerUp,//手指抬起回调
 this.onPointerCancel,//触摸事件取消回调
 this.behavior = HitTestBehavior.deferToChild, //在命中测试期间如何表现
 Widget child
})

```

Let's look at an example first, and then discuss `behavior`properties separately .

``` dart 
...
//定义一个状态，保存当前指针位置
PointerEvent _event;
...
Listener(
 child: Container(
   alignment: Alignment.center,
   color: Colors.blue,
   width: 300.0,
   height: 150.0,
   child: Text(_event?.toString()??"",style: TextStyle(color: Colors.white)),
 ),
 onPointerDown: (PointerDownEvent event) => setState(()=>_event=event),
 onPointerMove: (PointerMoveEvent event) => setState(()=>_event=event),
 onPointerUp: (PointerUpEvent event) => setState(()=>_event=event),
),

```

The effect after running is shown in Figure 8-1:

![Figure 8-1](https://pcdn.flutterchina.club/imgs/8-1.png)

Moving a finger to see the current pointer offset in the blue rectangular area, when a triggering event pointer, parameters `PointerDownEvent`, `PointerMoveEvent`, `PointerUpEvent`is `PointerEvent`a subclass of `PointerEvent`class includes some information about the current pointer, such as:

-   `position`: It is the offset of the mouse relative to the current global coordinate.
-   `delta`: `PointerMoveEvent`The distance between two pointer movement events ( ).
-   `pressure`: Press force. If the phone screen supports a pressure sensor (such as iPhone’s 3D Touch), this attribute will be more meaningful. If the phone does not support it, it will always be 1.
-   `orientation`: The pointer movement direction is an angle value.

The above are just `PointerEvent`some commonly used attributes, in addition to these there are many attributes, readers can check the API documentation.

Now, let's focus on the `behavior`attribute, which determines how the child component responds to the hit test. Its value type is `HitTestBehavior`, which is an enumeration class with three enumeration values:

-   `deferToChild`: The child components will perform hit tests one by one. If there is a test in the child component, the current component will pass, which means that if the pointer event acts on the child component, its parent component can definitely receive it The event.
   
-   `opaque`: In the hit test, the current component is treated as opaque (even if it is transparent), the final effect is equivalent to the entire area of ​​the current Widget is the click area. for example:
   
``` dart 
   Listener(
       child: ConstrainedBox(
           constraints: BoxConstraints.tight(Size(300.0, 150.0)),
           child: Center(child: Text("Box A")),
       ),
       //behavior: HitTestBehavior.opaque,
       onPointerDown: (event) => print("down A")
   ),
   
```
   
   The example above, only click on the text area will trigger a click event, as `deferToChild`will go sub-assembly test to determine whether a hit, but this example is the neutron component `Text("Box A")`. If we want the entire 300×150 rectangular area to be clickable, we can `behavior`set it `HitTestBehavior.opaque`. Note that this attribute cannot be used to intercept (ignore) events in the component tree, it just determines the size of the component when the test is hit.
   
-   `translucent`: When the transparent area of ​​the component is clicked, hit test can be performed on both the inner boundary and the visible area at the bottom. This means that when the transparent area of ​​the top component is clicked, both the top and bottom components can receive events, for example:
   
``` dart 
   Stack(
     children: <Widget>[
       Listener(
         child: ConstrainedBox(
           constraints: BoxConstraints.tight(Size(300.0, 200.0)),
           child: DecoratedBox(
               decoration: BoxDecoration(color: Colors.blue)),
         ),
         onPointerDown: (event) => print("down0"),
       ),
       Listener(
         child: ConstrainedBox(
           constraints: BoxConstraints.tight(Size(200.0, 100.0)),
           child: Center(child: Text("左上角200*100范围内非文本区域点击")),
         ),
         onPointerDown: (event) => print("down1"),
         //behavior: HitTestBehavior.translucent, //放开此行注释后可以"点透"
       )
     ],
   )
   
```
   
   In the above example, when the last line of code is commented out, when the non-text area is clicked in the upper left corner 200*100 (transparent area of ​​the top component), the console will only print "down0", which means that the top component does not receive the event , And only the bottom received it. When the comment is released, the top and bottom will receive the event when clicked again, and it will print at this time:
   
``` dart 
   I/flutter ( 3039): down1
   I/flutter ( 3039): down0
   
```
   
   If the `behavior`value is changed `HitTestBehavior.opaque`, only "down1" will be printed.
   

### Ignore PointerEvent

If we don’t want a subtree to respond `PointerEvent`, we can use `IgnorePointer`and `AbsorbPointer`. These two components can prevent the subtree from receiving pointer events. The difference is that `AbsorbPointer`they will participate in the hit test, but `IgnorePointer`they will not participate, which means `AbsorbPointer`they are You may receive pointer events (but not its subtree), but `IgnorePointer`not. A simple example is as follows:

``` dart 
Listener(
 child: AbsorbPointer(
   child: Listener(
     child: Container(
       color: Colors.red,
       width: 200.0,
       height: 100.0,
     ),
     onPointerDown: (event)=>print("in"),
   ),
 ),
 onPointerDown: (event)=>print("up"),
)

```

When clicked `Container`, because it is on `AbsorbPointer`the subtree, it will not respond to pointer events, so the log will not output "in", but `AbsorbPointer`it can receive pointer events, so it will output "up". If it is `AbsorbPointer`replaced `IgnorePointer`, neither will be output.