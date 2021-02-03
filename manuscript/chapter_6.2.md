# 6.2 SingleChildScrollView

`SingleChildScrollView`Similar to Android `ScrollView`, it can only receive one child component. It is defined as follows:

```
SingleChildScrollView({
  this.scrollDirection = Axis.vertical, //滚动方向，默认是垂直方向
  this.reverse = false, 
  this.padding, 
  bool primary, 
  this.physics, 
  this.controller,
  this.child,
})

```

In addition to the common attributes we introduced a scrollable components, we look at the focus `reverse`and `primary`two attributes:

-   `reverse`: The API documentation for this attribute explains whether to slide in the opposite direction of the reading direction. For example, the `scrollDirection`value is `Axis.horizontal`if the reading direction is from left to right (depending on the locale, Arabic is from right to left). `reverse`For the `true`time, then the sliding direction is from right to left. In fact, this attribute essentially determines whether the initial scroll position of the scrollable component is at the "head" or "tail". When it is taken `false`, the initial scroll position is at the "head", otherwise it is at the "tail". Readers can experiment by themselves.
-   `primary`: Refers to whether to use the default in the widget tree `PrimaryScrollController`; when the sliding direction is vertical ( `scrollDirection`value `Axis.vertical`) and not specified `controller`, the `primary`default is `true`.

It should be noted that `SingleChildScrollView`it should usually only be used when the expected content does not exceed the screen too much. This is because the `SingleChildScrollView`Sliver-based delayed instantiation model is not supported, so if the viewport is expected to contain too much content beyond the screen size, then It `SingleChildScrollView`will be very expensive to use (poor performance). At this time, you should use some scrollable components that support Sliver delay loading, such as `ListView`.

### Example

The following is an example of displaying the uppercase letters AZ in the vertical direction. Since the vertical space will exceed the height of the screen viewport, we use `SingleChildScrollView`:

```
class SingleChildScrollViewTestRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    return Scrollbar( // 显示进度条
      child: SingleChildScrollView(
        padding: EdgeInsets.all(16.0),
        child: Center(
          child: Column( 
            //动态创建一个List<Widget>  
            children: str.split("") 
                //每一个字母都用一个Text显示,字体为原来的两倍
                .map((c) => Text(c, textScaleFactor: 2.0,)) 
                .toList(),
          ),
        ),
      ),
    );
  }
}

```

The running effect is shown in Figure 6-1:

![Figure 6-1](https://pcdn.flutterchina.club/imgs/6-1.png)