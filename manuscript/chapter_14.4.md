# 14.4 Flutter operating mechanism-from startup to display

In this section, we mainly introduce the process of Flutter from startup to display.

### start up

The entrance of Flutter is in the `main()`function "lib/main.dart" , which is the starting point of the Dart application. In the Flutter application, `main()`the simplest implementation of the function is as follows:

```
void main() {
  runApp(MyApp());
}

```

It can be seen that the `main()`function only calls one `runApp()`method. Let's see what `runApp()`is done in the method:

```
void runApp(Widget app) {
  WidgetsFlutterBinding.ensureInitialized()
    ..attachRootWidget(app)
    ..scheduleWarmUpFrame();
}

```

The parameter `app`is a widget, which is the first Widget to be displayed after the Flutter application is started. And `WidgetsFlutterBinding`it is binding framework and Flutter engine widget bridge, defined as follows:

```
class WidgetsFlutterBinding extends BindingBase with GestureBinding, ServicesBinding, SchedulerBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding {
  static WidgetsBinding ensureInitialized() {
    if (WidgetsBinding.instance == null)
      WidgetsFlutterBinding();
    return WidgetsBinding.instance;
  }
}

```

You can see that it is `WidgetsFlutterBinding`inherited from `BindingBase`and mixed in a lot `Binding`. Before introducing these `Binding`, let's introduce it first `Window`. Here is `Window`the official explanation:

> The most basic interface to the host operating system's user interface.

Obviously, `Window`it is the interface of Flutter Framework to the host operating system. Let's take a look at `Window`the partial definition of the class:

```
class Window {

  // 当前设备的DPI，即一个逻辑像素显示多少物理像素，数字越大，显示效果就越精细保真。
  // DPI是设备屏幕的固件属性，如Nexus 6的屏幕DPI为3.5 
  double get devicePixelRatio => _devicePixelRatio;

  // Flutter UI绘制区域的大小
  Size get physicalSize => _physicalSize;

  // 当前系统默认的语言Locale
  Locale get locale;

  // 当前系统字体缩放比例。  
  double get textScaleFactor => _textScaleFactor;  

  // 当绘制区域大小改变回调
  VoidCallback get onMetricsChanged => _onMetricsChanged;  
  // Locale发生变化回调
  VoidCallback get onLocaleChanged => _onLocaleChanged;
  // 系统字体缩放变化回调
  VoidCallback get onTextScaleFactorChanged => _onTextScaleFactorChanged;
  // 绘制前回调，一般会受显示器的垂直同步信号VSync驱动，当屏幕刷新时就会被调用
  FrameCallback get onBeginFrame => _onBeginFrame;
  // 绘制回调  
  VoidCallback get onDrawFrame => _onDrawFrame;
  // 点击或指针事件回调
  PointerDataPacketCallback get onPointerDataPacket => _onPointerDataPacket;
  // 调度Frame，该方法执行后，onBeginFrame和onDrawFrame将紧接着会在合适时机被调用，
  // 此方法会直接调用Flutter engine的Window_scheduleFrame方法
  void scheduleFrame() native 'Window_scheduleFrame';
  // 更新应用在GPU上的渲染,此方法会直接调用Flutter engine的Window_render方法
  void render(Scene scene) native 'Window_render';

  // 发送平台消息
  void sendPlatformMessage(String name,
                           ByteData data,
                           PlatformMessageResponseCallback callback) ;
  // 平台通道消息处理回调  
  PlatformMessageCallback get onPlatformMessage => _onPlatformMessage;

  ... //其它属性及回调

}

```

You can see that the `Window`class contains some information about the current device and system, as well as some callbacks of the Flutter Engine. Now we come back to look at `WidgetsFlutterBinding`the various Bindings mixed in. By viewing the source code of these Bindings, we can find that these Bindings are basically listening and processing `Window`some events of the object, and then these events are packaged, abstracted and distributed according to the Framework model. It can be seen that `WidgetsFlutterBinding`it is the "glue" that connects the Flutter engine with the upper Framework.

-   `GestureBinding`: Provides a `window.onPointerDataPacket`callback, binds the Framework gesture subsystem, and is the binding entry for the Framework event model and the underlying events.
-   `ServicesBinding`: Provides a `window.onPlatformMessage`callback to bind the platform message channel (message channel), mainly dealing with native communication with Flutter.
-   `SchedulerBinding`: Provides `window.onBeginFrame`and `window.onDrawFrame`callbacks, monitors refresh events, and binds the Framework drawing scheduling subsystem.
-   `PaintingBinding`: Binding drawing library, mainly used to process image cache.
-   `SemanticsBinding`: The bridge between the semantic layer and the Flutter engine is mainly the underlying support for auxiliary functions.
-   `RendererBinding`: Provide `window.onMetricsChanged`, `window.onTextScaleFactorChanged`wait for callback. It is a bridge between the render tree and the Flutter engine.
-   `WidgetsBinding`: Provides `window.onLocaleChanged`, `onBuildScheduled`etc. callbacks. It is the bridge between the Flutter widget layer and the engine.

`WidgetsFlutterBinding.ensureInitialized()`Responsible for initializing a `WidgetsBinding`global singleton, `WidgetsBinding`the `attachRootWidget`method that will be called next , the method is responsible for adding the root Widget to `RenderView`it, the code is as follows:

```
void attachRootWidget(Widget rootWidget) {
  _renderViewElement = RenderObjectToWidgetAdapter<RenderBox>(
    container: renderView, 
    debugShortDescription: '[root]',
    child: rootWidget
  ).attachToRenderTree(buildOwner, renderViewElement);
}

```

Note that the code has `renderView`and `renderViewElement`two variables, `renderView`a `RenderObject`, which is the root of the tree rendered, and `renderViewElement`is `renderView`corresponding to `Element`the object, which is visible mainly completed widget root to the root `RenderObject`and then root `Element`entire association procedure. We look at `attachToRenderTree`the source code implementation:

```
RenderObjectToWidgetElement<T> attachToRenderTree(BuildOwner owner, [RenderObjectToWidgetElement<T> element]) {
  if (element == null) {
    owner.lockState(() {
      element = createElement();
      assert(element != null);
      element.assignOwner(owner);
    });
    owner.buildScope(element, () {
      element.mount(null, null);
    });
  } else {
    element._newWidget = this;
    element.markNeedsBuild();
  }
  return element;
}

```

This method is responsible for creating the root element, that is `RenderObjectToWidgetElement`, associating the element with the widget, that is, creating the element tree corresponding to the widget tree. If the element has already been created, set the associated widget in the root element as a new one, which shows that the element will only be created once and will be reused later. So `BuildOwner`what is it? In fact, it is the management class of the widget framework, which tracks which widgets need to be rebuilt.

### Rendering

Back `runApp`of implementation, when call finished `attachRootWidget`, the last line calls the `WidgetsFlutterBinding`instance `scheduleWarmUpFrame()`method, the method is implemented in `SchedulerBinding`, which will be called immediately after a draw (rather than waiting for "vsync" signal), the end of the draw Previously, this method will lock event distribution, which means that Flutter will not respond to various events before the completion of this drawing, which can ensure that no new redraws will be triggered during the drawing process. The following is `scheduleWarmUpFrame()`a partial implementation of the method (irrelevant code omitted):

```
void scheduleWarmUpFrame() {
  ...
  Timer.run(() {
    handleBeginFrame(null); 
  });
  Timer.run(() {
    handleDrawFrame();  
    resetEpoch();
  });
  // 锁定事件
  lockEvents(() async {
    await endOfFrame;
    Timeline.finishSync();
  });
 ...
}

```

You can see the main method calls `handleBeginFrame()`and `handleDrawFrame()`two methods, before looking at these two methods we first look at the concept of Frame and FrameCallback:

-   Frame: A drawing process, we call it a frame. The Flutter engine is continuously triggered by the display's vertical synchronization signal "VSync". We said that Flutter can achieve 60fps (Frame Per-Second), which means that 60 redraws can be triggered in one second. The larger the FPS value, the smoother the interface.
    
-   FrameCallback: `SchedulerBinding`There are three FrameCallback callback queues in the class. During a drawing process, these three callback queues will be executed at different times:
    
    1.  `transientCallbacks`: Used to store some temporary callbacks, and generally store animation callbacks. You can `SchedulerBinding.instance.scheduleFrameCallback`add callbacks via .
    2.  `persistentCallbacks`: Used to store some persistent callbacks. New drawing frames cannot be requested in such callbacks. Once registered, persistent callbacks cannot be removed. `SchedulerBinding.instance.addPersitentFrameCallback()`, This callback handles the layout and drawing work.
    3.  `postFrameCallbacks`: At the end of Frame will only be called once, the system will be removed after the call, can be `SchedulerBinding.instance.addPostFrameCallback()`registered, pay attention, do not trigger a new Frame in such a callback, which can cause a refresh cycle.

Now the reader to view themselves `handleBeginFrame()`and `handleDrawFrame()`the source of two methods can be found that the former is the implementation of the `transientCallbacks`queue, and the latter performed `persistentCallbacks`and `postFrameCallbacks`queues.

### draw

The rendering and drawing logic `RendererBinding`is implemented in. Check the source code and find the `initInstances()`following code in the method:

```
void initInstances() {
  ... //省略无关代码

  //监听Window对象的事件  
  ui.window
    ..onMetricsChanged = handleMetricsChanged
    ..onTextScaleFactorChanged = handleTextScaleFactorChanged
    ..onSemanticsEnabledChanged = _handleSemanticsEnabledChanged
    ..onSemanticsAction = _handleSemanticsAction;

  //添加PersistentFrameCallback    
  addPersistentFrameCallback(_handlePersistentFrameCallback);
}

```

We look at the last line and add a callback `addPersistentFrameCallback`to the `persistentCallbacks`queue via `_handlePersistentFrameCallback`:

```
void _handlePersistentFrameCallback(Duration timeStamp) {
  drawFrame();
}

```

Methods directly called by `RendererBinding`this `drawFrame()`method:

```
void drawFrame() {
  assert(renderView != null);
  pipelineOwner.flushLayout(); //布局
  pipelineOwner.flushCompositingBits(); //重绘之前的预处理操作，检查RenderObject是否需要重绘
  pipelineOwner.flushPaint(); // 重绘
  renderView.compositeFrame(); // 将需要绘制的比特数据发给GPU
  pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
}

```

Let's see what these methods do:

#### flushLayout()

```
void flushLayout() {
   ...
    while (_nodesNeedingLayout.isNotEmpty) {
      final List<RenderObject> dirtyNodes = _nodesNeedingLayout;
      _nodesNeedingLayout = <RenderObject>[];
      for (RenderObject node in 
           dirtyNodes..sort((RenderObject a, RenderObject b) => a.depth - b.depth)) {
        if (node._needsLayout && node.owner == this)
          node._layoutWithoutResize();
      }
    }
  } 
}

```

The source code is very simple. The main task of this method is to update all `RenderObject`the layout information marked as "dirty" . The main action occurs in the `node._layoutWithoutResize()`method, which will be called `performLayout()`for re-layout.

#### flushCompositingBits()

```
void flushCompositingBits() {
  _nodesNeedingCompositingBitsUpdate.sort(
      (RenderObject a, RenderObject b) => a.depth - b.depth
  );
  for (RenderObject node in _nodesNeedingCompositingBitsUpdate) {
    if (node._needsCompositingBitsUpdate && node.owner == this)
      node._updateCompositingBits(); //更新RenderObject.needsCompositing属性值
  }
  _nodesNeedingCompositingBitsUpdate.clear();
}

```

Check `RenderObject`whether it needs to be redrawn, and then update the `RenderObject.needsCompositing`attribute. If the attribute value is marked as, `true`it needs to be redrawn.

#### flushPaint()

```
void flushPaint() {
 ...
  try {
    final List<RenderObject> dirtyNodes = _nodesNeedingPaint; 
    _nodesNeedingPaint = <RenderObject>[];
    // 反向遍历需要重绘的RenderObject
    for (RenderObject node in 
         dirtyNodes..sort((RenderObject a, RenderObject b) => b.depth - a.depth)) {
      if (node._needsPaint && node.owner == this) {
        if (node._layer.attached) {
          // 真正的绘制逻辑  
          PaintingContext.repaintCompositedChild(node);
        } else {
          node._skippedPaintingOnLayer();
        }
      }
    }
  } 
}

```

This method performs the final drawing. It can be seen that it does not redraw everything `RenderObject`, but only redraws what needs to be redrawn `RenderObject`. The actual drawing is `PaintingContext.repaintCompositedChild()`drawn through, and this method will eventually call the Canvas API provided by the Flutter engine to complete the drawing.

#### compositeFrame ()

```
void compositeFrame() {
  ...
  try {
    final ui.SceneBuilder builder = ui.SceneBuilder();
    final ui.Scene scene = layer.buildScene(builder);
    if (automaticSystemUiAdjustment)
      _updateSystemChrome();
    ui.window.render(scene); //调用Flutter engine的渲染API
    scene.dispose(); 
  } finally {
    Timeline.finishSync();
  }
}

```

There is an `Scene`object in this method, and the Scene object is a data structure that saves the pixel information after the final rendering. This method passes the Canvas drawing `Scene`to the `window.render()`method, which will directly send the scene information to the Flutter engine, and finally the engine draws the image on the device screen.

#### At last

It should be noted that: Since it `RendererBinding`is just a mixin, and with it `WidgetsBinding`, we need to see `WidgetsBinding`whether the method is rewritten in the middle, and `WidgetsBinding`the `drawFrame()`source code of the method:

```
@override
void drawFrame() {
 ...//省略无关代码
  try {
    if (renderViewElement != null)
      buildOwner.buildScope(renderViewElement); 
    super.drawFrame(); //调用RendererBinding的drawFrame()方法
    buildOwner.finalizeTree();
  } 
}

```

We found that it `RendererBinding.drawFrame()`will be called before calling the method `buildOwner.buildScope()`(not the first drawing), and the method will perform the element marked as "dirty" `rebuild()`.

### to sum up

This section introduces the main flow of Flutter APP from startup to display on the screen. Readers can combine the introduction of Widget, Element and RenderObject in the previous chapters to strengthen their understanding of the details.