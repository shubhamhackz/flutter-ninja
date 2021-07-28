# 10.3 Combination example: TurnBox

As we have introduced before `RotatedBox`, it can rotate subcomponents, but it has two disadvantages: one is that it can only rotate its child nodes in multiples of 90 degrees; the other is that when the rotation angle changes, there is no animation in the rotation angle update process .

In this section, we will implement a `TurnBox`component that can not only rotate its child nodes at any angle, but also execute an animation to transition to a new state when the angle changes. At the same time, we can manually specify the animation speed.

`TurnBox`The complete code is as follows:

``` dart 
import 'package:flutter/widgets.dart';

class TurnBox extends StatefulWidget {
 const TurnBox({
   Key key,
   this.turns = .0, //旋转的“圈”数,一圈为360度，如0.25圈即90度
   this.speed = 200, //过渡动画执行的总时长
   this.child
 }) :super(key: key);

 final double turns;
 final int speed;
 final Widget child;

 @override
 _TurnBoxState createState() => new _TurnBoxState();
}

class _TurnBoxState extends State<TurnBox>
   with SingleTickerProviderStateMixin {
 AnimationController _controller;

 @override
 void initState() {
   super.initState();
   _controller = new AnimationController(
       vsync: this,
       lowerBound: -double.infinity,
       upperBound: double.infinity
   );
   _controller.value = widget.turns;
 }

 @override
 void dispose() {
   _controller.dispose();
   super.dispose();
 }

 @override
 Widget build(BuildContext context) {
   return RotationTransition(
     turns: _controller,
     child: widget.child,
   );
 }

 @override
 void didUpdateWidget(TurnBox oldWidget) {
   super.didUpdateWidget(oldWidget);
   //旋转角度发生变化时执行过渡动画  
   if (oldWidget.turns != widget.turns) {
     _controller.animateTo(
       widget.turns,
       duration: Duration(milliseconds: widget.speed??200),
       curve: Curves.easeOut,
     );
   }
 }
}

```

In the above code:

1.  We `RotationTransition`achieve the rotation effect by combining and child.
2.  In `didUpdateWidget`, we judge whether the angle to be rotated has changed, and if it has changed, perform a transition animation.

Below we test `TurnBox`the function, the test code is as follows:

``` dart 
import 'package:flutter/material.dart';
import '../widgets/index.dart';

class TurnBoxRoute extends StatefulWidget {
 @override
 _TurnBoxRouteState createState() => new _TurnBoxRouteState();
}

class _TurnBoxRouteState extends State<TurnBoxRoute> {
 double _turns = .0;

 @override
 Widget build(BuildContext context) {

   return Center(
     child: Column(
       children: <Widget>[
         TurnBox(
           turns: _turns,
           speed: 500,
           child: Icon(Icons.refresh, size: 50,),
         ),
         TurnBox(
           turns: _turns,
           speed: 1000,
           child: Icon(Icons.refresh, size: 150.0,),
         ),
         RaisedButton(
           child: Text("顺时针旋转1/5圈"),
           onPressed: () {
             setState(() {
               _turns += .2;
             });
           },
         ),
         RaisedButton(
           child: Text("逆时针旋转1/5圈"),
           onPressed: () {
             setState(() {
               _turns -= .2;
             });
           },
         )
       ],
     ),
   );
 }
}

```

The effect after the test code runs is shown in Figure 10-2:

![Figure 10-2](https://pcdn.flutterchina.club/imgs/10-2.png)

When we click the rotation button, the rotation of the two icons will rotate 1/5 circle, but the speed of rotation is different, readers can run an example to see the effect.

In fact, this example only combines `RotationTransition`one component, which is the simplest example of a combined component. In addition, if we encapsulate it `StatefulWidget`, we must pay attention to whether we need to synchronize the state when the component is updated. For example, we want to encapsulate a rich text display component `MyRichText`, which can automatically process url links, defined as follows:

``` dart 
class MyRichText extends StatefulWidget {
 MyRichText({
   Key key,
   this.text, // 文本字符串
   this.linkStyle, // url链接样式
 }) : super(key: key);

 final String text;
 final TextStyle linkStyle;

 @override
 _MyRichTextState createState() => _MyRichTextState();
}

```

Next we `_MyRichTextState`have two functions to be implemented in:

1.  Parse the text string "text" and generate a `TextSpan`cache;
2.  In `build`return the final rich text styles;

`_MyRichTextState` The implemented code is roughly as follows:

``` dart 
class _MyRichTextState extends State<MyRichText> {

 TextSpan _textSpan;

 @override
 Widget build(BuildContext context) {
   return RichText(
     text: _textSpan,
   );
 }

 TextSpan parseText(String text) {
   // 耗时操作：解析文本字符串，构建出TextSpan。
   // 省略具体实现。
 }

 @override
 void initState() {
   _textSpan = parseText(widget.text)
   super.initState();
 }
}

```

Since parsing the text string `TextSpan`is a time-consuming operation, in order not to parse it once every time we build, we `initState`cache the parsed result in, and then `build`use the parsed result directly `_textSpan`. This looks good, but the above code has a serious problem, that is `text`, when the incoming parent component changes (the component tree structure remains unchanged), then the `MyRichText`displayed content will not be updated, because it `initState`will only be updated when the State is created. Called, so when `text`there is a change, it is `parseText`not re-executed, resulting `_textSpan`in the old analytic value. To solve this problem is also very simple, we only need to add a `didUpdateWidget`callback, and then call it again `parseText`:

``` dart 
@override
void didUpdateWidget(MyRichText oldWidget) {
 if (widget.text != oldWidget.text) {
   _textSpan = parseText(widget.text);
 }
 super.didUpdateWidget(oldWidget);
}

```

Some readers may think that this point is also very simple. Yes, it is very simple. The reason why I have to repeat it here is because this point is easily overlooked in actual development. Although it is simple, it is very important. In short, when we cache some data that depends on Widget parameters in State, we must pay attention to whether we need to synchronize the state when the component is updated.