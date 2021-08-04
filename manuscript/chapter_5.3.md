# 5.3 DecoratedBox

`DecoratedBox`You can draw some decorations, such as background, border, gradient, etc., before (or after) the drawing of its subcomponents. `DecoratedBox`It is defined as follows:

``` dart 
const DecoratedBox({
 Decoration decoration,
 DecorationPosition position = DecorationPosition.background,
 Widget child
})

```

-   `decoration`: Represents the decoration to be drawn, its type is `Decoration`. `Decoration`It is an abstract class that defines an interface `createBoxPainter()`. The main responsibility of the subclass is to create a brush by implementing it, which is used to draw decoration.
-   `position`: This attribute determines where to draw `Decoration`, `DecorationPosition`the enumeration type it receives , the enumeration class has two values:
   -   `background`: Draw after the sub-component, that is, background decoration.
   -   `foreground`: Draw on the sub-component, that is, the foreground.

#### BoxDecoration

We usually use the `BoxDecoration`class directly , which is a subclass of Decoration, which implements the drawing of commonly used decorative elements.

``` dart 
BoxDecoration({
 Color color, //颜色
 DecorationImage image,//图片
 BoxBorder border, //边框
 BorderRadiusGeometry borderRadius, //圆角
 List<BoxShadow> boxShadow, //阴影,可以指定多个
 Gradient gradient, //渐变
 BlendMode backgroundBlendMode, //背景混合模式
 BoxShape shape = BoxShape.rectangle, //形状
})

```

Each attribute name is self-explanatory, readers can check the API documentation for details. Below we implement a button with a shaded background color gradient:

``` dart 
DecoratedBox(
   decoration: BoxDecoration(
     gradient: LinearGradient(colors:[Colors.red,Colors.orange[700]]), //背景渐变
     borderRadius: BorderRadius.circular(3.0), //3像素圆角
     boxShadow: [ //阴影
       BoxShadow(
           color:Colors.black54,
           offset: Offset(2.0,2.0),
           blurRadius: 4.0
       )
     ]
   ),
 child: Padding(padding: EdgeInsets.symmetric(horizontal: 80.0, vertical: 18.0),
   child: Text("Login", style: TextStyle(color: Colors.white),),
 )
)

```

The effect after running is shown in Figure 5-9:

![Figure 5-9](../resources/imgs/5-9.png)

Well, `BoxDecoration`we have realized the appearance of a gradient button, but this example is not a standard button yet, because it can't respond to click events yet, we will implement a complete function in the chapter "Custom Components" later `GradientButton`. In addition, the `LinearGradient`class is used in the above example , which is used to define the linear gradient class. Flutter also provides other gradient configuration classes, such as `RadialGradient`, `SweepGradient`and readers can check the API documentation if necessary.