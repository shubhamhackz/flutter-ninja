# 10.2 Combining existing components

In Flutter, the page UI is usually composed of some low-level components. When we need to encapsulate some common components, we should first consider whether it can be achieved by combining other components. If so, we should give priority to the combination, because Assembling directly through existing components will be very simple, flexible and efficient.

### Example: custom gradient button

The buttons in the Flutter Material component library do not support gradient backgrounds by default. In order to implement gradient background buttons, we customize a `GradientButton`component that needs to support the following functions:

1.  The background supports gradient colors
2.  There is a ripple effect when the finger is pressed
3.  Can support rounded corners

Let's take a look at the final effect (Figure 10-1):

![gradient-button](https://pcdn.flutterchina.club/imgs/10-1.png)

We `DecoratedBox`can support background color gradients and rounded corners, `InkWell`and there will be ripple effects when pressed by the finger, so we can achieve it by combining `DecoratedBox`and , the code is as follows:`InkWell``GradientButton`

```
import 'package:flutter/material.dart';

class GradientButton extends StatelessWidget {
  GradientButton({
    this.colors,
    this.width,
    this.height,
    this.onPressed,
    this.borderRadius,
    @required this.child,
  });

  // 渐变色数组
  final List<Color> colors;

  // 按钮宽高
  final double width;
  final double height;

  final Widget child;
  final BorderRadius borderRadius;

  //点击回调
  final GestureTapCallback onPressed;

  @override
  Widget build(BuildContext context) {
    ThemeData theme = Theme.of(context);

    //确保colors数组不空
    List<Color> _colors = colors ??
        [theme.primaryColor, theme.primaryColorDark ?? theme.primaryColor];

    return DecoratedBox(
      decoration: BoxDecoration(
        gradient: LinearGradient(colors: _colors),
        borderRadius: borderRadius,
      ),
      child: Material(
        type: MaterialType.transparency,
        child: InkWell(
          splashColor: _colors.last,
          highlightColor: Colors.transparent,
          borderRadius: borderRadius,
          onTap: onPressed,
          child: ConstrainedBox(
            constraints: BoxConstraints.tightFor(height: height, width: width),
            child: Center(
              child: Padding(
                padding: const EdgeInsets.all(8.0),
                child: DefaultTextStyle(
                  style: TextStyle(fontWeight: FontWeight.bold),
                  child: child,
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

```

It can be seen `GradientButton`by `DecoratedBox`, `Padding`, `Center`, `InkWell`and other component combination. Of course, the above code is just an example, it is not complete as a button, for example, there is no disabled state, readers can improve according to actual needs.

#### Use GradientButton

```
import 'package:flutter/material.dart';
import '../widgets/index.dart';

class GradientButtonRoute extends StatefulWidget {
  @override
  _GradientButtonRouteState createState() => _GradientButtonRouteState();
}

class _GradientButtonRouteState extends State<GradientButtonRoute> {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Column(
        children: <Widget>[
          GradientButton(
            colors: [Colors.orange, Colors.red],
            height: 50.0,
            child: Text("Submit"),
            onPressed: onTap,
          ),
          GradientButton(
            height: 50.0,
            colors: [Colors.lightGreen, Colors.green[700]],
            child: Text("Submit"),
            onPressed: onTap,
          ),
          GradientButton(
            height: 50.0,
            colors: [Colors.lightBlue[300], Colors.blueAccent],
            child: Text("Submit"),
            onPressed: onTap,
          ),
        ],
      ),
    );
  }
  onTap() {
    print("button click");
  }
}

```

### to sum up

There is no difference between defining the components by combination and the interface we wrote before, but we must consider the standardization of the code when extracting the individual components, and use `@required`annotations for necessary parameters . For optional parameters, they need to be blank or set in specific scenarios. Default values, etc. This is because most of the time users may not understand the internal details of the component, so in order to ensure the robustness of the code, we need to be compatible or report an error prompt (using the `assert`assertion function) when the user uses the component incorrectly .