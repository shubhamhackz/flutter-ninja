## 5.1 Padding

`Padding`You can add padding (blank) to its child nodes, similar to the margin effect. We have used it in many previous examples, now let's take a look at its definition:

``` dart 
Padding({
 ...
 EdgeInsetsGeometry padding,
 Widget child,
})

```

`EdgeInsetsGeometry`It is an abstract class. In development, we generally use a `EdgeInsets`class. It is `EdgeInsetsGeometry`a subclass of which defines some convenient methods for setting filling.

### EdgeInsets

Let's take a look at `EdgeInsets`the convenient methods provided:

-   `fromLTRB(double left, double top, double right, double bottom)`: Specify the filling in four directions respectively.
-   `all(double value)` : All directions are filled with the same value.
-   `only({left, top, right ,bottom })`: You can set the padding in a specific direction (multiple directions can be specified at the same time).
-   `symmetric({ vertical, horizontal })`: It is used to set the filling, `vertical`finger `top`sum `bottom`, and `horizontal`finger `left`sum of the symmetry direction `right`.

### Example

The following examples mainly show the `EdgeInsets`different usages and are relatively simple. The source code is as follows:

``` dart 
class PaddingTestRoute extends StatelessWidget {
 @override
 Widget build(BuildContext context) {
   return Padding(
     //上下左右各添加16像素补白
     padding: EdgeInsets.all(16.0),
     child: Column(
       //显式指定对齐方式为左对齐，排除对齐干扰
       crossAxisAlignment: CrossAxisAlignment.start,
       children: <Widget>[
         Padding(
           //左边添加8像素补白
           padding: const EdgeInsets.only(left: 8.0),
           child: Text("Hello world"),
         ),
         Padding(
           //上下各添加8像素补白
           padding: const EdgeInsets.symmetric(vertical: 8.0),
           child: Text("I am Jack"),
         ),
         Padding(
           // 分别指定四个方向的补白
           padding: const EdgeInsets.fromLTRB(20.0,.0,20.0,20.0),
           child: Text("Your friend"),
         )
       ],
     ),
   );
 }
}

```

The running effect is shown in Figure 5-1:

![Figure 5-1](https://pcdn.flutterchina.club/imgs/5-1.png)