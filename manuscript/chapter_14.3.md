# 14.3 RenderObject和RenderBox

In the previous section we said that each `Element`corresponds to one `RenderObject`, and we can `Element.renderObject`get it through . And we also said that `RenderObject`the main responsibility is Layout and drawing, all of which `RenderObject`will form a Render Tree. In this section we will focus on `RenderObject`the role.

`RenderObject`It is an object in the rendering tree. It has one `parent`and one `parentData`slot. The so-called slot refers to a reserved interface or position. This interface and position are accessed or occupied by other objects. This interface position or in the software is usually expressed reservation variable, and `parentData`is reserved for a variable, it is made `parent`to the assignment, `parent`usually through the sub `RenderObject`of `parentData`a number of sub-elements and associated data storage such as in the Stack layout, `RenderStack`will Store the offset data of the child element in the child element `parentData`(see the `Positioned`implementation for details ).

`RenderObject`The class itself implements a set of basic layout and drawing protocols, but does not define the child node model (for example, a node can have several child nodes, no child nodes? One? Two? Or more?). It also does not define the coordinate system (such as whether the child node is positioned in Cartesian coordinates or polar coordinates?) and the specific layout protocol (is it through width and height or through constraints and size?, or whether the parent node is placed before the child node or Then set the size and position of the child node, etc.). To this end, Flutter provides a `RenderBox`class, which inherits from it `` `RenderObject``. The layout coordinate system uses Cartesian coordinate system, which is consistent with the native coordinate system of Android and iOS. Both the top and left of the screen are the origin, and then the width and height are two. Axis, in most cases, we can use `RenderBox`it directly , unless we need to customize the layout model or coordinate system, let us focus on it below `RenderBox`.

## 14.3.1 Layout process

### Constraints

In `RenderBox`, there is a `size`property to save the width and height of the control. `RenderBox`The layout is achieved by passing `BoxConstraints`objects from top to bottom in the component tree . `BoxConstraints`The object can limit the maximum and minimum width and height of the child node, and the child node must comply with the restrictions given by the parent node.

In the layout phase, the parent node will call the `layout()`method of the child node . Let's take a look at the general implementation `RenderObject`of the `layout()`method (deleted some irrelevant code and exception capture):

``` dart 
void layout(Constraints constraints, { bool parentUsesSize = false }) {
  ...
  RenderObject relayoutBoundary; 
   if (!parentUsesSize || sizedByParent || constraints.isTight 
       || parent is! RenderObject) {
     relayoutBoundary = this;
   } else {
     final RenderObject parent = this.parent;
     relayoutBoundary = parent._relayoutBoundary;
   }
   ...
   if (sizedByParent) {
       performResize();
   }
   performLayout();
   ...
}

```

It can be seen that the `layout`method needs to pass in two parameters, the first is `constraints`the limit of the size of the child node by the parent node, and the value is determined according to the layout logic of the parent node. Another parameter is `parentUsesSize`that this value is used to determine whether `relayoutBoundary`the child node layout change affects the parent node. If it is `true`, the parent node will be marked as needing to be re-layout when the child node layout changes. If it is `false`, the child node layout will occur. The parent node will not be affected after the change.

#### relayoutBoundary

`layout()`A `relayoutBoundary`variable is defined in the above source code , what is it `relayoutBoundary`? In front of the introduction `Element`, we talked about when a `Element`mark is dirty will re-build, then `RenderObject`will be re-layout, by calling us `markNeedsBuild()`to mark `Element`as dirty is. In `RenderObject`there is a similar `markNeedsLayout()`method, it will `RenderObject`layout the status is marked as dirty, so in the next frame will be re-layout, we look at `RenderObject`the `markNeedsLayout()`part of the source code:

``` dart 
void markNeedsLayout() {
 ...
 assert(_relayoutBoundary != null);
 if (_relayoutBoundary != this) {
   markParentNeedsLayout();
 } else {
   _needsLayout = true;
   if (owner != null) {
     ...
     owner._nodesNeedingLayout.add(this);
     owner.requestVisualUpdate();
   }
 }
}

```

To the code determination logic itself generally is not `relayoutBoundary`, if not continue to look parent, has been to find a direction `relayoutBoundary`of `RenderObject`up, then it is marked as dirty. From this point of view, its role is more obvious, which means that when the size of a control is changed, its parent may be affected, so the parent needs to be re-layouted, so when is it heading? The answer is `relayoutBoundary`, if a `RenderObject`Shi `relayoutBoundary`, it means that it will not change the size affects the size of the parent, so the parent will not have to re-layout.

#### performResize 和 performLayout

`RenderBox`The actual measurement and is a logical layout `performResize()`and `performLayout()`two methods, RenderBox subclasses need to implement these two methods to customize their layout logic. According to `layout()`the source code can be seen only `sizedByParent`as a `true`time, `performResize()`it will be called, and `performLayout()`each time the layout will be called. `sizedByParent`Whether intended only for the size of the node passed to it by the parent of constraints can be identified, namely the size of the node is independent of its own properties and its child nodes, such as if a control is always full size of the parent, then `sizedByParent`it should return `true`at this time in its size `performResize()`in a determines, in the latter `performLayout()`it will no longer be modified method, in which case `performLayout()`only responsible for the layout of the child node.

In the `performLayout()`method in addition to completing the layout itself, it must also complete the layout of child nodes, because the layout process is not actually complete only after the parent-child completed. So the final call stack will become: _layout()> performResize()/performLayout()> child.layout()> ..._ , so the entire UI layout is completed recursively.

`RenderBox`Subclasses who want to customize the layout algorithm should not override the `layout()`method, because for any RenderBox subclass, its layout process is basically the same, and the difference is only in the specific layout algorithm, and the specific layout algorithm subclass should pass rewrite `performResize()`and `performLayout()`two ways to achieve, they will `layout()`be called in.

#### ParentData

When the layout is over, the position of each node (the offset relative to the parent node) has been determined, `RenderObject`and the final drawing can be performed according to the position information. But in the layout process, how to save the location information of the node? For most `RenderBox`subclasses, if the subclass has only one child node, the child node offset is generally the same `Offset.zero`. If there are multiple child nodes, the offset of each child node may be different. The offset data of the child node in the parent node is saved by `RenderObject`the `parentData`attribute. In `RenderBox`, its `parentData`property is an `BoxParentData`object by default , and the property can only be set by the `setupParentData()`method of the parent node :

``` dart 
abstract class RenderBox extends RenderObject {
 @override
 void setupParentData(covariant RenderObject child) {
   if (child.parentData is! BoxParentData)
     child.parentData = BoxParentData();
 }
 ...
}

```

`BoxParentData`It is defined as follows:

``` dart 
/// Parentdata 会被RenderBox和它的子类使用.
class BoxParentData extends ParentData {
 /// offset表示在子节点在父节点坐标系中的绘制偏移  
 Offset offset = Offset.zero;

 @override
 String toString() => 'offset=$offset';
}

```

> It must be noted that `RenderObject`the `parentData`can only be set by the parent element.

Of course, `ParentData`not only it can be used to store the offset information, and all the child nodes generally specific data can be stored in the child node `ParentData`, as `ContainerBox`the `ParentData`it saved a point sibling nodes `previousSibling`and `nextSibling`, `Element.visitChildren()`a method is also achieved by their pairs The traversal of nodes. Another example is the `KeepAlive`component, which uses `KeepAliveParentDataMixin`(inherited from `ParentData`) to save the `keepAlive`state of the subsection .

## 14.3.2 Drawing process

`RenderObject`You can use `paint()`methods to complete the specific drawing logic. The process is similar to the layout process. Subclasses can implement `paint()`methods to complete their own drawing logic. The `paint()`signature is as follows:

``` dart 
void paint(PaintingContext context, Offset offset) { }

```

After `context.canvas`you can get the `Canvas`object, you can call the `Canvas`API to implement the specific drawing logic.

If the node has child nodes, in addition to completing its own drawing logic, it also calls the drawing method of the child nodes. Let's take the `RenderFlex`object as an example:

``` dart 
@override
void paint(PaintingContext context, Offset offset) {

 // 如果子元素未超出当前边界，则绘制子元素  
 if (_overflow <= 0.0) {
   defaultPaint(context, offset);
   return;
 }

 // 如果size为空，则无需绘制
 if (size.isEmpty)
   return;

 // 剪裁掉溢出边界的部分
 context.pushClipRect(needsCompositing, offset, Offset.zero & size, defaultPaint);

 assert(() {
   final String debugOverflowHints = '...'; //溢出提示内容，省略
   // 绘制溢出部分的错误提示样式
   Rect overflowChildRect;
   switch (_direction) {
     case Axis.horizontal:
       overflowChildRect = Rect.fromLTWH(0.0, 0.0, size.width + _overflow, 0.0);
       break;
     case Axis.vertical:
       overflowChildRect = Rect.fromLTWH(0.0, 0.0, 0.0, size.height + _overflow);
       break;
   }  
   paintOverflowIndicator(context, offset, Offset.zero & size,
                          overflowChildRect, overflowHints: debugOverflowHints);
   return true;
 }());
}

```

The code is very simple, first determine whether there is overflow, if not, call `defaultPaint(context, offset)`to complete the drawing, the source code of this method is as follows:

``` dart 
void defaultPaint(PaintingContext context, Offset offset) {
 ChildType child = firstChild;
 while (child != null) {
   final ParentDataType childParentData = child.parentData;
   //绘制子节点， 
   context.paintChild(child, childParentData.offset + offset);
   child = childParentData.nextSibling;
 }
}

```

Obviously, since Flex itself has nothing to draw, it directly traverses its child nodes, and then calls `paintChild()`to draw the child nodes, and at the same time passes `ParentData`the offset saved in the layout phase of the child node plus its own offset as the second parameter `paintChild()`. And if there is a child node child node, `paintChild()`the method also calls the child nodes `paint()`method, so recursive finish drawing the entire tree of nodes, the final call stack is: _Paint ()> paintChild ()> Paint () ..._ .

When the size of the content to be drawn overflows the current space, will be executed `paintOverflowIndicator()`to draw the overflow prompt. This is the overflow prompt we often see, as shown in Figure 14-3:

![overflow](../resources/imgs/14-3.png)

### RepaintBoundary

We have already `CustomPaint`introduced it in the section, `RepaintBoundary`and now we have a deeper understanding. And `RelayoutBoundary`similar `RepaintBoundary`are used in determining the boundaries redrawn, and `RelayoutBoundary`the difference is, the border drawn by the need for the developer `RepaintBoundary`components themselves specify, such as:

``` dart 
CustomPaint(
 size: Size(300, 300), //指定画布大小
 painter: MyPainter(),
 child: RepaintBoundary(
   child: Container(...),
 ),
),

```

Let’s take a look at `RepaintBoundary`the principle. `RenderObject`There is an `isRepaintBoundary`attribute that determines whether the `RenderObject`redraw is independent of its parent element. If the attribute value is `true`, it will be drawn independently, otherwise it will be drawn together. How is independent drawing achieved? The answer is in the `paintChild()`source code:

``` dart 
void paintChild(RenderObject child, Offset offset) {
 ...
 if (child.isRepaintBoundary) {
   stopRecordingIfNeeded();
   _compositeChild(child, offset);
 } else {
   child._paintWithContext(this, offset);
 }
 ...
}

```

We can see that, when drawing a child node, if `child.isRepaintBoundary`is `true`is invoked `_compositeChild()`method, `_compositeChild()`source code is as follows:

``` dart 
void _compositeChild(RenderObject child, Offset offset) {
 // 给子节点创建一个layer ，然后再上面绘制子节点 
 if (child._needsPaint) {
   repaintCompositedChild(child, debugAlsoPaintedParent: true);
 } else {
   ...
 }
 assert(child._layer != null);
 child._layer.offset = offset;
 appendLayer(child._layer);
}

```

Obviously, independent drawing is done by drawing on different layers. Therefore, it is obvious that the correct use of `isRepaintBoundary`attributes can improve drawing efficiency and avoid unnecessary redrawing. The specific principle is: similar to triggering rebuild and layout, `RenderObject`a `markNeedsPaint()`method is also provided . The source code is as follows:

``` dart 
void markNeedsPaint() {
...
 //如果RenderObject.isRepaintBoundary 为true,则该RenderObject拥有layer，直接绘制  
 if (isRepaintBoundary) {
   ...
   if (owner != null) {
     //找到最近的layer，绘制  
     owner._nodesNeedingPaint.add(this);
     owner.requestVisualUpdate();
   }
 } else if (parent is RenderObject) {
   // 没有自己的layer, 会和一个祖先节点共用一个layer  
   assert(_layer == null);
   final RenderObject parent = this.parent;
   // 向父级递归查找  
   parent.markNeedsPaint();
   assert(parent == this.parent);
 } else {
   // 如果直到根节点也没找到一个Layer，那么便需要绘制自身，因为没有其它节点可以绘制根节点。  
   if (owner != null)
     owner.requestVisualUpdate();
 }
}

```

As can be seen, when you call `markNeedsPaint()`upon method, will from current `RenderObject`has begun to find the parent node until you find a `isRepaintBoundary`is `true`the `RenderObject`time to trigger redrawn, so that we can achieve local redrawn. When there is `RenderObject`drawn very frequent or very complex, it is possible to specify by RepaintBoundary Widget `isRepaintBoundary`is `true`, so when drawing only will redraw itself without having to redraw its parent, so can improve performance.

There is another question, through `RepaintBoundary`how to set the `isRepaintBoundary`properties? In fact, if used `RepaintBoundary`, its corresponding `RenderRepaintBoundary`will be automatically `isRepaintBoundary`set `true`to:

``` dart 
class RenderRepaintBoundary extends RenderProxyBox {
 /// Creates a repaint boundary around [child].
 RenderRepaintBoundary({ RenderBox child }) : super(child);

 @override
 bool get isRepaintBoundary => true;
}

```

## 14.3.3 Hit Test

We have already talked about the Flutter event mechanism and hit test process in the chapter "Event Handling and Notification". In this section, let's take a look at its internal implementation principles.

Whether an object can respond to an event depends on its return to the hit test. When a user event occurs, the `RenderView`hit test will start from the root node ( ). The following is `RenderView`the `hitTest()`source code:

``` dart 
bool hitTest(HitTestResult result, { Offset position }) {
 if (child != null)
   child.hitTest(result, position: position); //递归子RenderBox进行命中测试
 result.add(HitTestEntry(this)); //将测试结果添加到result中
 return true;
}

```

Let's look at the `RenderBox`default `hitTest()`implementation:

``` dart 
bool hitTest(HitTestResult result, { @required Offset position }) {
 ...  
 if (_size.contains(position)) {
   if (hitTestChildren(result, position: position) || hitTestSelf(position)) {
     result.add(BoxHitTestEntry(this, position));
     return true;
   }
 }
 return false;
}

```

We see that two methods `hitTestSelf()`and `hitTestChildren()`two are called in the default implementation. The default implementation of these two methods is as follows:

``` dart 

@protected
bool hitTestSelf(Offset position) => false;

@protected
bool hitTestChildren(HitTestResult result, { Offset position }) => false;

```

`hitTest`The method used to determine `RenderObject`whether the range is clicked, clicked at the same time responsible for `RenderBox`adding to the `HitTestResult`list of parameters `position`for the coordinates of the event-triggered (if any), returns true then expressed `RenderBox`by the hit test, you need to respond to events , Otherwise it is considered that `RenderBox`there is no hit currently . In succession `RenderBox`, you can override the direct `hitTest()`method, you can also rewrite `hitTestSelf()`or `hitTestChildren()`, the only difference is `hitTest()`the need to be added to the hit list of test results by the hit test node information, and `hitTestSelf()`and `hitTestChildren()`only need a simple return `true`or `false`.

## 14.3.4 Semantic

Semantics, namely Semantics, is mainly an interface provided to the screen reader software, and is also the basis for realizing auxiliary functions. Through the semantic interface, the machine can understand the content on the page, and users with visual impairment can use the screen reader software to understand UI content. If a `RenderObject`want to support semantic interface that can be achieved `describeApproximatePaintClip`and `visitChildrenForSemantics`the methods and `semanticsAnnotator`getter. More information about semantics can be found in the API documentation.

## 14.3.5 Summary

In this section, we introduced the `RenderObject`main functions and methods. Understanding these contents can help us better understand the underlying principles of Flutter UI. We can also see that if it `RenderObject`is troublesome to implement one from beginning to end , we must implement the layout, drawing and hit test logic, but fortunately, most of the time we can directly combine or `CustomPaint`complete the self in the Widget layer. Define the UI. If you can only define a new `RenderObject`scene (such as a layout container that implements a new layout algorithm), you can directly inherit from it `RenderBox`, which can help us reduce some of the work.