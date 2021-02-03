# 9.2 Basic structure of animation and status monitoring

## 9.2.1 Basic structure of animation

In Flutter, we can implement animation in a variety of ways. Below, we will demonstrate the difference between different implementations of animation in Flutter through the different implementations of an example of gradually enlarging a picture.

### Basic version

Below we demonstrate the most basic animation implementation:

```
class ScaleAnimationRoute extends StatefulWidget {
  @override
  _ScaleAnimationRouteState createState() => new _ScaleAnimationRouteState();
}

//需要继承TickerProvider，如果有多个AnimationController，则应该使用TickerProviderStateMixin。
class _ScaleAnimationRouteState extends State<ScaleAnimationRoute>  with SingleTickerProviderStateMixin{ 

  Animation<double> animation;
  AnimationController controller;

  initState() {
    super.initState();
    controller = new AnimationController(
        duration: const Duration(seconds: 3), vsync: this);
    //图片宽高从0变到300
    animation = new Tween(begin: 0.0, end: 300.0).animate(controller)
      ..addListener(() {
        setState(()=>{});
      });
    //启动动画(正向执行)
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return new Center(
       child: Image.asset("imgs/avatar.png",
          width: animation.value,
          height: animation.value
      ),
    );
  }

  dispose() {
    //路由销毁时需要释放动画资源
    controller.dispose();
    super.dispose();
  }
}

```

The `addListener()`function in the above code is called `setState()`, so every time the animation generates a new number, the current frame is marked as dirty, which will cause the widget `build()`method to be called again, and in the `build()`middle, the width and height of the Image are changed because of it The height and width are now used `animation.value`, so it will gradually enlarge. It is worth noting that the controller (call `dispose()`method) should be released when the animation is complete to prevent memory leaks.

Curve is not specified in the above example, so the zooming process is linear (uniform speed), below we specify a Curve to achieve an animation process similar to the spring effect, we only need to change `initState`the code to the following. :

```
  initState() {
    super.initState();
    controller = new AnimationController(
        duration: const Duration(seconds: 3), vsync: this);
    //使用弹性曲线
    animation=CurvedAnimation(parent: controller, curve: Curves.bounceIn);
    //图片宽高从0变到300
    animation = new Tween(begin: 0.0, end: 300.0).animate(animation)
      ..addListener(() {
        setState(() {
        });
      });
    //启动动画
    controller.forward();
  }

```

After the above code is executed, two of the frames are intercepted, and the effect is shown in Figure 9-1 and 9-2:

![Figure 9-1](https://pcdn.flutterchina.club/imgs/9-1.png)![Figure 9-2](https://pcdn.flutterchina.club/imgs/9-2.png)

### Simplify with AnimatedWidget

Attentive readers may have found that the step of updating the UI through `addListener()`and `setState()`in the above example is actually universal. It is more cumbersome to add such a sentence to each animation. `AnimatedWidget`The class encapsulates the `setState()`details of the call and allows us to separate the widget. The refactored code is as follows:

```
class AnimatedImage extends AnimatedWidget {
  AnimatedImage({Key key, Animation<double> animation})
      : super(key: key, listenable: animation);

  Widget build(BuildContext context) {
    final Animation<double> animation = listenable;
    return new Center(
      child: Image.asset("imgs/avatar.png",
          width: animation.value,
          height: animation.value
      ),
    );
  }
}


class ScaleAnimationRoute1 extends StatefulWidget {
  @override
  _ScaleAnimationRouteState createState() => new _ScaleAnimationRouteState();
}

class _ScaleAnimationRouteState extends State<ScaleAnimationRoute1>
    with SingleTickerProviderStateMixin {

  Animation<double> animation;
  AnimationController controller;

  initState() {
    super.initState();
    controller = new AnimationController(
        duration: const Duration(seconds: 3), vsync: this);
    //图片宽高从0变到300
    animation = new Tween(begin: 0.0, end: 300.0).animate(controller);
    //启动动画
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedImage(animation: animation,);
  }

  dispose() {
    //路由销毁时需要释放动画资源
    controller.dispose();
    super.dispose();
  }
}

```

### Refactor with AnimatedBuilder

AnimatedWidget can be used to separate the widget from the animation, and the rendering process of the animation (that is, setting the width and height) is still in the AnimatedWidget. If we add another animation that changes the transparency of the widget, then we need to implement another AnimatedWidget, which is not very elegant If we can also abstract the rendering process, it will be much better, and AnimatedBuilder is to separate the rendering logic, the code in the build method above can be changed to:

```
@override
Widget build(BuildContext context) {
  //return AnimatedImage(animation: animation,);
    return AnimatedBuilder(
      animation: animation,
      child: Image.asset("images/avatar.png"),
      builder: (BuildContext ctx, Widget child) {
        return new Center(
          child: Container(
              height: animation.value, 
              width: animation.value, 
              child: child,
          ),
        );
      },
    );
}

```

One confusing problem in the code above is that `child`it looks like it has been specified twice. However, what actually happens is that: the external reference `child`is passed to `AnimatedBuilder`the `AnimatedBuilder`then passed to the constructor anonymous, then the object as its child objects. The final result is that the `AnimatedBuilder`returned object is inserted into the widget tree.

You may say that this is not much different from the example we just started, but in fact it will bring three benefits:

1.  There is no need to explicitly add a frame listener and then call it `setState()`again. The benefits `AnimatedWidget`are the same.
    
2.  The scope of animation construction is reduced. If not `builder`, it `setState()`will be called in the context of the parent component, which will cause the `build`method of the parent component to be re-invoked; and `builder`after it has, it will only cause the `build`re-invocation of the animation widget itself , avoiding unnecessary rebuilds .
    
3.  `AnimatedBuilder`You can reuse animation by encapsulating common transition effects. Below we `GrowTransition`illustrate by encapsulating one , it can realize the zoom animation for the child widget:
    
    ```
    class GrowTransition extends StatelessWidget {
      GrowTransition({this.child, this.animation});
    
      final Widget child;
      final Animation<double> animation;
    
      Widget build(BuildContext context) {
        return new Center(
          child: new AnimatedBuilder(
              animation: animation,
              builder: (BuildContext context, Widget child) {
                return new Container(
                    height: animation.value, 
                    width: animation.value, 
                    child: child
                );
              },
              child: child
          ),
        );
      }
    }
    
    ```
    
    In this way, the original example can be changed to:
    
    ```
    ...
    Widget build(BuildContext context) {
        return GrowTransition(
        child: Image.asset("images/avatar.png"), 
        animation: animation,
        );
    }
    
    ```
    
    **Flutter encapsulates a lot of animations in this way, such as: FadeTransition, ScaleTransition, SizeTransition, etc. In many cases, these preset transition classes can be reused.**
    

## 9.2.2 Animation status monitoring

As mentioned above, we can add an animation state change listener through `Animation`the `addStatusListener()`method. In Flutter, there are four animation states, `AnimationStatus`defined in the enumeration class, let's explain one by one below:

Enumerated value

meaning

`dismissed`

Animation stops at the starting point

`forward`

Animation is being executed forward

`reverse`

Animation is being executed in reverse

`completed`

Animation stops at the end

#### Example

We change the example of zooming in the picture above to zoom in, then zoom out, then zoom in... this kind of loop animation. To achieve this effect, we only need to monitor the change of the animation state, that is, reverse the animation when the forward execution of the animation ends, and then execute the animation forward when the reverse execution of the animation ends. code show as below:

```
  initState() {
    super.initState();
    controller = new AnimationController(
        duration: const Duration(seconds: 1), vsync: this);
    //图片宽高从0变到300
    animation = new Tween(begin: 0.0, end: 300.0).animate(controller);
    animation.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        //动画执行结束时反向执行动画
        controller.reverse();
      } else if (status == AnimationStatus.dismissed) {
        //动画恢复到初始状态时执行动画（正向）
        controller.forward();
      }
    });

    //启动动画（正向）
    controller.forward();
  }

```