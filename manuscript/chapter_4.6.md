# 4.6 Alignment and relative positioning (Align)

In the previous section, we talked about the `Stack`sum `Positioned`, we can specify the exact offset of one or more child elements relative to each side of the parent element, and can overlap. But if we just want to simply adjust the position of **a** child element in the parent element, using `Align`components will be easier.

## 4.6.1 Align

`Align` The component can adjust the position of the sub-component, and can determine its own width and height according to the width and height of the sub-component, defined as follows:

```
Align({
  Key key,
  this.alignment = Alignment.center,
  this.widthFactor,
  this.heightFactor,
  Widget child,
})

```

-   `alignment`: A `AlignmentGeometry`type of value is required to indicate the starting position of the child component in the parent component. `AlignmentGeometry`Is an abstract class, it has two commonly used subclasses: `Alignment`and `FractionalOffset`, we will introduce in detail in the following example.
-   `widthFactor`The sum `heightFactor`is `Align`the attribute used to determine the width and height of the component; they are two scaling factors, which will be multiplied by the width and height of the child element respectively, and the final result is `Align`the width and height of the component. If the value is `null`, the width and height of the component will take up as much space as possible.

### Example

Let's first look at a simple example:

```
Container(
  height: 120.0,
  width: 120.0,
  color: Colors.blue[50],
  child: Align(
    alignment: Alignment.topRight,
    child: FlutterLogo(
      size: 60,
    ),
  ),
)

```

The running effect is shown in Figure 4-11:

![Figure 4-11](https://pcdn.flutterchina.club/imgs/4-11.png)

`FlutterLogo`It is a component provided by the Flutter SDK, and the content is the trademark of Flutter. In the above example, we explicitly specified the `Container`width and height to be 120. If we do not explicitly specify the width and height, the same effect can be achieved by specifying `widthFactor`and `heightFactor`as 2 at the same time:

```
Align(
  widthFactor: 2,
  heightFactor: 2,
  alignment: Alignment.topRight,
  child: FlutterLogo(
    size: 60,
  ),
),

```

Because `FlutterLogo`the width and height are 60, `Align`the final width and height are both `2*60=120`.

In addition, we `Alignment.topRight`will be `FlutterLogo`positioned in `Container`the upper right corner. That `Alignment.topRight`what is it? Through the source code we can see its definition as follows:

```
//右上角
static const Alignment topRight = Alignment(1.0, -1.0);

```

You can see that it is just `Alignment`an example, let's introduce it below `Alignment`.

### Alignment

`Alignment`Inherited from `AlignmentGeometry`, represents a point in the rectangular, he has two attributes `x`, and `y`each is offset in the horizontal and vertical directions, `Alignment`is defined as follows:

```
Alignment(this.x, this.y)

```

`Alignment`Widget will use **the center point of the rectangle as the origin of coordinates** , ie `Alignment(0.0, 0.0)`. `x`, `y`Values from -1 to 1 to represent the distance from the left and top of the rectangle on the right side in the end, and thus two horizontal (or vertical) unit of width of the rectangle is equal to (or higher), such as `Alignment(-1.0, -1.0)`rectangles representing the left vertex, And `Alignment(1.0, 1.0)`represents the bottom end point on the right side, and `Alignment(1.0, -1.0)`is the vertex on the right side, ie `Alignment.topRight`. For ease of use, the origin of the rectangle, the four vertices, and the end points of the four sides `Alignment`have been defined as static constants in the class.

`Alignment`The **coordinates** can be **converted** to the specific offset coordinates of the sub-element through its **coordinate conversion formula** :

```
(Alignment.x*childWidth/2+childWidth/2, Alignment.y*childHeight/2+childHeight/2)

```

Among them `childWidth`is the width `childHeight`of the child element and the height of the child element.

Now let's look at the above example again, we will `Alignment(1.0, -1.0)`bring in the above formula, `FlutterLogo`the actual offset coordinates available are (60, 0). Let's look at another example:

```
 Align(
  widthFactor: 2,
  heightFactor: 2,
  alignment: Alignment(2,0.0),
  child: FlutterLogo(
    size: 60,
  ),
)

```

We can first imagine the operating effect: `Alignment(2,0.0)`Bringing into the above coordinate conversion formula, `FlutterLogo`the actual offset coordinates that can be obtained are (90, 30). The actual operation is shown in Figure 4-12:

![Figure 4-12](https://pcdn.flutterchina.club/imgs/4-12.png)

### FractionalOffset

`FractionalOffset`Inherited from `Alignment`it and the `Alignment`only difference is the origin of the coordinate different! `FractionalOffset`The origin of the coordinates is the left vertex of the rectangle, which is consistent with the layout system, so it is easier to understand. `FractionalOffset`The coordinate conversion formula is:

```
实际偏移 = (FractionalOffse.x * childWidth, FractionalOffse.y * childHeight)

```

Let's look at an example:

```
Container(
  height: 120.0,
  width: 120.0,
  color: Colors.blue[50],
  child: Align(
    alignment: FractionalOffset(0.2, 0.6),
    child: FlutterLogo(
      size: 60,
    ),
  ),
)

```

The actual running effect is shown in Figure 4-13:

![Figure 4-13](https://pcdn.flutterchina.club/imgs/4-13.png)

We will `FractionalOffset(0.2, 0.6)`bring in the coordinate conversion formula and the `FlutterLogo`actual offset is (12, 36), which is consistent with the actual running effect.

## 4.6.2 Comparison between Align and Stack

As you can see, `Align`and `Stack`/ `Positioned`can be used to specify the offset of the child element relative to the parent element, but they still have two main differences:

1.  The positioning reference system is different; `Stack`/The `Positioned`positioning reference system can be the four vertices of the parent container rectangle; and `Align`you need `alignment`to determine the origin of the coordinates by parameters first , different `alignment`will correspond to different origins, the final offset is `alignment`the conversion formula that needs to be passed Calculate.
2.  `Stack`There can be multiple child elements, and child elements can be stacked, but `Align`there can only be one child element, there is no stacking.

## 4.6.3 Center component

We have used `Center`components to center sub-elements in the examples in the previous chapters , and now we officially introduce it. By looking up the SDK source code, we see that the `Center`components are defined as follows:

```
class Center extends Align {
  const Center({ Key key, double widthFactor, double heightFactor, Widget child })
    : super(key: key, widthFactor: widthFactor, heightFactor: heightFactor, child: child);
}

```

You can see that it is `Center`inherited from `Align`, and it has `Align`only one `alignment`parameter less than that ; because `Align`the `alignment`value of the constructor is `Alignment.center`, we can think that the `Center`component is actually aligned ( `Alignment.center`) `Align`.

When we talked about above `widthFactor`or `heightFactor`to `null`the component width and height will take up as much space, it needs special attention, we work through an example:

```
...//省略无关代码
DecoratedBox(
  decoration: BoxDecoration(color: Colors.red),
  child: Center(
    child: Text("xxx"),
  ),
),
DecoratedBox(
  decoration: BoxDecoration(color: Colors.red),
  child: Center(
    widthFactor: 1,
    heightFactor: 1,
    child: Text("xxx"),
  ),
)

```

The running effect is shown in Figure 4-14:

![Figure 4-14](https://pcdn.flutterchina.club/imgs/4-14.png)

## to sum up

This section focuses on the `Align`components and two offset types `Alignment`and `FractionalOffset`, readers need to understand the difference between the two offset types and their respective coordinate conversion formulas. In addition, it is recommended that readers should use `FractionalOffset`it first when they need to formulate some precise offsets , because its coordinate origin is the same as the layout system, which makes it easier to calculate the actual offset.

In the back, we introduced the `Align`assembly and `Stack`/ `Positioned`, `Center`the relationship between the reader can understand the comparison.

Also, students who are familiar with Web development may find that `Align`the characteristics of components are `position: relative`very similar to the relative positioning in Web development ( ), yes! Most of the time, we can directly use `Align`components to achieve the effect of relative positioning in the Web, and readers can compare memory.