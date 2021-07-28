# 10.4 Self-painted components (CustomPaint and Canvas)

For some complex or irregular UI, we may not be able to achieve it by combining other components. For example, we need a regular hexagon, a gradual circular progress bar, a chessboard, etc. Of course, sometimes we can use pictures to achieve this, but in some scenes that require dynamic interaction, static pictures cannot be achieved. For example, to implement a handwriting input panel, we need to draw the UI appearance ourselves.

Almost all UI systems provide a self-drawing UI interface. This interface usually provides a 2D canvas `Canvas`, `Canvas`which encapsulates some basic drawing APIs. Developers can `Canvas`draw various custom graphics. In Flutter, a `CustomPaint`component is provided, which can be combined with a brush `CustomPainter`to realize custom graphics drawing.

### CustomPaint

We look at the `CustomPaint`constructor:

``` dart 
CustomPaint({
 Key key,
 this.painter, 
 this.foregroundPainter,
 this.size = Size.zero, 
 this.isComplex = false, 
 this.willChange = false, 
 Widget child, //子节点，可以为空
})

```

-   `painter`: The background brush will be displayed behind the child node;
-   `foregroundPainter`: Foreground brush, will be displayed in front of the child node
-   `size`: When child is null, it represents the default drawing area size. If there is child, ignore this parameter, and the canvas size is child size. If you have a child but want to specify the canvas as a specific size, you can use SizeBox to wrap CustomPaint.
-   `isComplex`: Is it complicated to draw? If so, Flutter will apply some caching strategies to reduce the overhead of repeated rendering.
-   `willChange`: `isComplex`Used in conjunction with it, when caching is enabled, this attribute represents whether the drawing will change in the next frame.

As you can see, we need to provide foreground or background brushes when drawing, and both can also be provided at the same time. Our brushes need to inherit `CustomPainter`classes, and we implement the real drawing logic in the brush class.

#### note

If `CustomPaint`there are child nodes, in order to avoid unnecessary redrawing of the child nodes and improve performance, the child nodes are usually wrapped in `RepaintBoundary`components, so that a new drawing layer (Layer) will be created when drawing, and its child components It will be drawn on the new Layer, and the parent component will be drawn on the original Layer. That is to say `RepaintBoundary`, the drawing of the child component will be independent of the drawing of the parent component, and `RepaintBoundary`will isolate `CustomPaint`the drawing boundary of its child node and itself. Examples are as follows:

``` dart 
CustomPaint(
 size: Size(300, 300), //指定画布大小
 painter: MyPainter(),
 child: RepaintBoundary(child:...)), 
)

```

### CustomPainter

`CustomPainter`The mention defines a virtual function `paint`:

``` dart 
void paint(Canvas canvas, Size size);

```

`paint`There are two parameters:

-   `Canvas`: A canvas, including various drawing methods, we list the commonly used methods:
   
   |API name| Function| | ---------- | ------ | | drawLine | Draw line| | drawPoint | Draw a point| | drawPath | Draw path| | drawImage | Draw image | | drawRect | draw rectangle | | drawCircle | draw circle | | drawOval | draw ellipse | | drawArc | draw arc |
   
-   `Size`: The size of the current drawing area.
   

### Paint

Now that the canvas is there, we still lack a brush at the end. Flutter provides a `Paint`class to implement the brush. In `Paint`, we can configure various attributes of the brush such as thickness, color, style, etc. Such as:

``` dart 
var paint = Paint() //创建一个画笔并配置其属性
 ..isAntiAlias = true //是否抗锯齿
 ..style = PaintingStyle.fill //画笔样式：填充
 ..color=Color(0x77cdb175);//画笔颜色

```

For more configuration attributes, readers can refer to the Paint class definition.

### Example: Gomoku/Play

Below we demonstrate the process of self-drawing UI through the drawing of the board and pieces in a backgammon game. First, let's take a look at our target effect, as shown in Figure 10-3:

![Figure 10-3](https://pcdn.flutterchina.club/imgs/10-3.png)

Code:

``` dart 
import 'package:flutter/material.dart';
import 'dart:math';

class CustomPaintRoute extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   return Center(
     child: CustomPaint(
       size: Size(300, 300), //指定画布大小
       painter: MyPainter(),
     ),
   );
 }
}

class MyPainter extends CustomPainter {
 @override
 void paint(Canvas canvas, Size size) {
   double eWidth = size.width / 15;
   double eHeight = size.height / 15;

   //画棋盘背景
   var paint = Paint()
     ..isAntiAlias = true
     ..style = PaintingStyle.fill //填充
     ..color = Color(0x77cdb175); //背景为纸黄色
   canvas.drawRect(Offset.zero & size, paint);

   //画棋盘网格
   paint
     ..style = PaintingStyle.stroke //线
     ..color = Colors.black87
     ..strokeWidth = 1.0;

   for (int i = 0; i <= 15; ++i) {
     double dy = eHeight * i;
     canvas.drawLine(Offset(0, dy), Offset(size.width, dy), paint);
   }

   for (int i = 0; i <= 15; ++i) {
     double dx = eWidth * i;
     canvas.drawLine(Offset(dx, 0), Offset(dx, size.height), paint);
   }

   //画一个黑子
   paint
     ..style = PaintingStyle.fill
     ..color = Colors.black;
   canvas.drawCircle(
     Offset(size.width / 2 - eWidth / 2, size.height / 2 - eHeight / 2),
     min(eWidth / 2, eHeight / 2) - 2,
     paint,
   );

   //画一个白子
   paint.color = Colors.white;
   canvas.drawCircle(
     Offset(size.width / 2 + eWidth / 2, size.height / 2 - eHeight / 2),
     min(eWidth / 2, eHeight / 2) - 2,
     paint,
   );
 }

 //在实际场景中正确利用此回调可以避免重绘开销，本示例我们简单的返回true
 @override
 bool shouldRepaint(CustomPainter oldDelegate) => true;
}

```

### performance

Drawing is a relatively expensive operation, so we should consider performance overhead when implementing self-drawing controls. Here are two suggestions for performance optimization:

-   Make good use of the `shouldRepaint`return value as much as possible ; when the UI tree is rebuilt, the control will call this method before drawing to determine whether it is necessary to redraw; if the UI we draw does not depend on the external state, then we should always return `false`, because the external The state change will not affect our UI appearance when rebuilding; if the drawing depends on the external state, then we should `shouldRepaint`judge whether the dependent state has changed, if it has changed, we should return `true`to redraw, otherwise we should return `false`without redrawing painted.
   
-   Draw as many layers as possible; in the example of Gomoku above, we put the drawing of the chessboard and the chess pieces together, so there will be a problem: because the chessboard is always the same, the user changes only the chess pieces every time they place a piece , But if it is implemented according to the above code, the board must be redrawn every time a piece is drawn, which is unnecessary. The optimization method is to separate the chessboard as a component, set its `shouldRepaint`callback value `false`, and then use the chessboard component as the background. Then put the drawing of the chess pieces in another component so that only the chess pieces need to be drawn each time a piece is placed.
   

### to sum up

The self-drawing control is very powerful. In theory, it can realize any 2D graphics appearance. In fact, all the components provided by Flutter are finally drawn by calling Canvas, but the drawing logic is encapsulated. Readers who are interested can view the appearance styles. The source code of the component, find its corresponding `RenderObject`object, such as the `Text`corresponding `RenderParagraph`object will eventually `Canvas`realize the text drawing logic. In the next section, we will use an example of a self-drawn circular background gradient progress bar to help readers deepen their impression.