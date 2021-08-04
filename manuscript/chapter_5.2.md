# 5.2 Restricted size container

Size constraints limit the type of container to container size, provided Flutter plurality of such containers, such as `ConstrainedBox`, `SizedBox`, `UnconstrainedBox`, `AspectRatio`etc. This section describes some common.

## 5.2.1 ConstrainedBox

`ConstrainedBox`Used to add additional constraints to subcomponents. For example, if you want the minimum height of the subcomponent to be 80 pixels, you can use `const BoxConstraints(minHeight: 80.0)`the constraint as the subcomponent.

#### Example

Let's define one first `redBox`, it is a box with a red background color, without specifying its width and height:

``` dart 
Widget redBox=DecoratedBox(
 decoration: BoxDecoration(color: Colors.red),
);

```

We implement a red container with a minimum height of 50 and a width as large as possible.

``` dart 
ConstrainedBox(
 constraints: BoxConstraints(
   minWidth: double.infinity, //宽度尽可能大
   minHeight: 50.0 //最小高度为50像素
 ),
 child: Container(
     height: 5.0, 
     child: redBox 
 ),
)

```

The running effect is shown in Figure 5-2:

![Figure 5-2](../resources/imgs/5-2.png)

As you can see, although we set the height of the Container to 5 pixels, it ended up being 50 pixels. This is where the minimum height limit of the ConstrainedBox comes into effect. If the height of the Container is set to 80 pixels, the final height of the red area will also be 80 pixels, because in this example, the ConstrainedBox only limits the minimum height, not the maximum height.

#### BoxConstraints

BoxConstraints is used to set constraints, and its definition is as follows:

``` dart 
const BoxConstraints({
 this.minWidth = 0.0, //最小宽度
 this.maxWidth = double.infinity, //最大宽度
 this.minHeight = 0.0, //最小高度
 this.maxHeight = double.infinity //最大高度
})

```

BoxConstraints also defines some convenient constructors to quickly generate BoxConstraints with specific restriction rules. For example `BoxConstraints.tight(Size size)`, it can generate constraints of a given size; it `const BoxConstraints.expand()`can generate a BoxConstraints as large as possible to fill another container. In addition, there are some other convenient functions, readers can check the [API documentation](https://docs.flutter.io/flutter/rendering/BoxConstraints-class.html) .

## 5.2.2 SizedBox

`SizedBox`Used to specify a fixed width and height for child elements, such as:

``` dart 
SizedBox(
 width: 80.0,
 height: 80.0,
 child: redBox
)

```

The running effect is shown in Figure 5-3:

![Figure 5-3](../resources/imgs/5-3.png)

In fact, it `SizedBox`is just `ConstrainedBox`a customization. The above code is equivalent to:

``` dart 
ConstrainedBox(
 constraints: BoxConstraints.tightFor(width: 80.0,height: 80.0),
 child: redBox, 
)

```

And is `BoxConstraints.tightFor(width: 80.0,height: 80.0)`equivalent to:

``` dart 
BoxConstraints(minHeight: 80.0,maxHeight: 80.0,minWidth: 80.0,maxWidth: 80.0)

```

And in fact, `ConstrainedBox`and `SizedBox`are through `RenderConstrainedBox`to rendering, we can see `ConstrainedBox`and `SizedBox`the `createRenderObject()`methods return is a `RenderConstrainedBox`target:

``` dart 
@override
RenderConstrainedBox createRenderObject(BuildContext context) {
 return new RenderConstrainedBox(
   additionalConstraints: ...,
 );
}

```

## 5.2.3 Multiple restrictions

If a component has multiple parent `ConstrainedBox`restrictions, which one will take effect? Let's look at an example:

``` dart 
ConstrainedBox(
   constraints: BoxConstraints(minWidth: 60.0, minHeight: 60.0), //父
   child: ConstrainedBox(
     constraints: BoxConstraints(minWidth: 90.0, minHeight: 20.0),//子
     child: redBox,
   )
)

```

We have two fathers and sons above `ConstrainedBox`, and their restriction conditions are different. The effect after running is shown in Figure 5-4:

![Figure 5-4](../resources/imgs/5-4.png)

The final display is wide 90, high 60, that is a child `ConstrainedBox`'s `minWidth`entry into force, which `minHeight`is the parent `ConstrainedBox`to take effect. Based on this example alone, we still can’t sum up any rules. Let’s change the parent-child restrictions in the previous example:

``` dart 
ConstrainedBox(
   constraints: BoxConstraints(minWidth: 90.0, minHeight: 20.0),
   child: ConstrainedBox(
     constraints: BoxConstraints(minWidth: 60.0, minHeight: 60.0),
     child: redBox,
   )
)

```

The running effect is shown in Figure 5-5:

![Figure 5-5](../resources/imgs/5-5.png)

The final display is still 90, high 60, the same effect, but with different meaning, because the `minWidth`entry into force of the father `ConstrainedBox`, and `minHeight`a child `ConstrainedBox`to take effect.

Through the above example, we find that when there are multiple restrictions, for the `minWidth`sum `minHeight`, the corresponding value of the parent and child is the larger. In fact, the only way to ensure that the parent restriction does not conflict with the child restriction.

> Questions: For `maxWidth`and `maxHeight`multiple restriction strategy is what it?

## 5.2.4 UnconstrainedBox

`UnconstrainedBox`Does not impose any restrictions on the sub-components, it allows its sub-components to be drawn according to their own size. Under normal circumstances, we will rarely use this component directly, but it may be helpful to "remove" multiple restrictions. Let's look at the following code:

``` dart 
ConstrainedBox(
   constraints: BoxConstraints(minWidth: 60.0, minHeight: 100.0),  //父
   child: UnconstrainedBox( //“去除”父级限制
     child: ConstrainedBox(
       constraints: BoxConstraints(minWidth: 90.0, minHeight: 20.0),//子
       child: redBox,
     ),
   )
)

```

In the above code, if there is no intermediate `UnconstrainedBox`, then according to the multiple restriction rules described above, a 90×100 red box will eventually be displayed. But because the limit of `UnconstrainedBox`the parent `ConstrainedBox`is "removed" , it will eventually be `ConstrainedBox`drawn according to the limit of the child `redBox`, that is, 90×20:

![Figure 5-6](../resources/imgs/5-6.png)

However, readers please note that `UnconstrainedBox`the "removal" of the parent component restriction is not a true removal: although the size of the red area in the above example is 90×20, there is still 80 empty space above it. That is to say, the parent limit `minHeight`(100.0) is still in effect, but it does not affect `redBox`the size of the final child element , but still occupies the corresponding space. It can be considered that the parent at this time `ConstrainedBox`acts on the child `UnconstrainedBox`, and is `redBox`only `ConstrainedBox`restricted by the child. , Readers must pay attention to this point.

So is there any way to completely remove `ConstrainedBox`the restriction of the parent ? the answer is negative! Therefore, the reader is reminded that when defining a common component, if you want to specify restrictions on the sub-components, you must pay attention, because once the restriction conditions are specified, it may be very difficult to customize the size of the sub-components. A component cannot completely remove its restrictions without changing the code of the parent component.

In actual development, when we find that we have used `SizedBox`or `ConstrainedBox`specified the width and height of the child element, but there is still no effect, we can almost conclude that there are already restrictions on the parent element! For example, in the `AppBar`right menu of the (navigation bar) in the Material component library , we use to `SizedBox`specify the size of the loading button. The code is as follows:

``` dart 
AppBar(
  title: Text(title),
  actions: <Widget>[
        SizedBox(
            width: 20, 
            height: 20,
            child: CircularProgressIndicator(
                strokeWidth: 3,
                valueColor: AlwaysStoppedAnimation(Colors.white70),
            ),
        )
  ],
)

```

After the above code runs, the effect is shown in Figure 5-7:

![Figure 5-6](../resources/imgs/5-7.png)

We will find that the size of the loading button on the right has not changed! This is precisely because `AppBar`the `actions`button restrictions have been specified in , so if we want to customize the size of the loading button, we must `UnconstrainedBox`"remove" the restriction of the parent element. The code is as follows:

``` dart 
AppBar(
 title: Text(title),
 actions: <Widget>[
     UnconstrainedBox(
           child: SizedBox(
             width: 20,
             height: 20,
             child: CircularProgressIndicator(
               strokeWidth: 3,
               valueColor: AlwaysStoppedAnimation(Colors.white70),
             ),
         ),
     )
 ],
)

```

The effect after running is shown in Figure 5-8:

![Figure 5-8](../resources/imgs/5-8.png)

It works!

## 5.2.4 Other size restricted containers

In addition to the commonly used size-restricted containers described above, there are some other size-restricted containers. For example `AspectRatio`, it can specify the aspect ratio of the child component, `LimitedBox`specify the maximum width and height, and `FractionallySizedBox`can be based on the percentage of the parent container's width and height. To set the width and height of the sub-components, since these containers are relatively simple to use, we will not repeat them, and readers can understand by themselves.