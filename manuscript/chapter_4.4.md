# 4.4 Flow layout

When introducing Row and Colum, if the child widget exceeds the screen range, an overflow error will be reported, such as:

``` dart 
Row(
 children: <Widget>[
   Text("xxx"*100)
 ],
);

```

The running effect is shown in Figure 4-6:

![Figure 4-6](../resources/imgs/4-6.png)

As you can see, an error is reported in the overflow part on the right. This is because Row has only one line by default, and will not wrap if it exceeds the screen. We call a layout that automatically wraps lines beyond the display range of the screen as a stream layout. Flutter supports streaming layout through `Wrap`and `Flow`. After replacing Row in the above example with Wrap, the overflow part will automatically wrap. Below we introduce `Wrap`and respectively `Flow`.

## 4.4.1 Wrap

Here is the definition of Wrap:

``` dart 
Wrap({
 ...
 this.direction = Axis.horizontal,
 this.alignment = WrapAlignment.start,
 this.spacing = 0.0,
 this.runAlignment = WrapAlignment.start,
 this.runSpacing = 0.0,
 this.crossAxisAlignment = WrapCrossAlignment.start,
 this.textDirection,
 this.verticalDirection = VerticalDirection.down,
 List<Widget> children = const <Widget>[],
})

```

We can see a lot Wrap property `Row`(including `Flex`and `Column`) also has, such as `direction`, `crossAxisAlignment`, `textDirection`, `verticalDirection`etc. These parameters are the same sense, we will not repeat the introduction, the reader may consult introduced earlier `Row`section. Readers can think that `Wrap`and `Flex`(including `Row`and `Column`) `Wrap`behave basically the same, except that they will wrap after going beyond the display range . Let's take a look at `Wrap`a few unique attributes:

-   `spacing`: The spacing of the sub-widgets in the spindle direction
-   `runSpacing`: Spacing in the vertical axis direction
-   `runAlignment`: Alignment of the vertical axis

Let's look at an example:

``` dart 
Wrap(
 spacing: 8.0, // 主轴(水平)方向间距
 runSpacing: 4.0, // 纵轴（垂直）方向间距
 alignment: WrapAlignment.center, //沿主轴方向居中
 children: <Widget>[
   new Chip(
     avatar: new CircleAvatar(backgroundColor: Colors.blue, child: Text('A')),
     label: new Text('Hamilton'),
   ),
   new Chip(
     avatar: new CircleAvatar(backgroundColor: Colors.blue, child: Text('M')),
     label: new Text('Lafayette'),
   ),
   new Chip(
     avatar: new CircleAvatar(backgroundColor: Colors.blue, child: Text('H')),
     label: new Text('Mulligan'),
   ),
   new Chip(
     avatar: new CircleAvatar(backgroundColor: Colors.blue, child: Text('J')),
     label: new Text('Laurens'),
   ),
 ],
)

```

The running effect is shown in Figure 4-7:

![Figure 4-7](../resources/imgs/4-7.png)

## 4.4.2 Flow

We generally rarely use `Flow`it because it is too complicated and needs to implement the position conversion of the sub-widgets by ourselves. In many scenarios, the first thing to consider is `Wrap`whether to meet the needs. `Flow`Mainly used in some scenes that require custom layout strategies or high performance requirements (such as in animation). `Flow`Has the following advantages:

-   Good performance; `Flow`is a sub-assembly to adjust the size and position of a very efficient control, `Flow`position adjustment in the sub-assembly for conversion matrix when optimized: In `Flow`After positioning, if the size or position of the sub-assembly has changed, in `FlowDelegate`the In the `paintChildren()`method of calling `context.paintChild`to redraw, `context.paintChild`the transformation matrix is ​​used in the redrawing, and the component position is not actually adjusted.
-   Flexible; because we need to implement `FlowDelegate`the `paintChildren()`method ourselves , we need to calculate the position of each component ourselves, so we can customize the layout strategy.

Disadvantages:

-   Complex to use.
-   Not adaptive subassembly size must be achieved by specifying the size of the parent container or `TestFlowDelegate`a `getSize`return to a fixed size.

Example:

We customize the flow layout of six color blocks:

``` dart 
Flow(
 delegate: TestFlowDelegate(margin: EdgeInsets.all(10.0)),
 children: <Widget>[
   new Container(width: 80.0, height:80.0, color: Colors.red,),
   new Container(width: 80.0, height:80.0, color: Colors.green,),
   new Container(width: 80.0, height:80.0, color: Colors.blue,),
   new Container(width: 80.0, height:80.0,  color: Colors.yellow,),
   new Container(width: 80.0, height:80.0, color: Colors.brown,),
   new Container(width: 80.0, height:80.0,  color: Colors.purple,),
 ],
)

```

Implement TestFlowDelegate:

``` dart 
class TestFlowDelegate extends FlowDelegate {
 EdgeInsets margin = EdgeInsets.zero;
 TestFlowDelegate({this.margin});
 @override
 void paintChildren(FlowPaintingContext context) {
   var x = margin.left;
   var y = margin.top;
   //计算每一个子widget的位置  
   for (int i = 0; i < context.childCount; i++) {
     var w = context.getChildSize(i).width + x + margin.right;
     if (w < context.size.width) {
       context.paintChild(i,
           transform: new Matrix4.translationValues(
               x, y, 0.0));
       x = w + margin.left;
     } else {
       x = margin.left;
       y += context.getChildSize(i).height + margin.top + margin.bottom;
       //绘制子widget(有优化)  
       context.paintChild(i,
           transform: new Matrix4.translationValues(
               x, y, 0.0));
        x += context.getChildSize(i).width + margin.left + margin.right;
     }
   }
 }

 @override
 getSize(BoxConstraints constraints){
   //指定Flow的大小  
   return Size(double.infinity,200.0);
 }

 @override
 bool shouldRepaint(FlowDelegate oldDelegate) {
   return oldDelegate != this;
 }
}

```

The running effect is shown in Figure 4-8:

![Figure 4-8](../resources/imgs/4-8.png)

It can be seen that our main task is to achieve `paintChildren`, and its main task is to determine the location of each child widget. Since Flow cannot adapt to the size of child widgets, we `getSize`specify the size of Flow by returning a fixed size.