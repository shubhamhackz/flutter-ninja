# 7.3 Cross-component state sharing (Provider)

In Flutter development, state management is an eternal topic. The general principle is: if the state is private to the component, it should be managed by the component itself; if the state is to be shared across components, the state should be managed by the common parent element of each component. It is easy to understand the private state management of components, but there are more ways to manage the state shared across components, such as using the global event bus EventBus (which will be introduced in the next chapter), which is an implementation of the observer mode. Through it, cross-component state synchronization can be realized: the state holder (publisher) is responsible for updating and publishing the state, and the state consumer (observer) listens for state change events to perform some operations. Let's look at a simple example of login status synchronization:

Define the event:

```
enum Event{
  login,
  ... //省略其它事件
}

```

The login page code is roughly as follows:

```
// 登录状态改变后发布状态改变事件
bus.emit(Event.login);

```

Pages that depend on login status:

```
void onLoginChanged(e){
  //登录状态变化处理逻辑
}

@override
void initState() {
  //订阅登录状态改变事件
  bus.on(Event.login,onLogin);
  super.initState();
}

@override
void dispose() {
  //取消订阅
  bus.off(Event.login,onLogin);
  super.dispose();
}

```

We can find that there are some obvious shortcomings in implementing cross-component state sharing through the observer pattern:

1.  Various events must be explicitly defined, which is not easy to manage
2.  Subscribers must explicitly register the state change callback, and must manually unbind the callback when the component is destroyed to avoid memory leaks.

Is there a better way of cross-component state management in Flutter? The answer is yes, how is it done? We think about the previous introduction `InheritedWidget`, its natural feature is to be able to bind `InheritedWidget`the dependency relationship with its descendant components, and when the `InheritedWidget`data changes, it can automatically update the dependent descendant components! Using this feature, we can save the state that needs to be shared across components `InheritedWidget`and then reference it in the subcomponents. The `InheritedWidget`famous Provider package in the Flutter community is based on this idea to achieve a set of cross-component state sharing solutions. We will introduce in detail the usage and principle of Provider.

## Provider

In order to strengthen the reader's understanding, we do not directly look at the source code of the Provider package. On the contrary, I will take you `InheritedWidget`to implement a minimally functional Provider step by step according to the above-described idea of ​​implementation.

First of all, we need to save the data `InheritedWidget`that needs to be shared . Because the specific business data type is unpredictable, for versatility, we use generics and define a generic `InheritedProvider`class, which inherits from `InheritedWidget`:

```
// 一个通用的InheritedWidget，保存任需要跨组件共享的状态
class InheritedProvider<T> extends InheritedWidget {
  InheritedProvider({@required this.data, Widget child}) : super(child: child);

  //共享状态使用泛型
  final T data;

  @override
  bool updateShouldNotify(InheritedProvider<T> old) {
    //在此简单返回true，则每次更新都会调用依赖其的子孙节点的`didChangeDependencies`。
    return true;
  }
}

```

There is a place to save the data, then what we need to do next is to rebuild when the data changes `InheritedProvider`, then we now face two problems:

1.  How to notify when data changes?
2.  Who will rebuild `InheritedProvider`?

The first problem is actually very easy to solve. Of course, we can use the eventBus introduced before for event notification, but in order to be closer to Flutter development, we use the `ChangeNotifier`class provided in the Flutter SDK , which inherits from `Listenable`and implements a Flutter-style release The subscriber-subscriber model `ChangeNotifier`is roughly defined as follows:

```
class ChangeNotifier implements Listenable {
  List listeners=[];
  @override
  void addListener(VoidCallback listener) {
     //添加监听器
     listeners.add(listener);
  }
  @override
  void removeListener(VoidCallback listener) {
    //移除监听器
    listeners.remove(listener);
  }

  void notifyListeners() {
    //通知所有监听器，触发监听器回调 
    listeners.forEach((item)=>item());
  }

  ... //省略无关代码
}

```

We can call `addListener()`and `removeListener()`to add, remove listener (subscribers); by calling `notifyListeners()`can trigger all listeners callback.

Now, we put the state to be shared in a Model class, and then let it inherit from `ChangeNotifier`, so that when the shared state changes, we only need to call `notifyListeners()`to notify the subscriber, and then the subscriber will rebuild `InheritedProvider`, which is also the second The answer! Next we will implement this subscriber class:

```

class ChangeNotifierProvider<T extends ChangeNotifier> extends StatefulWidget {
  ChangeNotifierProvider({
    Key key,
    this.data,
    this.child,
  });

  final Widget child;
  final T data;

  //定义一个便捷方法，方便子树中的widget获取共享数据
  static T of<T>(BuildContext context) {
    final type = _typeOf<InheritedProvider<T>>();
    final provider =  context.dependOnInheritedWidgetOfExactType<InheritedProvider<T>>();
    return provider.data;
  }

  @override
  _ChangeNotifierProviderState<T> createState() => _ChangeNotifierProviderState<T>();
}

```

This class inherits `StatefulWidget`, and then defines a `of()`static method for subclasses to easily obtain `InheritedProvider`the shared state (model) stored in the Widget tree . Below we implement the corresponding `_ChangeNotifierProviderState`class of this class:

```
class _ChangeNotifierProviderState<T extends ChangeNotifier> extends State<ChangeNotifierProvider<T>> {
  void update() {
    //如果数据发生变化（model类调用了notifyListeners），重新构建InheritedProvider
    setState(() => {});
  }

  @override
  void didUpdateWidget(ChangeNotifierProvider<T> oldWidget) {
    //当Provider更新时，如果新旧数据不"=="，则解绑旧数据监听，同时添加新数据监听
    if (widget.data != oldWidget.data) {
      oldWidget.data.removeListener(update);
      widget.data.addListener(update);
    }
    super.didUpdateWidget(oldWidget);
  }

  @override
  void initState() {
    // 给model添加监听器
    widget.data.addListener(update);
    super.initState();
  }

  @override
  void dispose() {
    // 移除model的监听器
    widget.data.removeListener(update);
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return InheritedProvider<T>(
      data: widget.data,
      child: widget.child,
    );
  }
}

```

You can see `_ChangeNotifierProviderState`that the main function of the class is to rebuild the Widget tree when the shared state (model) changes. Note `_ChangeNotifierProviderState`that the calling `setState()`method in the class is `widget.child`always the same, so when the build is executed, `InheritedProvider`the child reference is always the same child widget, so `widget.child`it will not be renewed `build`, which is equivalent to `child`caching! Of course, if the `ChangeNotifierProvider`parent Widget is rebuilt, the passed-in `child`may change.

Now that the various tool classes we need have been completed, let's take a shopping cart example to see how to use the above classes.

### Shopping cart example

We need to implement a function that displays the total price of all items in the shopping cart:

1.  The total price is updated when new items are added to the shopping cart

Define a `Item`class to represent product information:

```
class Item {
  Item(this.price, this.count);
  double price; //商品单价
  int count; // 商品份数
  //... 省略其它属性
}

```

Define a `CartModel`class that saves product data in the shopping cart :

```
class CartModel extends ChangeNotifier {
  // 用于保存购物车中商品列表
  final List<Item> _items = [];

  // 禁止改变购物车里的商品信息
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  // 购物车中商品的总价
  double get totalPrice =>
      _items.fold(0, (value, item) => value + item.count * item.price);

  // 将 [item] 添加到购物车。这是唯一一种能从外部改变购物车的方法。
  void add(Item item) {
    _items.add(item);
    // 通知监听器（订阅者），重新构建InheritedProvider， 更新状态。
    notifyListeners();
  }
}

```

`CartModel`The model class to be shared across components. Finally, we build a sample page:

```
class ProviderRoute extends StatefulWidget {
  @override
  _ProviderRouteState createState() => _ProviderRouteState();
}

class _ProviderRouteState extends State<ProviderRoute> {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: ChangeNotifierProvider<CartModel>(
        data: CartModel(),
        child: Builder(builder: (context) {
          return Column(
            children: <Widget>[
              Builder(builder: (context){
                var cart=ChangeNotifierProvider.of<CartModel>(context);
                return Text("总价: ${cart.totalPrice}");
              }),
              Builder(builder: (context){
                print("RaisedButton build"); //在后面优化部分会用到
                return RaisedButton(
                  child: Text("添加商品"),
                  onPressed: () {
                    //给购物车中添加商品，添加后总价会更新
                    ChangeNotifierProvider.of<CartModel>(context).add(Item(20.0, 1));
                  },
                );
              }),
            ],
          );
        }),
      ),
    );
  }
}

```

The effect after running the example is shown in Figure 7-2:

![provider](https://pcdn.flutterchina.club/imgs/7-2.png)

Every time you click the "Add Product" button, the total price will increase by 20, and the desired function is realized! Some readers may wonder, does it make sense to implement such a simple function with a large circle? In fact, based on this example, we just update a state in the same routing page. `ChangeNotifierProvider`The advantage we use is not obvious, but what if we are making a shopping APP? Since shopping cart data is usually shared throughout the app, for example, it will be shared across routes. If we put it `ChangeNotifierProvider`at the root of the Widget tree of the entire application, then the entire APP can share the data of the shopping cart, and `ChangeNotifierProvider`the advantage at this time will be very obvious.

Although the above example is relatively simple, it clearly reflects the principle and process of Provider. Figure 7-3 is the schematic diagram of Provider:

![Figure 7-3](https://pcdn.flutterchina.club/imgs/7-3.png)

After the model changes, it will automatically notify `ChangeNotifierProvider`(subscribers), the `ChangeNotifierProvider`internal will be rebuilt `InheritedWidget`, and `InheritedWidget`the descendant widgets that depend on it will be updated.

We can find that using Provider will bring the following benefits:

1.  Our business code pays more attention to data. As long as the Model is updated, the UI will be updated automatically, instead of manually calling `setState()`to update the page explicitly after the state changes .
2.  The message delivery of data changes is blocked, and we don't need to manually handle the publishing and subscribing of state change events. All of this is encapsulated in the Provider. This is really great and saves us a lot of work!
3.  In large and complex applications, especially when there are many states that need to be shared globally, using Provider will greatly simplify our code logic, reduce the probability of errors, and improve development efficiency.

### optimization

What we have achieved above `ChangeNotifierProvider`is that there are two obvious shortcomings: code organization issues and performance issues, we will discuss them one by one below.

#### Code organization issues

Let's first look at the code that builds the text that displays the total price:

```
Builder(builder: (context){
  var cart=ChangeNotifierProvider.of<CartModel>(context);
  return Text("总价: ${cart.totalPrice}");
})

```

This code has two points that can be optimized:

1.  It needs to be explicitly called `ChangeNotifierProvider.of`. When the APP has a `CartModel`lot of internal dependencies , such code will be very redundant.
2.  Semantic ambiguity; Because `ChangeNotifierProvider`a subscriber, less dependent on `CartModel`the Widget subscribers nature is, in fact, is the state of the consumer, if we use `Builder`to build semantic is not very clear; if we could use a Widget with a clear semantics, such as It is called `Consumer`, so the final code semantics will be very clear, as long as `Consumer`we see it , we know that it is dependent on a cross-component or global state.

In order to optimize these two problems, we can encapsulate a `Consumer`Widget as follows:

```
// 这是一个便捷类，会获得当前context和指定数据类型的Provider
class Consumer<T> extends StatelessWidget {
  Consumer({
    Key key,
    @required this.builder,
    this.child,
  })  : assert(builder != null),
        super(key: key);

  final Widget child;

  final Widget Function(BuildContext context, T value) builder;

  @override
  Widget build(BuildContext context) {
    return builder(
      context,
      ChangeNotifierProvider.of<T>(context), //自动获取Model
    );
  }
}

```

`Consumer`The implementation is very simple. It `ChangeNotifierProvider.of`obtains the corresponding Model by specifying template parameters and then automatically calling it internally , and `Consumer`the name itself also has exact semantics (consumer). Now the above code block can be optimized as follows:

```
Consumer<CartModel>(
  builder: (context, cart)=> Text("总价: ${cart.totalPrice}");
)

```

Isn't it elegant!

#### Performance issues

The above code also has a performance problem, just where the code for "add button" is built:

```
Builder(builder: (context) {
  print("RaisedButton build"); // 构建时输出日志
  return RaisedButton(
    child: Text("添加商品"),
    onPressed: () {
      ChangeNotifierProvider.of<CartModel>(context).add(Item(20.0, 1));
    },
  );
}

```

After we click the "Add Product" button, since the total price of the shopping cart will change, the text update showing the total price is in line with expectations, but the "Add Product" button itself has not changed and should not be rebuilt. But when we run the example, every time the "Add Product" button is clicked, the console will output the "RaisedButton build" log, which means that the "Add Product" button will rebuild itself every time it is clicked! Why is this? If you understand the `InheritedWidget`update mechanism, one can see the answer: This is because the build `RaisedButton`of `Builder`the call `ChangeNotifierProvider.of`, that is dependent on the Widget above the tree `InheritedWidget`(that is `InheritedProvider`) Widget, so when you are finished adding goods, `CartModel`changes, Will be notified `ChangeNotifierProvider`, and the `ChangeNotifierProvider`subtree will be rebuilt, so it `InheritedProvider`will be updated, and the descendants of Widgets that depend on it will be rebuilt at this time.

The cause of the problem is clear, so how can we avoid this unnecessary refactoring? Since the button is rebuilt because the button has `InheritedWidget`established a dependency relationship, then we only need to break or remove this dependency relationship. So how to remove `InheritedWidget`the dependency of the button and ? Our previous section `InheritedWidget`when already talked about: Call `dependOnInheritedWidgetOfExactType()`and `getElementForInheritedWidgetOfExactType()`the difference is that the former will register dependencies, and the latter will not. So we only need to change `ChangeNotifierProvider.of`the implementation to the following:

```
 //添加一个listen参数，表示是否建立依赖关系
  static T of<T>(BuildContext context, {bool listen = true}) {
    final type = _typeOf<InheritedProvider<T>>();
    final provider = listen
        ? context.dependOnInheritedWidgetOfExactType<InheritedProvider<T>>()
        : context.getElementForInheritedWidgetOfExactType<InheritedProvider<T>>()?.widget
            as InheritedProvider<T>;
    return provider.data;
  }

```

Then we change the calling part of the code to:

```
Column(
    children: <Widget>[
      Consumer<CartModel>(
        builder: (BuildContext context, cart) =>Text("总价: ${cart.totalPrice}"),
      ),
      Builder(builder: (context) {
        print("RaisedButton build");
        return RaisedButton(
          child: Text("添加商品"),
          onPressed: () {
            // listen 设为false，不建立依赖关系
            ChangeNotifierProvider.of<CartModel>(context, listen: false)
                .add(Item(20.0, 1));
          },
        );
      })
    ],
  )

```

Run the above example again after modification, we will find that after clicking the "Add Product" button, the console will no longer output "RaisedButton build", that is, the button will not be rebuilt. The total price will still be updated. This is because the default value is true when `Consumer`called , so the dependency relationship will still be established.`ChangeNotifierProvider.of``listen`

So far we have implemented a mini Provider, which has the core functions of the Provider Package on Pub; but our mini version is not comprehensive, such as only a monitorable ChangeNotifierProvider, but not only for data sharing Provider; In addition, some boundaries of our implementation are not considered, such as how to ensure that the Model is always a singleton when the Widget tree is rebuilt. Therefore, it is recommended that readers still use Provider Package in actual combat, and the main purpose of implementing this mini Provider in this section is to help readers understand the underlying principles of Provider Package.

### Other state management packages

Now the Flutter community has many packages dedicated to state management. Here we list a few relatively high scores:

| Package names                                                                                                             | Introduction                                                                                          |
| ------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| [Provider](https://pub.flutter-io.cn/packages/provider) & [Scoped Model](https://pub.flutter-io.cn/packages/scoped_model) | These two packages are based on `InheritedWidget`similar principles                                   |
| [Redux](https://pub.flutter-io.cn/packages/flutter_redux)                                                                 | It is the Flutter implementation of the Redux package in the React ecosystem in web development       |
| [MobX](https://pub.dev/packages/flutter_mobx)                                                                             | It is the Flutter implementation of the MobX package in the React ecological chain in web development |
| [BLoC](https://pub.dev/packages/flutter_bloc)                                                                             | Flutter implementation of BLoC mode                                                                   |


The author does not recommend these packages. Readers who are interested can study them to understand their respective thoughts.

### to sum up

This section introduces some shortcomings of the event bus in cross-component sharing, leading `InheritedWidget`to the idea of ​​realizing state sharing, and then implements a simple Provider based on this idea, and in-depth exploration of `InheritedWidget`its dependencies during the implementation process Registration mechanism and update mechanism. Through the study of this section, readers should achieve two goals, first is a `InheritedWidget`thorough understanding, and second is the design idea of ​​Provider.

`InheritedWidget`It is a very important Widget in Flutter, such as internationalization, themes, etc. are realized through it, so we do not hesitate to introduce it through several sections, in the next section, we will introduce another `InheritedWidget`component based on Theme (theme).