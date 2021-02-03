# 10.1 Introduction to the custom component method

When the existing components provided by Flutter cannot meet our needs, or we need to encapsulate some common components in order to share code, then we need custom components. There are three ways to customize components in Flutter: by combining other components, self-drawing, and implementing RenderObject. In this section we first introduce the characteristics of these three methods respectively, and the details of them will be introduced in detail in the following chapters.

### Combine other widgets

In this way, other components are assembled to form a new component. For example, we introduced earlier `Container`is a combination of components, it is composed of `DecoratedBox`, `ConstrainedBox`, `Transform`, `Padding`, `Align`and other components.

In Flutter, the idea of ​​combination is very important. Flutter provides a lot of basic components, and our interface development is actually combining these components as needed to achieve various layouts.

### Self-painted

If we can’t achieve the required UI through existing components, we can do it by self-drawing components. For example, we need a circular progress bar with a gradual color gradient, and Flutter `CircularProgressIndicator`does not support displaying precise progress. When applying a gradient color to the progress bar (its `valueColor`property only supports changing the color of the Indicator when performing a rotation animation), the best way at this time is to draw the desired appearance through custom components. We can implement UI self-drawing through the `CustomPaint`sum provided in Flutter `Canvas`.

### Implement RenderObject

UI component itself has the appearance of a Flutter offered, such as text `Text`, `Image`are through the corresponding `RenderObject`(we will "Flutter core principles" described in detail in the chapter `RenderObject`) rendered, such as Text is `RenderParagraph`rendered; and `Image`a is `RenderImage`rendered. `RenderObject`Is an abstract class, it defines an abstract method `paint(...)`:

```
void paint(PaintingContext context, Offset offset)

```

`PaintingContext`The drawing context representing the component `PaintingContext.canvas`can be obtained through `Canvas`, and the drawing logic is mainly `Canvas`implemented through API. Subclasses need to rewrite this method to implement their own drawing logic. If they `RenderParagraph`need to implement text drawing logic, they `RenderImage`need to implement picture drawing logic.

It can be found that `RenderObject`in the end, it is also `Canvas`drawn through API, so what is the difference between the way of implementation and `RenderObject`the way of passing `CustomPaint`and `Canvas`self-drawing described above ? In fact, the answer is very simple. It `CustomPaint`is just a proxy class encapsulated for the convenience of developers. It directly inherits from it `SingleChildRenderObjectWidget`. `RenderCustomPaint`The `paint`method used is to connect `Canvas`with the brush `Painter`(which needs to be implemented by the developer, introduced in the following chapter) to achieve the final drawing (drawing logic is `Painter`in) .

### to sum up

"Combination" is the easiest way to customize components. In any scenario where custom components are required, we should give priority to whether it can be achieved through combination. Self-drawing and implementation `RenderObject`methods are essentially the same. Developers need to call the `Canvas`API to draw the UI manually. The advantage is that it is powerful and flexible. In theory, it can realize any appearance of the UI. The disadvantage is that you must understand the `Canvas`details of the API and have to do it yourself. To implement the drawing logic.

In the next section of this chapter, we will introduce the process of customizing the UI in detail through some examples. Since the latter two methods are essentially the same, and many basic components in Flutter are implemented through `RenderObject`the form, we will follow up Only the method of `CustomPaint`sum is introduced `Canvas`. If readers `RenderObject`are curious about the custom method, they can check `RenderObject`the implementation source code corresponding to the relevant basic components in Flutter , such as `RenderParagraph`or `RenderImage`.