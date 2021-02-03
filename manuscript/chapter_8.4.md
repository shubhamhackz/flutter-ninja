# 8.4 Notification

Notification is an important mechanism in Flutter. In the widget tree, each node can distribute notifications, and the notifications will be passed up the current node, and all parent nodes can `NotificationListener`listen for notifications. Flutter will be notified by this mechanism is transmitted to the parent sub-referred **notification bubble** (Notification Bubbling). Notification bubbling and user touch event bubbling are similar, but there is one difference: notification bubbling can be stopped, but user touch events cannot.

> Notification bubbling is similar to the principle of browser event bubbling in web development. Events are passed from the starting source layer by layer to the upper level. We can listen to notifications/events anywhere on the upper node, and we can also terminate the bubbling process. After being bubbled, the notification will no longer be passed up.

Notifications are used in many places in Flutter. For example, scrollable widgets (Scrollable Widget) will distribute **scroll notifications** (ScrollNotification) when they slide , and Scrollbar determines the position of the scroll bar by listening to ScrollNotification.

The following is an example of listening to scroll notifications of scrollable components:

```
NotificationListener(
  onNotification: (notification){
    switch (notification.runtimeType){
      case ScrollStartNotification: print("开始滚动"); break;
      case ScrollUpdateNotification: print("正在滚动"); break;
      case ScrollEndNotification: print("滚动停止"); break;
      case OverscrollNotification: print("滚动到边界"); break;
    }
  },
  child: ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(title: Text("$index"),);
      }
  ),
);

```

In the above example, scroll notifications such as `ScrollStartNotification`, `ScrollUpdateNotification`etc. are all inherited from the `ScrollNotification`class. Different types of notification subclasses will contain different information. For example, `ScrollUpdateNotification`there is an `scrollDelta`attribute that records the displacement of the movement. Readers of other notification attributes can view the SDK document by themselves.

In the above example, we used `NotificationListener`to monitor `ListView`the scroll notification of the child , which is `NotificationListener`defined as follows:

```
class NotificationListener<T extends Notification> extends StatelessWidget {
  const NotificationListener({
    Key key,
    @required this.child,
    this.onNotification,
  }) : super(key: key);
 ...//省略无关代码 
}

```

We can see that:

1.  `NotificationListener`Inherited from the `StatelessWidget`class, so it can be directly nested into the Widget tree.
    
2.  `NotificationListener`You can specify a template parameter, and the template parameter type must be inherited from `Notification`; when you explicitly specify a template parameter, `NotificationListener`you will only receive notifications of that parameter type. For example, if we change the above example code to:
    
    ```
    //指定监听通知的类型为滚动结束通知(ScrollEndNotification)
    NotificationListener<ScrollEndNotification>(
      onNotification: (notification){
        //只会在滚动结束时才会触发此回调
        print(notification);
      },
      child: ListView.builder(
          itemCount: 100,
          itemBuilder: (context, index) {
            return ListTile(title: Text("$index"),);
          }
      ),
    );
    
    ```
    
    After the above code runs, it will only print out the notification information on the console when the scrolling ends.
    
3.  `onNotification`Callback is a notification processing callback, and its function signature is as follows:
    
    ```
    typedef NotificationListenerCallback<T extends Notification> = bool Function(T notification);
    
    ```
    
    Its return value type is boolean. When the return value `true`is to prevent bubbling, its parent Widget will never receive the notification again; when the return value is `false`, it continues to bubble up notifications.
    

Flutter UI framework to achieve, in addition to emit during scrolling in a scrollable components `ScrollNotification`, there are some other notification, such as `SizeChangedLayoutNotification`, `KeepAliveNotification`, `LayoutChangedNotification`and the like, Flutter is the parent element to make the notification mechanism in this specific time Come do something.

#### Custom notification

In addition to Flutter internal notifications, we can also customize notifications. Let's see how to implement custom notifications:

1.  Define a notification class, which must inherit from the Notification class;
    
    ```
    class MyNotification extends Notification {
      MyNotification(this.msg);
      final String msg;
    }
    
    ```
    
2.  Distribution notice.
    
    `Notification`There is a `dispatch(context)`method, which is used to distribute notifications. We said that `context`it is actually `Element`an interface of operation . It `Element`corresponds to the node on the tree, and the notification will bubble up from the `context`corresponding `Element`node.
    

Let's look at a complete example:

```
class NotificationRoute extends StatefulWidget {
  @override
  NotificationRouteState createState() {
    return new NotificationRouteState();
  }
}

class NotificationRouteState extends State<NotificationRoute> {
  String _msg="";
  @override
  Widget build(BuildContext context) {
    //监听通知  
    return NotificationListener<MyNotification>(
      onNotification: (notification) {
        setState(() {
          _msg+=notification.msg+"  ";
        });
       return true;
      },
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
//          RaisedButton(
//           onPressed: () => MyNotification("Hi").dispatch(context),
//           child: Text("Send Notification"),
//          ),  
            Builder(
              builder: (context) {
                return RaisedButton(
                  //按钮点击时分发通知  
                  onPressed: () => MyNotification("Hi").dispatch(context),
                  child: Text("Send Notification"),
                );
              },
            ),
            Text(_msg)
          ],
        ),
      ),
    );
  }
}

class MyNotification extends Notification {
  MyNotification(this.msg);
  final String msg;
}

```

In the above code, we will distribute a `MyNotification`type of notification every time we click the button . We listen to the notification on the Widget root. After receiving the notification, we will display the notification on the screen through Text.

> Note: The commented part of the code does not work properly, because this `context`is the root Context, and NotificationListener is the subtree of monitoring, so we `Builder`build RaisedButton to get the context of the button position.

The running effect is shown in Figure 8-6:

![Figure 8-6](https://pcdn.flutterchina.club/imgs/8-6.png)

### Stop bubbling

We change the above example to:

```
class NotificationRouteState extends State<NotificationRoute> {
  String _msg="";
  @override
  Widget build(BuildContext context) {
    //监听通知
    return NotificationListener<MyNotification>(
      onNotification: (notification){
        print(notification.msg); //打印通知
        return false;
      },
      child: NotificationListener<MyNotification>(
        onNotification: (notification) {
          setState(() {
            _msg+=notification.msg+"  ";
          });
          return false; 
        },
        child: ...//省略重复代码
      ),
    );
  }
}

```

The two `NotificationListener`in the above column are nested, and `NotificationListener`the `onNotification`callback of the child returns `false`, which means that the bubbling is not prevented, so the parent `NotificationListener`will still be notified, so the console will print out the notification information; if the return value `NotificationListener`of the `onNotification`callback of the child is changed to `true`, then The parent `NotificationListener`will no longer print the notification because the child `NotificationListener`has stopped the notification bubbling.

### Notification bubble principle

We introduced the phenomenon and use of notification bubbling above. Now we go a little deeper and introduce how notification bubbling is implemented in the Flutter framework. In order to clarify this problem, you must look at the source code. We start from the source of the notification distribution, and then follow the vine. Since the notification is sent by `Notification`the `dispatch(context)`method, let's take a look at what `dispatch(context)`is done in the method first . Here is the relevant source code:

```
void dispatch(BuildContext target) {
  target?.visitAncestorElements(visitAncestor);
}

```

`dispatch(context)`The method of the current context is called in `visitAncestorElements`, which will traverse the parent element from the current Element upwards; `visitAncestorElements`there is a traversal callback parameter, which will be executed for the parent element traversed during the traversal process. The termination condition of the traversal is: the root Element has been traversed or a traversal callback returns `false`. `visitAncestorElements`The traversal callback passed to the method in the source code is the `visitAncestor`method. Let's look at `visitAncestor`the implementation of the method:

```
//遍历回调，会对每一个父级Element执行此回调
bool visitAncestor(Element element) {
  //判断当前element对应的Widget是否是NotificationListener。

  //由于NotificationListener是继承自StatelessWidget，
  //故先判断是否是StatelessElement
  if (element is StatelessElement) {
    //是StatelessElement，则获取element对应的Widget，判断
    //是否是NotificationListener 。
    final StatelessWidget widget = element.widget;
    if (widget is NotificationListener<Notification>) {
      //是NotificationListener，则调用该NotificationListener的_dispatch方法
      if (widget._dispatch(this, element)) 
        return false;
    }
  }
  return true;
}

```

`visitAncestor`It will judge whether each traversed parent Widget is `NotificationListener`, if not, return `true`to continue traversing upwards, if it is, then call `NotificationListener`the `_dispatch`method, we look at `_dispatch`the source code of the method:

```
  bool _dispatch(Notification notification, Element element) {
    // 如果通知监听器不为空，并且当前通知类型是该NotificationListener
    // 监听的通知类型，则调用当前NotificationListener的onNotification
    if (onNotification != null && notification is T) {
      final bool result = onNotification(notification);
      // 返回值决定是否继续向上遍历
      return result == true; 
    }
    return false;
  }

```

We can see that `NotificationListener`the `onNotification`callback is finally `_dispatch`executed in the method, and then based on the return value to determine whether to continue to bubble up. The implementation of the above source code is actually not complicated. By reading these source codes, readers can pay attention to some additional points:

1.  `Context`The above also provides a method to traverse the Element tree.
2.  We can `Element.widget`get `element`the widget corresponding to the node; we have repeatedly talked about the correspondence between Widget and Element, and readers can use these source codes to deepen their understanding.

### to sum up

Flutter implements a set of low-to-up message transmission mechanism through notification bubbling. This is similar to the principle of event bubbling in browsers in web development. Web developers can learn by analogy. In addition, we have learned about the process and principle of Flutter notification bubbling through the source code, which is convenient for readers to deepen their understanding and study of Flutter's framework design ideas. Here, we again recommend that readers can read the source code more in their daily studies, which will definitely benefit a lot.