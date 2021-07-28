# 5.7 Clip

Flutter provides some trim functions for trimming components.

| Tailor Widget | Effect                                                                                                                                  |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| ClipOval      | When the sub-component is square, it will be cut into an inner circle, and when it is a rectangle, it will be cut into an inner ellipse |
| ClipRRect     | Cut the subcomponent into a rounded rectangle                                                                                           |
| ClipRect      | Trim the sub-components to the actual size of the rectangle occupied (the overflow part is trimmed)                                     |


Let's look at an example:

``` dart 
import 'package:flutter/material.dart';

class ClipTestRoute extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   // 头像  
   Widget avatar = Image.asset("imgs/avatar.png", width: 60.0);
   return Center(
     child: Column(
       children: <Widget>[
         avatar, //不剪裁
         ClipOval(child: avatar), //剪裁为圆形
         ClipRRect( //剪裁为圆角矩形
           borderRadius: BorderRadius.circular(5.0),
           child: avatar,
         ), 
         Row(
           mainAxisAlignment: MainAxisAlignment.center,
           children: <Widget>[
             Align(
               alignment: Alignment.topLeft,
               widthFactor: .5,//宽度设为原来宽度一半，另一半会溢出
               child: avatar,
             ),
             Text("你好世界", style: TextStyle(color: Colors.green),)
           ],
         ),
         Row(
           mainAxisAlignment: MainAxisAlignment.center,
           children: <Widget>[
             ClipRect(//将溢出部分剪裁
               child: Align(
                 alignment: Alignment.topLeft,
                 widthFactor: .5,//宽度设为原来宽度一半
                 child: avatar,
               ),
             ),
             Text("你好世界",style: TextStyle(color: Colors.green))
           ],
         ),
       ],
     ),
   );
 }
}

```

The running effect is shown in Figure 5-24:

![Figure 5-24](https://pcdn.flutterchina.club/imgs/5-24.png)

The above sample code comments are more detailed, so I won’t repeat them here. But it is worth mentioning the last two `Row`! After they are `Align`set `widthFactor`to 0.5, the actual width of the picture is equal to 60×0.5, which is half of the original width, but the overflow part of the picture will still be displayed at this time, so the first "Hello World" will overlap with another part of the picture for cropping `Row`To `ClipRect`remove the overflow part, we cut off the overflow part in the second one .

### CustomClipper

If we want to crop a specific area of ​​the sub-component, for example, in the image of the example above, what should we do if we only want to capture the 40×30 pixels in the middle of the image? At this time, we can use to `CustomClipper`customize the clipping area, the implementation code is as follows:

First, customize one `CustomClipper`:

``` dart 
class MyClipper extends CustomClipper<Rect> {
 @override
 Rect getClip(Size size) => Rect.fromLTWH(10.0, 15.0, 40.0, 30.0);

 @override
 bool shouldReclip(CustomClipper<Rect> oldClipper) => false;
}

```

-   `getClip()`It is the interface used to obtain the cropping area. Since the size of the picture is 60×60, we return the cropping area as the `Rect.fromLTWH(10.0, 15.0, 40.0, 30.0)`40×30 pixel range in the middle of the picture.
-   `shouldReclip()`The interface decides whether to re-cut. If in the application, the clipping area should always be returned when there is no change `false`, so that it will not trigger re-clipping and avoid unnecessary performance overhead. If the cropping area changes (for example, an animation is performed on the cropping area), then after the change, you should return `true`to perform the crop again.

Then, we use `ClipRect`to perform cropping. In order to see the actual position of the picture, we set a red background:

``` dart 
DecoratedBox(
 decoration: BoxDecoration(
   color: Colors.red
 ),
 child: ClipRect(
     clipper: MyClipper(), //使用自定义的clipper
     child: avatar
 ),
)

```

The running effect is shown in Figure 5-25:

![Figure 5-25](https://pcdn.flutterchina.club/imgs/5-25.png)

It can be seen that our cropping was successful, but the size of the space occupied by the picture is still 60×60 (red area). This is because the cropping is done in the drawing stage after the layout is completed, so it will not affect the size of the component. And the `Transform`principle is similar.