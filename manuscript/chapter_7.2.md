# 7.2 Data Sharing (InheritedWidget)

`InheritedWidget`It is a very important functional component in Flutter. It provides a way to transfer and share data from top to bottom in the widget tree. For example `InheritedWidget`, if we share a data in the root widget of the application , then we can Get the shared data in any sub-widget! This feature is very convenient in some scenarios that need to share data in the widget tree! For example, the Flutter SDK uses InheritedWidget to share application theme ( `Theme`) and Locale (current locale) information.

> `InheritedWidget`Similar to the context function in React, compared to passing data level by level, they can enable components to transmit data across levels. `InheritedWidget`The data transfer direction in the widget tree is from top to bottom, which `Notification`is the opposite of the transfer direction of notifications (which will be introduced in the next chapter).

### didChangeDependencies

In the previous introduction `StatefulWidget`, we mentioned that the `State`object has a `didChangeDependencies`callback, which will be called by the Flutter Framework when the "dependency" changes. And this "dependency" refers to whether the child widget uses the `InheritedWidget`data in the parent widget ! If it is used, it means that the child widget has a dependency `InheritedWidget`; if it is not used, it means it has no dependency. This mechanism allows sub-components `InheritedWidget`to update themselves when the dependent changes! For example, when the theme, locale (language), etc. change, the `didChangeDependencies`methods of the dependent child widgets will be called.

Let's take a look at the previous `InheritedWidget`version of the "Counter" sample application . It should be noted that this example is mainly for demonstrating `InheritedWidget`the functional characteristics, not the recommended implementation of the counter.

First, we `InheritedWidget`save the current counter hits in `ShareDataWidget`the `data`attribute through inheritance :

``` dart 
class ShareDataWidget extends InheritedWidget {
 ShareDataWidget({
   @required this.data,
   Widget child
 }) :super(child: child);

 final int data; //需要在子树中共享的数据，保存点击次数

 //定义一个便捷方法，方便子树中的widget获取共享数据  
 static ShareDataWidget of(BuildContext context) {
   return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
 }

 //该回调决定当data发生变化时，是否通知子树中依赖data的Widget  
 @override
 bool updateShouldNotify(ShareDataWidget old) {
   //如果返回true，则子树中依赖(build函数中有调用)本widget
   //的子widget的`state.didChangeDependencies`会被调用
   return old.data != data;
 }
}

```

Then we implement a sub-component that references the data `_TestWidget`in its `build`method `ShareDataWidget`. At the same time, `didChangeDependencies()`print the log in its callback:

``` dart 
class _TestWidget extends StatefulWidget {
 @override
 __TestWidgetState createState() => new __TestWidgetState();
}

class __TestWidgetState extends State<_TestWidget> {
 @override
 Widget build(BuildContext context) {
   //使用InheritedWidget中的共享数据
   return Text(ShareDataWidget
       .of(context)
       .data
       .toString());
 }

 @override
 void didChangeDependencies() {
   super.didChangeDependencies();
   //父或祖先widget中的InheritedWidget改变(updateShouldNotify返回true)时会被调用。
   //如果build中没有依赖InheritedWidget，则此回调不会被调用。
   print("Dependencies change");
 }
}

```

Finally, we create a button that will `ShareDataWidget`increment its value every time it is clicked :

``` dart 
class InheritedWidgetTestRoute extends StatefulWidget {
 @override
 _InheritedWidgetTestRouteState createState() => new _InheritedWidgetTestRouteState();
}

class _InheritedWidgetTestRouteState extends State<InheritedWidgetTestRoute> {
 int count = 0;

 @override
 Widget build(BuildContext context) {
   return  Center(
     child: ShareDataWidget( //使用ShareDataWidget
       data: count,
       child: Column(
         mainAxisAlignment: MainAxisAlignment.center,
         children: <Widget>[
           Padding(
             padding: const EdgeInsets.only(bottom: 20.0),
             child: _TestWidget(),//子widget中依赖ShareDataWidget
           ),
           RaisedButton(
             child: Text("Increment"),
             //每点击一次，将count自增，然后重新build,ShareDataWidget的data将被更新  
             onPressed: () => setState(() => ++count),
           )
         ],
       ),
     ),
   );
 }
}

```

The interface after running is shown in Figure 7-1:

![Figure 7-1](https://pcdn.flutterchina.club/imgs/7-1.png)

Each time you click the button, the counter will increment, and the console will print a log:

``` dart 
I/flutter ( 8513): Dependencies change

```

It `didChangeDependencies()`can be called after the dependency changes . However, readers should note that **if the data of ShareDataWidget is not used in the build method of _TestWidget, it `didChangeDependencies()`will not be called because it does not rely on ShareDataWidget** . For example, if we change the `__TestWidgetState`code to the following, it `didChangeDependencies()`will not be called:

``` dart 
class __TestWidgetState extends State<_TestWidget> {
 @override
 Widget build(BuildContext context) {
   // 使用InheritedWidget中的共享数据
   //    return Text(ShareDataWidget
   //        .of(context)
   //        .data
   //        .toString());
    return Text("text");
 }

 @override
 void didChangeDependencies() {
   super.didChangeDependencies();
   // build方法中没有依赖InheritedWidget，此回调不会被调用。
   print("Dependencies change");
 }
}

```

The above code, we will `build()`approach relies `ShareDataWidget`code commented out, then return to a fixed `Text`, so that, when clicking Increment button `ShareDataWidget`of `data`Although the change, but `__TestWidgetState`not dependent on `ShareDataWidget`, so `__TestWidgetState`the `didChangeDependencies`method will not be called. In fact, this mechanism is easy to understand, because it is reasonable and performance-friendly to only update the Widget that uses the data when the data changes.

> Questions: How does the Flutter framework know whether child widgets rely on InheritedWidget?

#### What should be done in didChangeDependencies()?

Generally speaking, child widgets rarely rewrite this method, because the framework will also call the `build()`method after the dependency changes . However, if you need to perform some expensive operations after the dependency changes, such as network requests, then the best way is to perform them in this method, so that you can avoid `build()`performing these expensive operations every time .

### Learn more about InheritedWidget

Now let’s think about it. What if we only want `__TestWidgetState`to reference `ShareDataWidget`data in but don’t want `ShareDataWidget`to call `__TestWidgetState`the `didChangeDependencies()`method when it changes ? In fact, the answer is very simple, we only need to change `ShareDataWidget.of()`the implementation:

``` dart 
//定义一个便捷方法，方便子树中的widget获取共享数据
static ShareDataWidget of(BuildContext context) {
 //return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
 return context.getElementForInheritedWidgetOfExactType<ShareDataWidget>().widget;
}

```

The only change is `ShareDataWidget`the way to get the object, the `dependOnInheritedWidgetOfExactType()`method is replaced , then what is the difference between them, let's take a look at the source code of these two methods (the implementation code is in the class, and the relationship between and we will be introduced later):`context.getElementForInheritedWidgetOfExactType<ShareDataWidget>().widget``Element``Context``Element`

``` dart 
@override
InheritedElement getElementForInheritedWidgetOfExactType<T extends InheritedWidget>() {
 assert(_debugCheckStateIsActiveForAncestorLookup());
 final InheritedElement ancestor = _inheritedWidgets == null ? null : _inheritedWidgets[T];
 return ancestor;
}
@override
InheritedWidget dependOnInheritedWidgetOfExactType({ Object aspect }) {
 assert(_debugCheckStateIsActiveForAncestorLookup());
 final InheritedElement ancestor = _inheritedWidgets == null ? null : _inheritedWidgets[T];
 //多出的部分
 if (ancestor != null) {
   assert(ancestor is InheritedElement);
   return dependOnInheritedElement(ancestor, aspect: aspect) as T;
 }
 _hadUnsatisfiedDependencies = true;
 return null;
}

```

We can see that `dependOnInheritedWidgetOfExactType()`more than `getElementForInheritedWidgetOfExactType()`more than a transfer `dependOnInheritedElement`method, `dependOnInheritedElement`source code is as follows:

``` dart 
 @override
 InheritedWidget dependOnInheritedElement(InheritedElement ancestor, { Object aspect }) {
   assert(ancestor != null);
   _dependencies ??= HashSet<InheritedElement>();
   _dependencies.add(ancestor);
   ancestor.updateDependencies(this, aspect);
   return ancestor.widget;
 }

```

You can see `dependOnInheritedElement`that the dependency is mainly registered in the method! See here also clear, **call `dependOnInheritedWidgetOfExactType()`, and `getElementForInheritedWidgetOfExactType()`the difference is that the former will register dependencies, and the latter will not** , so call `dependOnInheritedWidgetOfExactType()`time, `InheritedWidget`and the children and grandchildren depend on it will complete the registration component relationships, then when `InheritedWidget`the time changes, will be updated Depend on its descendant components, that is, will adjust the `didChangeDependencies()`methods and `build()`methods of these descendant components . And when the call is `getElementForInheritedWidgetOfExactType()`the time, because there is no register dependencies, so then when `InheritedWidget`the time changes, it will not update the descendants Widget.

Note that if you change the `ShareDataWidget.of()`method implementation in the above example to call `getElementForInheritedWidgetOfExactType()`, after running the example, click the "Increment" button, you will find that `__TestWidgetState`the `didChangeDependencies()`method will not be called anymore, but it `build()`will still be called! The reason for this is actually that after clicking the "Increment" button, `_InheritedWidgetTestRouteState`the `setState()`method will be called , `__TestWidget`and the entire page will be rebuilt at this time. Since there is no cache in the example, it will also be rebuilt, so the `build()`method will also be called .

So, now there is a problem: In fact, we only want to update the dependent `ShareDataWidget`components in the subtree , and now as long as `_InheritedWidgetTestRouteState`the `setState()`method is called , all the child nodes will be rebuilt. This is unnecessary, so what can be done? Avoid it? The answer is caching! A simple way is to `StatefulWidget`cache the sub-Widget tree by encapsulating one . In the next section, we will implement a `Provider`Widget to demonstrate how to cache and how to use `InheritedWidget`Flutter to achieve global state sharing.