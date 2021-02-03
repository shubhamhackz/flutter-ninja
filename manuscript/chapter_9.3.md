# 9.3 Custom routing switching animation

We talked about in the "Routing Management" section of Chapter 2: The Material Component Library provides a `MaterialPageRoute`component that can use the routing switching animation consistent with the platform style, such as sliding left and right on iOS, and switching on Android. Swipe up and down to switch. Now, if we want to use left and right switching styles on Android, what should we do? A simple way is to use it directly `CupertinoPageRoute`, such as:

```
 Navigator.push(context, CupertinoPageRoute(  
   builder: (context)=>PageB(),
 ));

```

`CupertinoPageRoute`It is an iOS-style routing switch component provided by the Cupertino component library. It implements a left-right sliding switch. So how do we customize the route switching animation? The answer is `PageRouteBuilder`. Let's take a look at how to use `PageRouteBuilder`custom routing switch animation. For example, we want to implement routing transition with fade-in animation, the implementation code is as follows:

```
Navigator.push(
  context,
  PageRouteBuilder(
    transitionDuration: Duration(milliseconds: 500), //动画时间为500毫秒
    pageBuilder: (BuildContext context, Animation animation,
        Animation secondaryAnimation) {
      return new FadeTransition(
        //使用渐隐渐入过渡,
        opacity: animation,
        child: PageB(), //路由B
      );
    },
  ),
);

```

We can see that `pageBuilder`there is a `animation`parameter, which is provided by the Flutter router manager, which `pageBuilder`will be called back in every animation frame when the route is switched , so we can `animation`customize the transition animation through the object.

Whether it is `MaterialPageRoute`, `CupertinoPageRoute`or `PageRouteBuilder`they are inherited from PageRoute class, but `PageRouteBuilder`really just `PageRoute`a package, we can directly inherited `PageRoute`classes to implement custom routing, the above example can be achieved by:

1.  Define a routing class`FadeRoute`
    
    ```
    class FadeRoute extends PageRoute {
      FadeRoute({
        @required this.builder,
        this.transitionDuration = const Duration(milliseconds: 300),
        this.opaque = true,
        this.barrierDismissible = false,
        this.barrierColor,
        this.barrierLabel,
        this.maintainState = true,
      });
    
      final WidgetBuilder builder;
    
      @override
      final Duration transitionDuration;
    
      @override
      final bool opaque;
    
      @override
      final bool barrierDismissible;
    
      @override
      final Color barrierColor;
    
      @override
      final String barrierLabel;
    
      @override
      final bool maintainState;
    
      @override
      Widget buildPage(BuildContext context, Animation<double> animation,
          Animation<double> secondaryAnimation) => builder(context);
    
      @override
      Widget buildTransitions(BuildContext context, Animation<double> animation,
          Animation<double> secondaryAnimation, Widget child) {
         return FadeTransition( 
           opacity: animation,
           child: builder(context),
         );
      }
    }
    
    ```
    
2.  use`FadeRoute`
    
    ```
    Navigator.push(context, FadeRoute(builder: (context) {
      return PageB();
    }));
    
    ```
    

Although the above two methods can implement custom switching animations, PageRouteBuilder should be given priority in actual use, so there is no need to define a new routing class, which will be more convenient to use. But sometimes it `PageRouteBuilder`cannot meet the requirements. For example, when applying transition animation, we need to read some properties of the current route. At this time, we can only use inheritance `PageRoute`. For example, if we only want to apply when opening a new route Animation, and animation is not used when returning, then we must determine whether the current routing `isActive`property is when constructing the transition animation `true`, the code is as follows:

```
@override
Widget buildTransitions(BuildContext context, Animation<double> animation,
    Animation<double> secondaryAnimation, Widget child) {
 //当前路由被激活，是打开新路由
 if(isActive) {
   return FadeTransition(
     opacity: animation,
     child: builder(context),
   );
 }else{
   //是返回，则不应用过渡动画
   return Padding(padding: EdgeInsets.zero);
 }
}

```

For detailed information about routing parameters, readers can consult the API documentation by themselves, which is relatively simple and will not be repeated.