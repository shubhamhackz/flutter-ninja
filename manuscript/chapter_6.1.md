# 6.1 Introduction to scrollable components

When the component content exceeds the currently displayed viewport (ViewPort), if there is no special handling, Flutter will prompt an Overflow error. To this end, Flutter provides a variety of Scrollable Widgets for displaying lists and long layouts. In this chapter, we first introduce the commonly used scrollable components (such as `ListView`, `GridView`etc.), and then introduce them `ScrollController`. Scrollable components include a `Scrollable`component directly or indirectly , so they include some common properties. To avoid repeating the introduction, we will introduce it here:

``` dart 
Scrollable({
 ...
 this.axisDirection = AxisDirection.down,
 this.controller,
 this.physics,
 @required this.viewportBuilder, //后面介绍
})

```

-   `axisDirection`Scroll direction.
-   `physics`: This property accepts a `ScrollPhysics`type of object, which determines how the scrollable component responds to user operations. For example, after the user slides and lifts the finger, continue to perform the animation; or how to display when it slides to the boundary. By default, Flutter will use different `ScrollPhysics`objects according to the specific platform, and apply different display effects. For example, if you continue to drag when sliding to the border, there will be an elastic effect on iOS, and a shimmer effect on Android. . If you want to use the same effect on all platforms, you can explicitly specify a fixed one `ScrollPhysics`. The Flutter SDK contains two `ScrollPhysics`subclasses, which can be used directly:
   -   `ClampingScrollPhysics`: Shimmer effect under Android.
   -   `BouncingScrollPhysics`: Elastic effect under iOS.
-   `controller`: This attribute accepts an `ScrollController`object. `ScrollController`The main function is to control the scroll position and monitor scroll events. By default, there will be a default in the Widget tree `PrimaryScrollController`. If the scrollable component in the subtree is not explicitly specified `controller`and the `primary`attribute value is `true`time (the default is `true`), the scrollable component will use this default `PrimaryScrollController`. The advantage of this mechanism is that the parent component can control the scrolling behavior of the scrollable component in the subtree. For example, `Scaffold`it is this mechanism that implements the function of clicking the navigation bar to return to the top in iOS. We will introduce it in detail in the "Scroll Control" section later in this chapter `ScrollController`.

### Scrollbar

`Scrollbar`It is a Material style scroll indicator (scroll bar). If you want to add a scroll bar to a scrollable component, you only need to `Scrollbar`use any parent component of the scrollable component, such as:

``` dart 
Scrollbar(
 child: SingleChildScrollView(
   ...
 ),
);

```

`Scrollbar`And `CupertinoScrollbar`both determine the position of the scroll bar by listening to the scroll notification. We will introduce the details of the rolling notification in the last section of this chapter.

#### Cupertino Scrollbar

`CupertinoScrollbar`It is an iOS style scroll bar. If you are using `Scrollbar`it, it will automatically switch to on the iOS platform `CupertinoScrollbar`.

### ViewPort

There is the concept of ViewPort in many layout systems. In Flutter, the term ViewPort (viewport), unless otherwise specified, refers to the actual display area of ​​a Widget. For example, the `ListView`height of a display area is 800 pixels, although the total height of its list items may far exceed 800 pixels, its ViewPort is still 800 pixels.

### Delayed construction based on Sliver

Usually the scrollable component may have a lot of sub-components and occupy a very large total height; it will be very expensive to build all the sub-components at once! For this reason, Flutter proposes a concept of Sliver (meaning "slice" in Chinese). If a scrollable component supports the Sliver model, then the scrolling can divide the sub-components into multiple "slivers" (Sliver), only when Sliver appears in It is built only when it is in the viewport. This model is also called "Sliver-based delayed construction model". Scrollable component has a lot of support to build model based on Sliver delay, such as `ListView`, `GridView`but also does not support the model, such as `SingleChildScrollView`.

### Main axis and vertical axis

In the coordinate description of scrollable components, the rolling direction is usually called the main axis, and the non-rolling direction is called the vertical axis. Since the default direction of scrollable components is generally along the vertical direction, the main axis refers to the vertical direction by default, and the horizontal direction is the same.