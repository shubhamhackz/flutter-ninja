# 6.5 CustomScrollView

`CustomScrollView`It is a component that can use Sliver to customize the scrolling model (effect). It can contain multiple scrolling models. For example, suppose there is a page, one at the top and one `GridView`at the bottom `ListView`, and the sliding effect of the entire page is required to be uniform, that is, they look as a whole. If you use `GridView`+ `ListView`to achieve, you cannot guarantee a consistent sliding effect, because their scrolling effects are separated, so you need a "glue" to "stick" these independent scrollable components, and `CustomScrollView`the function It is equivalent to "glue".

### Sliver version of scrollable components

As mentioned earlier, Sliver means thin slices and slices. In Flutter, Sliver usually refers to scrollable component sub-elements (just like slices). However `CustomScrollView`, the scrollable components that need to be glued `CustomScrollView`are Sliver. If you directly use `ListView`and `GridView`as `CustomScrollView`a component , it will not work, because they are scrollable components and not Sliver! Therefore, in order to enable scrollable components to be `CustomScrollView`used together, Flutter provides some Sliver versions of scrollable components, such as SliverList, SliverGrid, etc. In fact, the biggest difference between the Sliver version of the scrollable component and the non-Sliver version of the scrollable component is that the **former does not include a scrolling model (it cannot scroll anymore), while the latter includes a scrolling model** . Because of this, `CustomScrollView`multiple Slivers can be combined." stick "together, these common Sliver `CustomScrollView`of `Scrollable`, so they finally achieve a uniform sliding effect.

> There are many Sliver series of Widgets, we will not introduce them one by one. Readers only need to remember its characteristics and check the documents when needed. The reason why "most" Slivers are said to correspond to scrollable components above is because there are some such as SliverPadding, SliverAppBar, etc. that are not related to scrollable components. They are mainly used in conjunction with CustomScrollView, because **of the subcomponents of CustomScrollView Must be Sliver** .

### Example

``` dart 
import 'package:flutter/material.dart';

class CustomScrollViewTestRoute extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   //因为本路由没有使用Scaffold，为了让子级Widget(如Text)使用
   //Material Design 默认的样式风格,我们使用Material作为本路由的根。
   return Material(
     child: CustomScrollView(
       slivers: <Widget>[
         //AppBar，包含一个导航栏
         SliverAppBar(
           pinned: true,
           expandedHeight: 250.0,
           flexibleSpace: FlexibleSpaceBar(
             title: const Text('Demo'),
             background: Image.asset(
               "./images/avatar.png", fit: BoxFit.cover,),
           ),
         ),

         SliverPadding(
           padding: const EdgeInsets.all(8.0),
           sliver: new SliverGrid( //Grid
             gridDelegate: new SliverGridDelegateWithFixedCrossAxisCount(
               crossAxisCount: 2, //Grid按两列显示
               mainAxisSpacing: 10.0,
               crossAxisSpacing: 10.0,
               childAspectRatio: 4.0,
             ),
             delegate: new SliverChildBuilderDelegate(
                   (BuildContext context, int index) {
                 //创建子widget      
                 return new Container(
                   alignment: Alignment.center,
                   color: Colors.cyan[100 * (index % 9)],
                   child: new Text('grid item $index'),
                 );
               },
               childCount: 20,
             ),
           ),
         ),
         //List
         new SliverFixedExtentList(
           itemExtent: 50.0,
           delegate: new SliverChildBuilderDelegate(
                   (BuildContext context, int index) {
                 //创建列表项      
                 return new Container(
                   alignment: Alignment.center,
                   color: Colors.lightBlue[100 * (index % 9)],
                   child: new Text('list item $index'),
                 );
               },
               childCount: 50 //50个列表项
           ),
         ),
       ],
     ),
   );
 }
}

```

The code is divided into three parts:

-   Head `SliverAppBar`: `SliverAppBar`Corresponding `AppBar`, the difference between the two is that they `SliverAppBar`can be integrated into `CustomScrollView`. `SliverAppBar`It can be combined to `FlexibleSpaceBar`realize the head telescopic model in Material Design. Readers can run this example to view the specific effects.
-   Middle `SliverGrid`: It uses `SliverPadding`wrap to `SliverGrid`add filler. `SliverGrid`It is a two-column grid with an aspect ratio of 4. It has 20 sub-components.
-   Bottom `SliverFixedExtentList`: It is a list with all child elements of 50 pixels in height.

The running effect is as follows:

![Figure 6-12](https://pcdn.flutterchina.club/imgs/6-12.png)![Figure 6-13](https://pcdn.flutterchina.club/imgs/6-13.png)