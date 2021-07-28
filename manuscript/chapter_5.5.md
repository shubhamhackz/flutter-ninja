# 5.5 Container

We have used `Container`components many times in the examples in the previous chapters . In this section, we will introduce `Container`components in detail . `Container`Is a combination of class container itself does not correspond to specific `RenderObject`, it is `DecoratedBox`, `ConstrainedBox、Transform`, `Padding`, `Align`and other combinations of components a multi-purpose container, so we simply by a `Container`may be implemented simultaneously to be decorated, transformation, restriction scene components. Here is `Container`the definition:

``` dart 
Container({
 this.alignment,
 this.padding, //容器内补白，属于decoration的装饰范围
 Color color, // 背景色
 Decoration decoration, // 背景装饰
 Decoration foregroundDecoration, //前景装饰
 double width,//容器的宽度
 double height, //容器的高度
 BoxConstraints constraints, //容器大小的限制条件
 this.margin,//容器外补白，不属于decoration的装饰范围
 this.transform, //变换
 this.child,
})

```

`Container`Most of the properties have been introduced when introducing other containers, so I won’t repeat them, but there are two points that need to be explained:

-   The size of the container can be specified by `width`and `height`attributes or by `constraints`specifying; if they exist at the same time, `width`and `height`take precedence. In fact, the Container will generate one based on `width`and .`height``constraints`
-   `color`And `decoration`are mutually exclusive, if they are set at the same time, an error will be reported! In fact, when specified `color`, `Container`one is automatically created inside `decoration`.

### Instance

We use `Container`to achieve the card shown in Figure 5-16:

![Figure 5-16](https://pcdn.flutterchina.club/imgs/5-16.png)

The implementation code is as follows:

``` dart 
Container(
 margin: EdgeInsets.only(top: 50.0, left: 120.0), //容器外填充
 constraints: BoxConstraints.tightFor(width: 200.0, height: 150.0), //卡片大小
 decoration: BoxDecoration(//背景装饰
     gradient: RadialGradient( //背景径向渐变
         colors: [Colors.red, Colors.orange],
         center: Alignment.topLeft,
         radius: .98
     ),
     boxShadow: [ //卡片阴影
       BoxShadow(
           color: Colors.black54,
           offset: Offset(2.0, 2.0),
           blurRadius: 4.0
       )
     ]
 ),
 transform: Matrix4.rotationZ(.2), //卡片倾斜变换
 alignment: Alignment.center, //卡片内文字居中
 child: Text( //卡片文字
   "5.20", style: TextStyle(color: Colors.white, fontSize: 40.0),
 ),
);

```

You can see the `Container`functions of a variety of components. By looking at the `Container`source code, we will easily find that it is a combination of the multiple components we introduced earlier. In Flutter, `Container`components are also instances where composition takes precedence over inheritance.

### Padding和Margin

Next, let's study the difference between `Container`components `margin`and `padding`attributes:

``` dart 
...
Container(
 margin: EdgeInsets.all(20.0), //容器外补白
 color: Colors.orange,
 child: Text("Hello world!"),
),
Container(
 padding: EdgeInsets.all(20.0), //容器内补白
 color: Colors.orange,
 child: Text("Hello world!"),
),
...

```

![Figure 5-17](https://pcdn.flutterchina.club/imgs/5-17.png)

It can be found that the intuitive feeling is that `margin`the white space is outside the container, while `padding`the white space is inside the container. Readers need to remember this difference. In fact, the `Container`inner `margin`sum `padding`is achieved through `Padding`components, the above sample code is actually equivalent to:

``` dart 
...
Padding(
 padding: EdgeInsets.all(20.0),
 child: DecoratedBox(
   decoration: BoxDecoration(color: Colors.orange),
   child: Text("Hello world!"),
 ),
),
DecoratedBox(
 decoration: BoxDecoration(color: Colors.orange),
 child: Padding(
   padding: const EdgeInsets.all(20.0),
   child: Text("Hello world!"),
 ),
),
...

```
