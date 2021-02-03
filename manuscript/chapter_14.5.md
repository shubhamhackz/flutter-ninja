# 14.5 Image loading principle and caching

In the previous chapters of this book, I have introduced `Image`components, and mentioned that the Flutter framework has a cache (memory) for loaded pictures. The default maximum cache number is 1000, and the maximum cache space is 100M. This section will introduce the principle of Image and the image caching mechanism in detail `ImageProvider`. Let's take a look at the class first .

## 14.5.1 ImageProvider

We already know `Image`that the `image`parameter of the component is a required parameter, it is a `ImageProvider`type. Let's introduce it in detail below `ImageProvider`. It `ImageProvider`is an abstract class that defines related interfaces for image data acquisition and loading. It has two main responsibilities:

1.  Provide image data source
2.  Cache image

We look at `ImageProvider`the detailed definition of the abstract class:

```
abstract class ImageProvider<T> {

  ImageStream resolve(ImageConfiguration configuration) {
    // 实现代码省略
  }
  Future<bool> evict({ ImageCache cache,
                      ImageConfiguration configuration = ImageConfiguration.empty }) async {
    // 实现代码省略
  }

  Future<T> obtainKey(ImageConfiguration configuration); 
  @protected
  ImageStreamCompleter load(T key); // 需子类实现
}

```

#### `load(T key)`method

The interface for loading image data sources. Different data sources have different loading methods, and each `ImageProvider`subclass must implement it. For example, `NetworkImage`classes and `AssetImage`classes, they are all `ImageProvider`subclasses, but they need to load image data from different data sources: image data `NetworkImage`is loaded from the network, but loaded `AssetImage`from the final application package (loading hits application installation Resource pictures in the package). Let's take `NetworkImage`an example and look at the implementation of its load method:

```

@override
ImageStreamCompleter load(image_provider.NetworkImage key) {

  final StreamController<ImageChunkEvent> chunkEvents = StreamController<ImageChunkEvent>();

  return MultiFrameImageStreamCompleter(
    codec: _loadAsync(key, chunkEvents), //调用
    chunkEvents: chunkEvents.stream,
    scale: key.scale,
    ... //省略无关代码
  );
}

```

We see that `load`the return value type of the method is `ImageStreamCompleter`that it is an abstract class that defines some interfaces for managing the image loading process `Image`. It is through it that Widget monitors the image loading status (we will introduce the `Image`principle in detail below ).

`MultiFrameImageStreamCompleter`Is `ImageStreamCompleter`a subclass is flutter sdk preset class, by class, we have a convenient and easily create an `ImageStreamCompleter`instance to do for the `load`method's return value.

We can see that `MultiFrameImageStreamCompleter`a `codec`parameter is required , and the parameter type is . It is a handler for the image encoding and decoding class. In fact, it is just a wrapper class of the flutter engine API, which means that the image encoding and decoding logic is not implemented in the Dart code part, but in the flutter engine. The class part is defined as follows:`Future<ui.Codec>``Codec``Codec`

```
@pragma('vm:entry-point')
class Codec extends NativeFieldWrapperClass2 {
  // 此类由flutter engine创建，不应该手动实例化此类或直接继承此类。
  @pragma('vm:entry-point')
  Codec._();

  /// 图片中的帧数(动态图会有多帧)
  int get frameCount native 'Codec_frameCount';

  /// 动画重复的次数
  /// * 0 表示只执行一次
  /// * -1 表示循环执行
  int get repetitionCount native 'Codec_repetitionCount';

  /// 获取下一个动画帧
  Future<FrameInfo> getNextFrame() {
    return _futurize(_getNextFrame);
  }

  String _getNextFrame(_Callback<FrameInfo> callback) native 'Codec_getNextFrame';

```

We can see that the `Codec`final result is one or more (motion picture) frames, and these frames will eventually be drawn on the screen.

`MultiFrameImageStreamCompleter 的`  `codec`The parameter value `_loadAsync`is the return value of the method, we continue to look at `_loadAsync`the implementation of the method:

```

 Future<ui.Codec> _loadAsync(
    NetworkImage key,
    StreamController<ImageChunkEvent> chunkEvents,
  ) async {
    try {
      //下载图片
      final Uri resolved = Uri.base.resolve(key.url);
      final HttpClientRequest request = await _httpClient.getUrl(resolved);
      headers?.forEach((String name, String value) {
        request.headers.add(name, value);
      });
      final HttpClientResponse response = await request.close();
      if (response.statusCode != HttpStatus.ok)
        throw Exception(...);
      // 接收图片数据 
      final Uint8List bytes = await consolidateHttpClientResponseBytes(
        response,
        onBytesReceived: (int cumulative, int total) {
          chunkEvents.add(ImageChunkEvent(
            cumulativeBytesLoaded: cumulative,
            expectedTotalBytes: total,
          ));
        },
      );
      if (bytes.lengthInBytes == 0)
        throw Exception('NetworkImage is an empty file: $resolved');
      // 对图片数据进行解码
      return PaintingBinding.instance.instantiateImageCodec(bytes);
    } finally {
      chunkEvents.close();
    }
  }

```

You can see that the `_loadAsync`method mainly does two things:

1.  Download the picture.
2.  Decode the downloaded picture data.

The download logic is relatively simple: by `HttpClient`downloading the image from the Internet, in addition, the download request will set some custom headers, which the developer can pass through `NetworkImage`the `headers`named parameters.

After the picture is downloaded, it is called `PaintingBinding.instance.instantiateImageCodec(bytes)`to decode the picture. It is worth noting that it is `instantiateImageCodec(...)`also a Native API package, which actually calls the Flutter engine `instantiateImageCodec`method. The source code is as follows:

```
String _instantiateImageCodec(Uint8List list, _Callback<Codec> callback, _ImageInfo imageInfo, int targetWidth, int targetHeight)
  native 'instantiateImageCodec';

```

#### `obtainKey(ImageConfiguration)`method

This interface is mainly to cooperate with the realization of image caching. `ImageProvider`After loading the data from the data source, the `ImageCache`image data will be cached globally . The image data cache is a Map, and the key of the Map is the return value of calling this method. The key represents different image data caches.

#### `resolve(ImageConfiguration)` method

`resolve`The method is `ImageProvider`the `Image`main entry method that is exposed to . It accepts a `ImageConfiguration`parameter and returns `ImageStream`, which is the image data stream. We focus on the `resolve`execution process:

```
ImageStream resolve(ImageConfiguration configuration) {
  ... //省略无关代码
  final ImageStream stream = ImageStream();
  T obtainedKey; //
  //定义错误处理函数
  Future<void> handleError(dynamic exception, StackTrace stack) async {
    ... //省略无关代码
    stream.setCompleter(imageCompleter);
    imageCompleter.setError(...);
  }

  // 创建一个新Zone，主要是为了当发生错误时不会干扰MainZone
  final Zone dangerZone = Zone.current.fork(...);

  dangerZone.runGuarded(() {
    Future<T> key;
    // 先验证是否已经有缓存
    try {
      // 生成缓存key，后面会根据此key来检测是否有缓存
      key = obtainKey(configuration);
    } catch (error, stackTrace) {
      handleError(error, stackTrace);
      return;
    }
    key.then<void>((T key) {
      obtainedKey = key;
      // 缓存的处理逻辑在这里，记为A，下面详细介绍
      final ImageStreamCompleter completer = PaintingBinding.instance
          .imageCache.putIfAbsent(key, () => load(key), onError: handleError);
      if (completer != null) {
        stream.setCompleter(completer);
      }
    }).catchError(handleError);
  });
  return stream;
}

```

`ImageConfiguration`Contains relevant information about the picture and the device, such as the size of the picture, where it is located `AssetBundle`(only pictures that have been loaded into the installation package exist), and the current device platform, devicePixelRatio (device pixel ratio, etc.). The Flutter SDK provides a convenient function `createLocalImageConfiguration`to create `ImageConfiguration`objects:

```
ImageConfiguration createLocalImageConfiguration(BuildContext context, { Size size }) {
  return ImageConfiguration(
    bundle: DefaultAssetBundle.of(context),
    devicePixelRatio: MediaQuery.of(context, nullOk: true)?.devicePixelRatio ?? 1.0,
    locale: Localizations.localeOf(context, nullOk: true),
    textDirection: Directionality.of(context),
    size: size,
    platform: defaultTargetPlatform,
  );
}

```

We can find that this information is basically obtained through `Context`.

A process at the above code is the primary code cache, here `PaintingBinding.instance.imageCache`is `ImageCache`an example, which is `PaintingBinding`a property, and Flutter frame `PaintingBinding.instance`is a single embodiment, `imageCache`indeed, a single embodiment, that is global image cache, Unified `PaintingBinding.instance.imageCache`management.

Let's look at the `ImageCache`class definition below :

```
const int _kDefaultSize = 1000;
const int _kDefaultSizeBytes = 100 << 20; // 100 MiB

class ImageCache {
  // 正在加载中的图片队列
  final Map<Object, _PendingImage> _pendingImages = <Object, _PendingImage>{};
  // 缓存队列
  final Map<Object, _CachedImage> _cache = <Object, _CachedImage>{};

  // 缓存数量上限(1000)
  int _maximumSize = _kDefaultSize;
  // 缓存容量上限 (100 MB)
  int _maximumSizeBytes = _kDefaultSizeBytes;

  // 缓存上限设置的setter
  set maximumSize(int value) {...}
  set maximumSizeBytes(int value) {...}

  ... // 省略部分定义

  // 清除所有缓存
  void clear() {
    // ...省略具体实现代码
  }

  // 清除指定key对应的图片缓存
  bool evict(Object key) {
   // ...省略具体实现代码
  }


  ImageStreamCompleter putIfAbsent(Object key, ImageStreamCompleter loader(), { ImageErrorListener onError }) {
    assert(key != null);
    assert(loader != null);
    ImageStreamCompleter result = _pendingImages[key]?.completer;
    // 图片还未加载成功，直接返回
    if (result != null)
      return result;

    // 有缓存，继续往下走
    // 先移除缓存，后再添加，可以让最新使用过的缓存在_map中的位置更近一些，清理时会LRU来清除
    final _CachedImage image = _cache.remove(key);
    if (image != null) {
      _cache[key] = image;
      return image.completer;
    }
    try {
      result = loader();
    } catch (error, stackTrace) {
      if (onError != null) {
        onError(error, stackTrace);
        return null;
      } else {
        rethrow;
      }
    }
    void listener(ImageInfo info, bool syncCall) {
      final int imageSize = info?.image == null ? 0 : info.image.height * info.image.width * 4;
      final _CachedImage image = _CachedImage(result, imageSize);
      // 下面是缓存处理的逻辑
      if (maximumSizeBytes > 0 && imageSize > maximumSizeBytes) {
        _maximumSizeBytes = imageSize + 1000;
      }
      _currentSizeBytes += imageSize;
      final _PendingImage pendingImage = _pendingImages.remove(key);
      if (pendingImage != null) {
        pendingImage.removeListener();
      }

      _cache[key] = image;
      _checkCacheSize();
    }
    if (maximumSize > 0 && maximumSizeBytes > 0) {
      final ImageStreamListener streamListener = ImageStreamListener(listener);
      _pendingImages[key] = _PendingImage(result, streamListener);
      // Listener is removed in [_PendingImage.removeListener].
      result.addListener(streamListener);
    }
    return result;
  }

  // 当缓存数量超过最大值或缓存的大小超过最大缓存容量，会调用此方法清理到缓存上限以内
  void _checkCacheSize() {
   while (_currentSizeBytes > _maximumSizeBytes || _cache.length > _maximumSize) {
      final Object key = _cache.keys.first;
      final _CachedImage image = _cache[key];
      _currentSizeBytes -= image.sizeBytes;
      _cache.remove(key);
    }
    ... //省略无关代码
  }
}

```

If there is a cache, use the cache, if there is no cache, call the load method to load the picture. After the load is successful:

1.  First determine whether the image data is cached, and if there is, return directly `ImageStream`.
2.  If there is no cache, call the `load(T key)`method to load the image data from the data source, cache first after loading successfully, and then return to ImageStream.

In addition, we can see `ImageCache`that there is a setter for setting the upper limit of the cache in the class, so if we can customize the upper limit of the cache:

```
 PaintingBinding.instance.imageCache.maximumSize=2000; //最多2000张
 PaintingBinding.instance.imageCache.maximumSizeBytes = 200 << 20; //最大200M

```

Now we look at the cached key, because the value of the same key in the Map will be overwritten, that is to say, the key is a unique identifier of the image cache, as long as it is a different key, the image data will be cached separately (even if it is actually the same image). So what is the unique identification of the picture? Tracing the source code, it is easy to find that the key is `ImageProvider.obtainKey()`the return value of the method, and this method needs `ImageProvider`to be rewritten by subclasses, which means that different `ImageProvider`key definition logic will be different. In fact, it is also very easy to understand. For example `NetworkImage`, it is appropriate to use the url of the picture as the key, and for `AssetImage`the "package name + path", the only key should be used. Let's take `NetworkImage`an example and take a look at its `obtainKey()`implementation:

```
@override
Future<NetworkImage> obtainKey(image_provider.ImageConfiguration configuration) {
  return SynchronousFuture<NetworkImage>(this);
}

```

The code is very simple, it creates a synchronized future, and then directly returns itself as a key. Because the `NetworkImage`"==" operator is used in Map to determine whether the key (the object at this time ) is equal, the logic for defining the key is `NetworkImage`the "==" operator:

```
@override
bool operator ==(dynamic other) {
  ... //省略无关代码
  final NetworkImage typedOther = other;
  return url == typedOther.url
      && scale == typedOther.scale;
}

```

It is clear that for web pictures, the "url+zoom ratio" is used as the cache key. In other words, **if the url or scale of the two pictures is different, they will be re-downloaded and cached separately** .

In addition, we need to note that the image cache is in the memory, and there is no local file persistent storage, which is why the network image needs to be downloaded again after the application restarts.

It also means that during the application life cycle, if the cache does not exceed the upper limit, the same image will only be downloaded once.

### to sum up

The above mainly combines the source code to explore the `ImageProvider`main functions and principles. If you want to summarize the `ImageProvider`functions in one sentence , it should be: load image data, cache and decode it. I would like to remind readers once again that the source code of Flutter is very good first-hand information. Readers are advised to explore more. In addition, you must have a summary while reading the source code to learn, so as not to get lost in the source code.

## 14.5.2 Image component principle

`Image`The basic usage that we introduced in the previous chapters , now we go deeper and study how `Image`to `ImageProvider`cooperate to obtain the final decoded data, and then how to draw the picture to the screen.

To change the way of thinking in this section, let's not look at `Image`the source code directly , but implement a simplified version of the " `Image`component" based on the knowledge we have mastered `MyImage`. The code is roughly as follows:

```
class MyImage extends StatefulWidget {
  const MyImage({
    Key key,
    @required this.imageProvider,
  })
      : assert(imageProvider != null),
        super(key: key);

  final ImageProvider imageProvider;

  @override
  _MyImageState createState() => _MyImageState();
}

class _MyImageState extends State<MyImage> {
  ImageStream _imageStream;
  ImageInfo _imageInfo;

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // 依赖改变时，图片的配置信息可能会发生改变
    _getImage();
  }

  @override
  void didUpdateWidget(MyImage oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.imageProvider != oldWidget.imageProvider)
      _getImage();
  }

  void _getImage() {
    final ImageStream oldImageStream = _imageStream;
    // 调用imageProvider.resolve方法，获得ImageStream。
    _imageStream =
        widget.imageProvider.resolve(createLocalImageConfiguration(context));
    //判断新旧ImageStream是否相同，如果不同，则需要调整流的监听器
    if (_imageStream.key != oldImageStream?.key) {
      final ImageStreamListener listener = ImageStreamListener(_updateImage);
      oldImageStream?.removeListener(listener);
      _imageStream.addListener(listener);
    }
  }

  void _updateImage(ImageInfo imageInfo, bool synchronousCall) {
    setState(() {
      // Trigger a build whenever the image changes.
      _imageInfo = imageInfo;
    });
  }

  @override
  void dispose() {
    _imageStream.removeListener(ImageStreamListener(_updateImage));
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return RawImage(
      image: _imageInfo?.image, // this is a dart:ui Image object
      scale: _imageInfo?.scale ?? 1.0,
    );
  }
}

```

The above code flow is as follows:

1.  `imageProvider.resolve`You can get one `ImageStream`(picture data stream) through the method , and then monitor `ImageStream`the changes. When the image data source changes, the `ImageStream`corresponding event will be triggered. In this example, we only set the listener for the success of the image `_updateImage`, and `_updateImage`only update it `_imageInfo`. It is worth noting that if it is a static image, `ImageStream`only one time will be triggered. If it is a dynamic image, multiple events will be triggered, and each time there will be a decoded image frame.
2.  `_imageInfo`It will be rebuild after the update, and a `RawImage`Widget will be created at this time . `RawImage`Eventually it will pass `RenderImage`to draw the picture on the screen. If you continue to follow up `RenderImage`the class, we will find `RenderImage`a `paint`method called `paintImage`method, and the `paintImage`method by `Canvas`the `drawImageRect(…)`, `drawImageNine(...)`and other methods to complete the final draw.
3.  The final drawn by `RawImage`to complete.

Test it below `MyImage`:

```
class ImageInternalTestRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        MyImage(
          imageProvider: NetworkImage(
            "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4",
          ),
        )
      ],
    );
  }
}

```

The running effect is shown in Figure 14-4:

![Figure 14-4](https://pcdn.flutterchina.club/imgs/14-4.png)

Succeeded! Now, presumably `Image`the source code of Widget is no longer necessary to spend a chapter to introduce it, and readers who are interested can read it by themselves.

## to sum up

This section mainly introduces the loading, caching and drawing process of Flutter images. Which `ImageProvider`is mainly responsible for loading and caching picture data, drawn mainly by the part of the logic `RawImage`is completed to. And `Image`it is connected from `ImageProvider`and `RawImage`bridges.