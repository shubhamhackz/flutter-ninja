# 6.4 GridView

`GridView`You can build a two-dimensional grid list, the default constructor is defined as follows:

```
GridView({
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController controller,
  bool primary,
  ScrollPhysics physics,
  bool shrinkWrap = false,
  EdgeInsetsGeometry padding,
  @required SliverGridDelegate gridDelegate, //控制子widget layout的委托
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double cacheExtent,
  List<Widget> children = const <Widget>[],
})

```

We can see, `GridView`and `ListView`most of the parameters are the same, their meanings are also the same, wondering if the reader can flip through a ListView, not discussed here. The only thing we need to pay attention to is the `gridDelegate`parameter, the type is `SliverGridDelegate`, its role is to control `GridView`how the sub-components are arranged (layout).

`SliverGridDelegate`It is an abstract class that defines `GridView`Layout related interfaces, and subclasses need to implement them to implement specific layout algorithms. Flutter provides two `SliverGridDelegate`subclasses `SliverGridDelegateWithFixedCrossAxisCount`and `SliverGridDelegateWithMaxCrossAxisExtent`we can use them directly. Let's introduce them separately.

### SliverGridDelegateWithFixedCrossAxisCount

This subclass implements a layout algorithm with a fixed number of sub-elements on the horizontal axis, and its constructor is:

```
SliverGridDelegateWithFixedCrossAxisCount({
  @required double crossAxisCount, 
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
})

```

-   `crossAxisCount`: The number of child elements on the horizontal axis. After the attribute value is determined, the length of the child element on the horizontal axis is determined, that is, the quotient `crossAxisCount`of the length of the ViewPort horizontal axis divided by the length of the horizontal axis .
-   `mainAxisSpacing`: Spacing in the main axis direction.
-   `crossAxisSpacing`: The spacing of child elements in the horizontal axis direction.
-   `childAspectRatio`: The ratio of the length of the child element on the horizontal axis to the length of the main axis. After `crossAxisCount`specifying, the length of the horizontal axis of the sub-element is determined, and then the length of the sub-element in the main axis can be determined through this parameter value.

Can be found, the size of the child elements is through `crossAxisCount`and the `childAspectRatio`two parameters joint decision. Note that the child element here refers to the maximum display space of the child component. Take care to ensure that the actual size of the child component does not exceed the space of the child element.

Let's look at an example:

```
GridView(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 3, //横轴三个子widget
      childAspectRatio: 1.0 //宽高比为1时，子widget
  ),
  children:<Widget>[
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast)
  ]
);

```

![Figure 6-9](https://pcdn.flutterchina.club/imgs/6-9.png)

#### GridView.count

`GridView.count`The constructor is used internally `SliverGridDelegateWithFixedCrossAxisCount`, and we can quickly create a fixed number of sub-elements on the horizontal axis through it `GridView`. The above sample code is equivalent to:

```
GridView.count( 
  crossAxisCount: 3,
  childAspectRatio: 1.0,
  children: <Widget>[
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast),
  ],
);

```

### SliverGridDelegateWithMaxCrossAxisExtent

This subclass implements a layout algorithm with a fixed maximum length for the horizontal axis sub-elements, and its constructor is:

```
SliverGridDelegateWithMaxCrossAxisExtent({
  double maxCrossAxisExtent,
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
})

```

`maxCrossAxisExtent`Is the maximum length of the child element on the horizontal axis. The reason for the “maximum” length is **that the length of each child element in the horizontal axis direction is still equally divided** . For example, if the horizontal axis length of ViewPort is 450, then when `maxCrossAxisExtent`If the value of is in the interval [450/4, 450/3), the final actual length of the sub-element is 112.5, and `childAspectRatio`the length ratio between the horizontal axis and the main axis of the sub-element referred to is the **final length ratio** . Other parameters are the `SliverGridDelegateWithFixedCrossAxisCount`same.

Let's look at an example:

```
GridView(
  padding: EdgeInsets.zero,
  gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
      maxCrossAxisExtent: 120.0,
      childAspectRatio: 2.0 //宽高比为2
  ),
  children: <Widget>[
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast),
  ],
);

```

![Figure 6-10](https://pcdn.flutterchina.club/imgs/6-10.png)

#### GridView.extent

The GridView.extent constructor uses SliverGridDelegateWithMaxCrossAxisExtent internally, through which we can quickly create a GridView with a fixed maximum length for the vertical axis child element. The above sample code is equivalent to:

```
GridView.extent(
   maxCrossAxisExtent: 120.0,
   childAspectRatio: 2.0,
   children: <Widget>[
     Icon(Icons.ac_unit),
     Icon(Icons.airport_shuttle),
     Icon(Icons.all_inclusive),
     Icon(Icons.beach_access),
     Icon(Icons.cake),
     Icon(Icons.free_breakfast),
   ],
 );

```

### GridView.builder

The GridView we introduced above requires a widget array as its child elements. These methods will build all child widgets in advance, so it is only applicable when the number of child widgets is relatively small. When there are more child widgets, we can `GridView.builder`dynamically create child widgets. widget. `GridView.builder`There are two parameters that must be specified:

```
GridView.builder(
 ...
 @required SliverGridDelegate gridDelegate, 
 @required IndexedWidgetBuilder itemBuilder,
)

```

Among them `itemBuilder`is the child widget builder.

#### Example

Suppose we need to get some batches asynchronous data from a source (such as a network) `Icon`, and then `GridView`to show:

```
class InfiniteGridView extends StatefulWidget {
  @override
  _InfiniteGridViewState createState() => new _InfiniteGridViewState();
}

class _InfiniteGridViewState extends State<InfiniteGridView> {

  List<IconData> _icons = []; //保存Icon数据

  @override
  void initState() {
    // 初始化数据  
    _retrieveIcons();
  }

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 3, //每行三列
            childAspectRatio: 1.0 //显示区域宽高相等
        ),
        itemCount: _icons.length,
        itemBuilder: (context, index) {
          //如果显示到最后一个并且Icon总数小于200时继续获取数据
          if (index == _icons.length - 1 && _icons.length < 200) {
            _retrieveIcons();
          }
          return Icon(_icons[index]);
        }
    );
  }

  //模拟异步获取数据
  void _retrieveIcons() {
    Future.delayed(Duration(milliseconds: 200)).then((e) {
      setState(() {
        _icons.addAll([
          Icons.ac_unit,
          Icons.airport_shuttle,
          Icons.all_inclusive,
          Icons.beach_access, Icons.cake,
          Icons.free_breakfast
        ]);
      });
    });
  }
}

```

-   `_retrieveIcons()`: In this method, we `Future.delayed`simulate obtaining data from an asynchronous data source. It takes 200 milliseconds to obtain data each time. After obtaining the data successfully, add new data to _icons, and then call setState to rebuild.
-   In itemBuilder, if the last one is displayed, it is judged whether it is necessary to continue to obtain data, and then an Icon is returned.

### More

Flutter's `GridView`default sub-elements display space is equal, but in actual development, you may encounter a situation where the size of sub-elements is not equal, such as the following layout:

![Figure 6-11](https://pcdn.flutterchina.club/imgs/6-11.png)

There is a package "flutter_staggered_grid_view" on Pub, which implements a staggered GridView layout model. This layout can be easily implemented. Readers can understand the details.