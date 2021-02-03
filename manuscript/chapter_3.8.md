# 3.8 Progress indicator

There are two types of progress indicators provided in the Material component library: `LinearProgressIndicator`and `CircularProgressIndicator`, they can both be used for both accurate and fuzzy progress indicators. Accurate progress is usually used in situations where task progress can be calculated and estimated, such as file downloads; while fuzzy progress is situations where user task progress cannot be accurately obtained, such as pull refresh, data submission, etc.

### LinearProgressIndicator

`LinearProgressIndicator`It is a linear, bar-shaped progress bar, defined as follows:

```
LinearProgressIndicator({
  double value,
  Color backgroundColor,
  Animation<Color> valueColor,
  ...
})

```

-   `value`: `value`Indicates the current progress in the range [0,1]; if `value`is `null`the indicator performs a loop animation (Fuzzy progress) when; if `value`not `null`progress bar, the indicator for a specific schedule.
-   `backgroundColor`: The background color of the indicator.
-   `valueColor`: The color of the progress bar of the indicator; it is worth noting that the value type is , which allows us to specify animation for the color of the progress bar. If we don't need to animate the color of the progress bar, in other words, we want to apply a fixed color to the progress bar, at this time we can specify it through .`Animation<Color>``AlwaysStoppedAnimation`

### Example

```
// 模糊进度条(会执行一个动画)
LinearProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
),
//进度条显示50%
LinearProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
  value: .5, 
)

```

The running effect is shown in Figure 3-30:

![Figure 3-30](https://pcdn.flutterchina.club/imgs/3-30.png)

The first progress bar is performing a loop animation: the blue bar is always moving, while the second progress bar is static and stops at 50%.

### CircularProgressIndicator

`CircularProgressIndicator`It is a circular progress bar, defined as follows:

```
 CircularProgressIndicator({
  double value,
  Color backgroundColor,
  Animation<Color> valueColor,
  this.strokeWidth = 4.0,
  ...   
})

```

The first three parameters are the `LinearProgressIndicator`same and will not be repeated. `strokeWidth`Indicates the thickness of the circular progress bar. Examples are as follows:

```
// 模糊进度条(会执行一个旋转动画)
CircularProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
),
//进度条显示50%，会显示一个半圆
CircularProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
  value: .5,
),

```

The running effect is shown in Figure 3-31:

![Figure 3-31](https://pcdn.flutterchina.club/imgs/3-31.png)

The first progress bar will perform a rotating animation, while the second progress bar is static, it stops at 50%.

### Custom size

We can find that the `LinearProgressIndicator`sum `CircularProgressIndicator`does not provide parameters for setting the size of the circular progress bar; what if we want `LinearProgressIndicator`the line to be thinner or `CircularProgressIndicator`the circle to be larger?

In fact, the `LinearProgressIndicator`sum `CircularProgressIndicator`is based on the size of the parent container as the drawing boundary. Knowing this, we can, as restricted by the size of Widget `ConstrainedBox`, `SizedBox`(we will introduce in the back of the container class components chapter) to specify the size, such as:

```
// 线性进度条高度指定为3
SizedBox(
  height: 3,
  child: LinearProgressIndicator(
    backgroundColor: Colors.grey[200],
    valueColor: AlwaysStoppedAnimation(Colors.blue),
    value: .5,
  ),
),
// 圆形进度条直径指定为100
SizedBox(
  height: 100,
  width: 100,
  child: CircularProgressIndicator(
    backgroundColor: Colors.grey[200],
    valueColor: AlwaysStoppedAnimation(Colors.blue),
    value: .7,
  ),
),

```

The running effect is shown in Figure 3-32:

![Figure 3-32](https://pcdn.flutterchina.club/imgs/3-32.png)

Note that if `CircularProgressIndicator`the width and height of the display space are different, it will be displayed as an ellipse. Such as:

```
// 宽高不等
SizedBox(
  height: 100,
  width: 130,
  child: CircularProgressIndicator(
    backgroundColor: Colors.grey[200],
    valueColor: AlwaysStoppedAnimation(Colors.blue),
    value: .7,
  ),
),

```

The running effect is shown in Figure 3-33:

![progress_oval](https://pcdn.flutterchina.club/imgs/progress_oval.png)

### Progress color animation

As mentioned earlier, you can `valueColor`animate the color of the progress bar. We will introduce the animation in detail in a special chapter later. Here is an example. The reader will look back after understanding the Flutter animation chapter.

We implement an animation where the progress bar changes from gray to blue in 3 seconds:

```
import 'package:flutter/material.dart';

class ProgressRoute extends StatefulWidget {
  @override
  _ProgressRouteState createState() => _ProgressRouteState();
}

class _ProgressRouteState extends State<ProgressRoute>
    with SingleTickerProviderStateMixin {
  AnimationController _animationController;

  @override
  void initState() {
    //动画执行时间3秒  
    _animationController =
        new AnimationController(vsync: this, duration: Duration(seconds: 3));
    _animationController.forward();
    _animationController.addListener(() => setState(() => {}));
    super.initState();
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      child: Column(
        children: <Widget>[
            Padding(
            padding: EdgeInsets.all(16),
            child: LinearProgressIndicator(
              backgroundColor: Colors.grey[200],
              valueColor: ColorTween(begin: Colors.grey, end: Colors.blue)
                .animate(_animationController), // 从灰色变成蓝色
              value: _animationController.value,
            ),
          );
        ],
      ),
    );
  }
}

```

### Custom progress indicator style

Customize the style of the progress indicator, you can `CustomPainter`customize the drawing logic through Widget, `LinearProgressIndicator`and `CircularProgressIndicator`it is actually and precisely through `CustomPainter`to achieve the appearance of drawing. Regarding `CustomPainter`, we will introduce in detail in the chapter "Custom Widget" later.

> [The flutter_spinkit](https://pub.flutter-io.cn/packages/flutter_spinkit) package provides a variety of styles of fuzzy progress indicators, readers can refer to them if they are interested.