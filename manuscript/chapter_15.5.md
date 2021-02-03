# 15.5 Network request encapsulation

In this section, we will encapsulate the network request interface used in the APP based on the dio network library introduced earlier, and apply a simple caching strategy at the same time. Let's first introduce the principle of network interface caching, and then encapsulate the service request interface of the APP.

## 15.5.1 Network interface cache

Due to the slow speed of accessing Github servers in China, we apply some simple caching strategies: use the requested URL as the key, cache the return value of the request in a specified time period, and set a maximum cache number, when the maximum is exceeded Remove the oldest one after the number of caches. But it is also necessary to provide a mechanism for deciding whether to enable caching for a specific interface or request. This mechanism can specify which interfaces or that request does not apply caching. This mechanism is necessary. For example, the login interface should not be cached. Another example is that users should no longer apply the cache when they pull down to refresh. Before implementing the cache, we first define the `CacheObject`class that stores the cache information :

```
class CacheObject {
  CacheObject(this.response)
      : timeStamp = DateTime.now().millisecondsSinceEpoch;
  Response response;
  int timeStamp; // 缓存创建时间

  @override
  bool operator ==(other) {
    return response.hashCode == other.hashCode;
  }

  //将请求uri作为缓存的key
  @override
  int get hashCode => response.realUri.hashCode;
}

```

Next we need to implement a specific caching strategy. Since we are using the dio package, we can implement the caching strategy directly through the interceptor:

```
import 'dart:collection';
import 'package:dio/dio.dart';
import '../index.dart';

class CacheObject {
  CacheObject(this.response)
      : timeStamp = DateTime.now().millisecondsSinceEpoch;
  Response response;
  int timeStamp;

  @override
  bool operator ==(other) {
    return response.hashCode == other.hashCode;
  }

  @override
  int get hashCode => response.realUri.hashCode;
}

class NetCache extends Interceptor {
  // 为确保迭代器顺序和对象插入时间一致顺序一致，我们使用LinkedHashMap
  var cache = LinkedHashMap<String, CacheObject>();

  @override
  onRequest(RequestOptions options) async {
    if (!Global.profile.cache.enable) return options;
    // refresh标记是否是"下拉刷新"
    bool refresh = options.extra["refresh"] == true;
    //如果是下拉刷新，先删除相关缓存
    if (refresh) {
      if (options.extra["list"] == true) {
        //若是列表，则只要url中包含当前path的缓存全部删除（简单实现，并不精准）
        cache.removeWhere((key, v) => key.contains(options.path));
      } else {
        // 如果不是列表，则只删除uri相同的缓存
        delete(options.uri.toString());
      }
      return options;
    }
    if (options.extra["noCache"] != true &&
        options.method.toLowerCase() == 'get') {
      String key = options.extra["cacheKey"] ?? options.uri.toString();
      var ob = cache[key];
      if (ob != null) {
        //若缓存未过期，则返回缓存内容
        if ((DateTime.now().millisecondsSinceEpoch - ob.timeStamp) / 1000 <
            Global.profile.cache.maxAge) {
          return cache[key].response;
        } else {
          //若已过期则删除缓存，继续向服务器请求
          cache.remove(key);
        }
      }
    }
  }

  @override
  onError(DioError err) async {
    // 错误状态不缓存
  }

  @override
  onResponse(Response response) async {
    // 如果启用缓存，将返回结果保存到缓存
    if (Global.profile.cache.enable) {
      _saveCache(response);
    }
  }

  _saveCache(Response object) {
    RequestOptions options = object.request;
    if (options.extra["noCache"] != true &&
        options.method.toLowerCase() == "get") {
      // 如果缓存数量超过最大数量限制，则先移除最早的一条记录
      if (cache.length == Global.profile.cache.maxCount) {
        cache.remove(cache[cache.keys.first]);
      }
      String key = options.extra["cacheKey"] ?? options.uri.toString();
      cache[key] = CacheObject(object);
    }
  }

  void delete(String key) {
    cache.remove(key);
  }
}

```

The explanation of the code is in the comments. What needs to be explained here is that the dio package `option.extra`is specifically used to extend the request parameters. We have defined two parameters "refresh" and "noCache" to achieve "for a specific interface or request Decide whether to enable caching mechanism", the meaning of these two parameters are as follows:

| Parameter name | Types of | Explanation                                                                                      |
| -------------- | -------- | ------------------------------------------------------------------------------------------------ |
| refresh        | bool     | If true, the cache is not used for this request, but the new request result will still be cached |
| noCache        | bool     | Caching is disabled for this request, and the request result will not be cached.                 |

## 15.5.2 Encapsulating network requests

A complete APP may involve many network requests. In order to facilitate management and converge the request entry, the best practice in engineering is to put all network requests in the same source file. Since our interfaces are all APIs provided by the requested Github development platform, we define a Git class specifically for Github API interface calls. In addition, in the debugging process, we usually need some tools to view network requests and response messages. Using network proxy tools to debug network data problems is the mainstream way. To configure the proxy, you need to specify the address and port of the proxy server in the application. In addition, the Github API is HTTPS protocol, so after configuring the proxy, you should also disable certificate verification. These configurations are performed when the Git class is initialized ( `init()方法`). Here is the source code of the Git class:

```
import 'dart:async';
import 'dart:convert';
import 'dart:io';
import 'package:dio/dio.dart';
import 'package:dio/adapter.dart';
import 'package:flutter/material.dart';
import '../index.dart';

class Git {
  // 在网络请求过程中可能会需要使用当前的context信息，比如在请求失败时
  // 打开一个新路由，而打开新路由需要context信息。
  Git([this.context]) {
    _options = Options(extra: {"context": context});
  }

  BuildContext context;
  Options _options;
  static Dio dio = new Dio(BaseOptions(
    baseUrl: 'https://api.github.com/',
    headers: {
      HttpHeaders.acceptHeader: "application/vnd.github.squirrel-girl-preview,"
          "application/vnd.github.symmetra-preview+json",
    },
  ));

  static void init() {
    // 添加缓存插件
    dio.interceptors.add(Global.netCache);
    // 设置用户token（可能为null，代表未登录）
    dio.options.headers[HttpHeaders.authorizationHeader] = Global.profile.token;

    // 在调试模式下需要抓包调试，所以我们使用代理，并禁用HTTPS证书校验
    if (!Global.isRelease) {
      (dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate =
          (client) {
        client.findProxy = (uri) {
          return "PROXY 10.1.10.250:8888";
        };
        //代理工具会提供一个抓包的自签名证书，会通不过证书校验，所以我们禁用证书校验
        client.badCertificateCallback =
            (X509Certificate cert, String host, int port) => true;
      };
    }
  }

  // 登录接口，登录成功后返回用户信息
  Future<User> login(String login, String pwd) async {
    String basic = 'Basic ' + base64.encode(utf8.encode('$login:$pwd'));
    var r = await dio.get(
      "/users/$login",
      options: _options.merge(headers: {
        HttpHeaders.authorizationHeader: basic
      }, extra: {
        "noCache": true, //本接口禁用缓存
      }),
    );
    //登录成功后更新公共头（authorization），此后的所有请求都会带上用户身份信息
    dio.options.headers[HttpHeaders.authorizationHeader] = basic;
    //清空所有缓存
    Global.netCache.cache.clear();
    //更新profile中的token信息
    Global.profile.token = basic;
    return User.fromJson(r.data);
  }

  //获取用户项目列表
  Future<List<Repo>> getRepos(
      {Map<String, dynamic> queryParameters, //query参数，用于接收分页信息
      refresh = false}) async {
    if (refresh) {
      // 列表下拉刷新，需要删除缓存（拦截器中会读取这些信息）
      _options.extra.addAll({"refresh": true, "list": true});
    }
    var r = await dio.get<List>(
      "user/repos",
      queryParameters: queryParameters,
      options: _options,
    );
    return r.data.map((e) => Repo.fromJson(e)).toList();
  }
}

```

You can see that in our `init()`method, we judged whether it is a debugging environment, and then did some network configuration for the debugging environment (set proxy and disable certificate verification). The `Git.init()`method is called when the application starts (the `Global.init()`method will be called `Git.init()`).

In addition, it should be noted that all our network requests are `dio`sent through the same instance (static variable). When creating the `dio`instance, we set the base address of the Github API and the header supported by the API globally, so that all the requests sent through the `dio`instance The request will default to some configuration of the user.

In this example, we only use the login interface and the interface to obtain user projects, so `Git`only the `login(…)`and `getRepos(…)`methods are defined in the class . If the reader wants to expand the functions on the basis of this example, the reader can add other interface request methods to `Git`In the class, this realizes the centralized management and maintenance of the network request interface at the code level.