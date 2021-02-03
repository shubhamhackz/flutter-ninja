# 4.2 Linear layout (Row and Column)

The so-called linear layout refers to the arrangement of sub-components in a horizontal or vertical direction. Flutter uses `Row`and `Column`to achieve linear layout, similar to `LinearLayout`controls in Android . Both `Row`and `Column`are inherited from `Flex`, we will introduce in detail in the flexible layout section `Flex`.

### Main axis and vertical axis

For linear layout, there are main axis and vertical axis. If the layout is along the horizontal direction, then the main axis refers to the horizontal direction, and the vertical axis refers to the vertical direction; if the layout is along the vertical direction, then the main axis refers to the vertical direction, and the vertical axis is horizontal direction. In the linear layout, there are two enumeration classes that define the alignment `MainAxisAlignment`and `CrossAxisAlignment`represent the main axis alignment and the vertical axis alignment respectively.

### Row

Row can arrange its child widgets in the horizontal direction. It is defined as follows:

```
Row({
  ...  
  TextDirection textDirection,    
  MainAxisSize mainAxisSize = MainAxisSize.max,    
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  VerticalDirection verticalDirection = VerticalDirection.down,  
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  List<Widget> children = const <Widget>[],
})

```

-   `textDirection`: Represents the layout order of the horizontal sub-components (from left to right or from right to left), the default is the text direction of the current Locale environment of the system (such as Chinese and English are from left to right, while Arabic is from right to right) left).
-   `mainAxisSize`: Indicates the `Row`space occupied by the spindle (horizontal) direction, by default `MainAxisSize.max`, it represents as much space occupied by the horizontal direction, then no matter how much actual child widgets horizontal space occupied by `Row`the width of the maximum width is always equal to the horizontal direction; and `MainAxisSize.min`represent the best It may occupy less horizontal space. When the sub-component does not occupy the horizontal remaining space, `Row`the actual width is equal to the horizontal space occupied by all the sub-components;
-   `mainAxisAlignment`: Indicates the `Row`alignment of the sub-component in the horizontal space occupied. If the `mainAxisSize`value is the value `MainAxisSize.min`, this attribute is meaningless, because the width of `Row`the sub-component is equal to the width of. Only when the `mainAxisSize`value of `MainAxisSize.max`the time, this property is significant, `MainAxisAlignment.start`showing along `textDirection`the initial alignment direction, such as the `textDirection`value of `TextDirection.ltr`the time, it `MainAxisAlignment.start`represents a left alignment, `textDirection`the value of `TextDirection.rtl`the time indicates the right alignment. And `MainAxisAlignment.end`and `MainAxisAlignment.start`opposite; `MainAxisAlignment.center`it represents centered. Readers can understand it this way: `textDirection`Yes `mainAxisAlignment`, the frame of reference.
-   `verticalDirection`: Indicates `Row`the alignment direction of the vertical axis (vertical), the default is `VerticalDirection.down`, which means from top to bottom.
-   `crossAxisAlignment`: Represents the sub-assembly is aligned in the longitudinal direction, `Row`a height equal to the subassembly height of the tallest child, and its value `MainAxisAlignment`as (comprising `start`, `end`, `center`three values), except that the `crossAxisAlignment`reference line is `verticalDirection`, i.e., `verticalDirection`the value of `VerticalDirection.down`Hour `crossAxisAlignment.start`means top alignment, when `verticalDirection`value means bottom alignment; and is exactly the opposite;`VerticalDirection.up``crossAxisAlignment.start``crossAxisAlignment.end``crossAxisAlignment.start`
-   `children` : An array of sub-components.

### Example

Please read the following code, first imagine the result of the operation:

```
Column(
  //测试Row对齐方式，排除Column默认居中对齐的干扰
  crossAxisAlignment: CrossAxisAlignment.start,
  children: <Widget>[
    Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(" hello world "),
        Text(" I am Jack "),
      ],
    ),
    Row(
      mainAxisSize: MainAxisSize.min,
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(" hello world "),
        Text(" I am Jack "),
      ],
    ),
    Row(
      mainAxisAlignment: MainAxisAlignment.end,
      textDirection: TextDirection.rtl,
      children: <Widget>[
        Text(" hello world "),
        Text(" I am Jack "),
      ],
    ),
    Row(
      crossAxisAlignment: CrossAxisAlignment.start,  
      verticalDirection: VerticalDirection.up,
      children: <Widget>[
        Text(" hello world ", style: TextStyle(fontSize: 30.0),),
        Text(" I am Jack "),
      ],
    ),
  ],
);

```

The actual running result is shown in Figure 4-1:

![Pic 4-1](https://pcdn.flutterchina.club/imgs/4-1.png)

Explanation: The first one `Row`is very simple, the default is center alignment; the second one `Row`, because the `mainAxisSize`value of `MainAxisSize.min`, `Row`the width is equal to the sum of the two `Text`widths, so the alignment is meaningless, so it will be displayed from left to right; the third `Row`setting `textDirection`value is `TextDirection.rtl`, so the order of the sub-assembly from right to left, but at this time `MainAxisAlignment.end`represents a left-aligned, so that the final result looks like the display in the third row of FIG.; row fourth test is longitudinal alignment, since the two sub The text font is different, so its height is also different. We specified the `verticalDirection`value `VerticalDirection.up`, which is arranged from low to top, and the `crossAxisAlignment`value at this time `CrossAxisAlignment.start`indicates bottom alignment.

### Column

`Column`Its sub-components can be arranged in the vertical direction. The parameters are the `Row`same. The difference is that the layout direction is vertical, and the main axis is the opposite. Readers can `Row`understand it by analogy . Let's look at an example:

```
import 'package:flutter/material.dart';

class CenterColumnRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.center,
      children: <Widget>[
        Text("hi"),
        Text("world"),
      ],
    );
  }
}

```

The running effect is shown in Figure 4-2:

![Figure 4-2 Example](https://pcdn.flutterchina.club/imgs/4-2.png)

Explanation:

-   Since we did not specify `Column`the `mainAxisSize`, so the default value `MainAxisSize.max`, it `Column`will take up as much space in the vertical direction, in this case, the screen height.
-   Because we specify the `crossAxisAlignment`property `CrossAxisAlignment.center`, then the child in `Column`the longitudinal direction (the horizontal direction) will be centered. Note that the alignment in the horizontal direction is bounded, the total width is `Column`the actual width of the occupied space, and the actual width depends on the Widget with the largest width among the children. In this example, `Column`there are two child widgets, and the `Text`width of the display "world" is the largest, so `Column`the actual width is `Text("world")`the width of, so it `Text("hi")`will be displayed in `Text("world")`the middle part after centering .

**In fact, the `Row`sum `Column`will only take up as much space as possible in the main axis direction, and the length of the vertical axis depends on the length of their largest child element** . If we want the two text controls in this example to be aligned in the middle of the entire phone screen, we have two methods:

-   The `Column`width is specified as the screen width; this is very simple, we can `ConstrainedBox`or `SizedBox`(we will introduce two devoted Widget in later chapters) to forcibly change the width restrictions, such as:
    
    ```
    ConstrainedBox(
      constraints: BoxConstraints(minWidth: double.infinity), 
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.center,
        children: <Widget>[
          Text("hi"),
          Text("world"),
        ],
      ),
    );
    
    ```
    
    Set `minWidth`to `double.infinity`make the width take up as much space as possible.
    
-   Use `Center`Widget; we will introduce it in later chapters.
    

### Special cases

If `Row`nested inside `Row`, or `Column`inside another nested `Column`, only the outermost `Row`or `Column`take up as much space inside `Row`or `Column`the space occupied by the actual size, in order to below `Column`as an example:

```
Container(
  color: Colors.green,
  child: Padding(
    padding: const EdgeInsets.all(16.0),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      mainAxisSize: MainAxisSize.max, //有效，外层Colum高度为整个屏幕
      children: <Widget>[
        Container(
          color: Colors.red,
          child: Column(
            mainAxisSize: MainAxisSize.max,//无效，内层Colum高度为实际高度  
            children: <Widget>[
              Text("hello world "),
              Text("I am Jack "),
            ],
          ),
        )
      ],
    ),
  ),
);

```

The running effect is shown in Figure 4-3:

![Figure 4-3](https://pcdn.flutterchina.club/imgs/4-3.png)

If you want the inside to `Column`fill the outside `Column`, you can use `Expanded`components:

```
Expanded( 
  child: Container(
    color: Colors.red,
    child: Column(
      mainAxisAlignment: MainAxisAlignment.center, //垂直方向居中对齐
      children: <Widget>[
        Text("hello world "),
        Text("I am Jack "),
      ],
    ),
  ),
)

```

The running effect is shown in Figure 4-4:

![Figure 4-4](https://pcdn.flutterchina.club/imgs/4-4.png)

We will introduce Expanded in detail when introducing flexible layout.