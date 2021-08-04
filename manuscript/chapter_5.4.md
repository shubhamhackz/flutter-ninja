# 5.4 Transform

`Transform`You can apply some matrix transformations to its subcomponents to achieve some special effects when drawing. `Matrix4`It is a 4D matrix, through which we can achieve various matrix operations, the following is an example:

``` dart 
Container(
 color: Colors.black,
 child: new Transform(
   alignment: Alignment.topRight, //相对于坐标系原点的对齐方式
   transform: new Matrix4.skewY(0.3), //沿Y轴倾斜0.3弧度
   child: new Container(
     padding: const EdgeInsets.all(8.0),
     color: Colors.deepOrange,
     child: const Text('Apartment for rent!'),
   ),
 ),
);

```

The running effect is shown in Figure 5-10:

![Figure 5-10](../resources/imgs/5-10.png)

> The related content of matrix transformation belongs to the category of linear algebra. This book will not discuss it. Readers who are interested can understand it by themselves. In this book, we focus on some common transformation effects in Flutter. In addition, since the matrix changes occur at the time of drawing, without the need to re-layout and build, the performance is very good.

### Pan

`Transform.translate`Receiving a `offset`parameter may be plotted along the time `x`, `y`the shaft sub-assembly translated specified distance.

``` dart 
DecoratedBox(
 decoration:BoxDecoration(color: Colors.red),
 //默认原点为左上角，左移20像素，向上平移5像素  
 child: Transform.translate(
   offset: Offset(-20.0, -5.0),
   child: Text("Hello world"),
 ),
)

```

The effect is shown in Figure 5-11:

![Figure 5-11](../resources/imgs/5-11.png)

### Spin

`Transform.rotate`You can rotate and transform subcomponents, such as:

``` dart 
DecoratedBox(
 decoration:BoxDecoration(color: Colors.red),
 child: Transform.rotate(
   //旋转90度
   angle:math.pi/2 ,
   child: Text("Hello world"),
 ),
)；

```

> Note: To use, you `math.pi`need to carry out the following guide package first.
> 
``` dart 
> import 'dart:math' as math;
> 
```

The effect is shown in Figure 5-12:

![Figure 5-12](../resources/imgs/5-12.png)

### Zoom

`Transform.scale`You can reduce or enlarge the sub-components, such as:

``` dart 
DecoratedBox(
 decoration:BoxDecoration(color: Colors.red),
 child: Transform.scale(
     scale: 1.5, //放大到1.5倍
     child: Text("Hello world")
 )
);

```

The effect is shown in Figure 5-13:

![Figure 5-13](../resources/imgs/5-13.png)

### note

-   `Transform`The transformation of is applied in the drawing phase, not in the layout phase, so no matter what changes are applied to the sub-components, the size of the occupied space and the position on the screen are fixed, because these are It is determined at the layout stage. Let us explain in detail below:
   
``` dart 
    Row(
     mainAxisAlignment: MainAxisAlignment.center,
     children: <Widget>[
       DecoratedBox(
         decoration:BoxDecoration(color: Colors.red),
         child: Transform.scale(scale: 1.5,
             child: Text("Hello world")
         )
       ),
       Text("你好", style: TextStyle(color: Colors.green, fontSize: 18.0),)
     ],
   )
   
```
   
   The running effect is shown in Figure 5-14:
   
   ![Figure 5-14](../resources/imgs/5-14.png)
   
   Since the first one `Text`applies transformation (enlargement), it will be enlarged when drawing, but the space it occupies is still the red part, so the second one `Text`will be next to the red part, and eventually the text will overlap.
   
-   Since the matrix change only affects the drawing stage, in some scenarios, when the UI needs to be changed, you can directly use the matrix change to achieve visual UI changes without retriggering the build process, which will save layout Overhead, so performance will be better. Like the `Flow`components introduced before , it uses matrix transformation to update the UI. In addition, Flutter's animation components are also used extensively `Transform`to improve performance.
   

> Question: `Transform`Is the final effect of using its sub-components translated and then rotated and rotated and then translated? why?

### RotatedBox

`RotatedBox`And `Transform.rotate`functionally similar, they can be subassemblies rotational transform, but one difference: `RotatedBox`the transformation is in the layout stage, will affect the size and position of the subassembly. Let `Transform.rotate`'s change the example introduced above :

``` dart 
Row(
 mainAxisAlignment: MainAxisAlignment.center,
 children: <Widget>[
   DecoratedBox(
     decoration: BoxDecoration(color: Colors.red),
     //将Transform.rotate换成RotatedBox  
     child: RotatedBox(
       quarterTurns: 1, //旋转90度(1/4圈)
       child: Text("Hello world"),
     ),
   ),
   Text("你好", style: TextStyle(color: Colors.green, fontSize: 18.0),)
 ],
),

```

The effect is shown in Figure 5-15:

![Figure 5-15](../resources/imgs/5-15.png)

Since it `RotatedBox`is in the layout stage, the sub-component will be rotated 90 degrees (not just the content drawn), which `decoration`will affect the actual space occupied by the sub-component, so the final effect is the above picture. The reader can `Transform.rotate`compare and understand the previous example .