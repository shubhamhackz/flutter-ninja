# 4.1 Introduction to layout components

Layout components will contain one or more subcomponents, and different layout components have different layouts for the subcomponents. We said earlier that the `Element`tree is the final drawing tree. The `Element`tree is created (through `Widget.createElement()`) the Widget tree . Widget is actually the configuration data of Element. In Flutter, Widgets are divided into three categories according to whether they need to contain child nodes, corresponding to three kinds of Elements, as shown in the following table:

Widget

Corresponding Element

use

LeafRenderObjectWidget

LeafRenderObjectElement

The leaf nodes of the Widget tree are used for widgets that have no child nodes. Usually, the basic components belong to this category, such as Image.

SingleChildRenderObjectWidget

SingleChildRenderObjectElement

Contains a sub Widget, such as: ConstrainedBox, DecoratedBox, etc.

MultiChildRenderObjectWidget

MultiChildRenderObjectElement

Contains multiple child Widgets, generally have a children parameter and accept an array of Widgets. Such as Row, Column, Stack, etc.

> Note that many Widgets in Flutter are directly inherited from StatelessWidget or StatefulWidget, and then `build()`build a real RenderObjectWidget in the method, such as Text, which is actually inherited from StatelessWidget, and then `build()`use RichText to build its subtree in the method, and RichText is Inherited from MultiChildRenderObjectWidget. So in order to facilitate the description, we can also directly say that Text belongs to MultiChildRenderObjectWidget (other widgets can also be described in this way), this is the essence. After reading this, we will also find that, in fact, **StatelessWidget and StatefulWidget are two base classes for combining Widgets, and they are not associated with the final rendering object (RenderObjectWidget)** .

Layout components refer to `MultiChildRenderObjectWidget`Widgets that are directly or indirectly inherited (included) , and they generally have an `children`attribute for receiving child Widgets. Let's look at the inheritance relationship Widget> RenderObjectWidget> (Leaf/SingleChild/MultiChild)RenderObjectWidget.

`RenderObjectWidget`The creation and update `RenderObject`methods are defined in the class, and the subclass must implement them. `RenderObject`We only need to know that it is the final layout and rendering UI interface object. That is to say, for layout components, the layout algorithms are all It is realized by the corresponding `RenderObject`object, so if the reader is interested in the principle of a certain layout component introduced next, you can check its corresponding `RenderObject`implementation. For example `Stack`, the corresponding `RenderObject`object (stacked layout) is `RenderStack`, and the implementation of stacked layout is In `RenderStack`.

In this chapter, in order to give readers a quick understanding of the layout Widget, we will not go into `RenderObject`the details. When studying this chapter, the reader's focus is to master the layout characteristics of different layout components, specific principles and details, etc. After we get started with Flutter as a whole, we will study if we are interested.