# 8.3 Event bus

In an APP, we often need a broadcast mechanism for cross-page event notification. For example, in an APP that requires login, the page will pay attention to user login or logout events to perform some status updates. At this time, an event bus will be very useful. The event bus usually implements the subscriber model. The subscriber model includes two roles: publisher and subscriber. Events can be triggered and monitored through the event bus. In this section, we implement a simple For the global event bus, we use the singleton mode, the code is as follows:

```
//订阅者回调签名
typedef void EventCallback(arg);

class EventBus {
  //私有构造函数
  EventBus._internal();

  //保存单例
  static EventBus _singleton = new EventBus._internal();

  //工厂构造函数
  factory EventBus()=> _singleton;

  //保存事件订阅者队列，key:事件名(id)，value: 对应事件的订阅者队列
  var _emap = new Map<Object, List<EventCallback>>();

  //添加订阅者
  void on(eventName, EventCallback f) {
    if (eventName == null || f == null) return;
    _emap[eventName] ??= new List<EventCallback>();
    _emap[eventName].add(f);
  }

  //移除订阅者
  void off(eventName, [EventCallback f]) {
    var list = _emap[eventName];
    if (eventName == null || list == null) return;
    if (f == null) {
      _emap[eventName] = null;
    } else {
      list.remove(f);
    }
  }

  //触发事件，事件触发后该事件所有订阅者会被调用
  void emit(eventName, [arg]) {
    var list = _emap[eventName];
    if (list == null) return;
    int len = list.length - 1;
    //反向遍历，防止订阅者在回调中移除自身带来的下标错位 
    for (var i = len; i > -1; --i) {
      list[i](arg);
    }
  }
}

//定义一个top-level（全局）变量，页面引入该文件后可以直接使用bus
var bus = new EventBus();

```

Use example:

```
//页面A中
...
 //监听登录事件
bus.on("login", (arg) {
  // do something
});

//登录页B中
...
//登录成功后触发登录事件，页面A中订阅者会被调用
bus.emit("login", userInfo);

```

> Note: The standard way to implement singleton mode in Dart is to use static variable + factory constructor, so that you can `new EventBus()`always return the same instance. Readers should understand and master this method.

Event bus is usually used for state sharing between components, but there are also some special packages for state sharing between components, such as redux and the Provider described earlier. For some simple applications, the event bus is sufficient to meet business needs. If you decide to use the state management package, you must think about whether it is really necessary for your APP to prevent "simplification to complex" and over-design.