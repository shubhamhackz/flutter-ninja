# 15.4 Global variables and shared state

The application usually contains some variable information throughout the life cycle of the APP. This information may be used in most parts of the APP, such as current user information and local information. In Flutter, we divide the information that needs to be shared globally into two categories: global variables and shared state. Global variables simply refer to variables that run through the entire APP life cycle and are used to simply save some information or encapsulate some global tools and methods. Shared state refers to the information that needs to be shared across components or routes. This information is usually also global variables. The difference between shared state and global variables is that when the former changes, all components using the state need to be notified, while the latter does not need . To this end, we separate global variables and shared state management.

## 15.4.1 Global Variables-Global Class

We create a `Global`class in the "lib/common" directory , which mainly manages the global variables of APP, defined as follows:

```
// 提供五套可选主题色
const _themes = <MaterialColor>[
  Colors.blue,
  Colors.cyan,
  Colors.teal,
  Colors.green,
  Colors.red,
];

class Global {
  static SharedPreferences _prefs;
  static Profile profile = Profile();
  // 网络缓存对象
  static NetCache netCache = NetCache();

  // 可选的主题列表
  static List<MaterialColor> get themes => _themes;

  // 是否为release版
  static bool get isRelease => bool.fromEnvironment("dart.vm.product");

  //初始化全局信息，会在APP启动时执行
  static Future init() async {
    _prefs = await SharedPreferences.getInstance();
    var _profile = _prefs.getString("profile");
    if (_profile != null) {
      try {
        profile = Profile.fromJson(jsonDecode(_profile));
      } catch (e) {
        print(e);
      }
    }

    // 如果没有缓存策略，设置默认缓存策略
    profile.cache = profile.cache ?? CacheConfig()
      ..enable = true
      ..maxAge = 3600
      ..maxCount = 100;

    //初始化网络请求相关配置
    Git.init();
  }

  // 持久化Profile信息
  static saveProfile() =>
      _prefs.setString("profile", jsonEncode(profile.toJson()));
}

```

The meaning of each field of the Global class is commented, so I won't repeat it here. It should be noted that it `init()`needs to be executed when the App starts, so the application `main`method is as follows:

```
void main() => Global.init().then((e) => runApp(MyApp()));

```

Here, we must ensure that the `Global.init()`method does not throw an exception, or `runApp(MyApp())`simply unreachable.

## 15.4.2 Shared state

With global variables, we also need to consider how to share state across components. Of course, if we replace all the state we want to share with global variables, it is also possible, but this is not a good idea in Flutter development, because the state of the component is related to the UI, and we expect to rely on the state when the state changes. UI components are automatically updated. If global variables are used, then we have to manually handle state change notifications, receiving mechanisms, and variable and component dependencies. Therefore, in this example, we use the Provider package introduced earlier to achieve cross-component state sharing, so we need to define related Providers. In this example, the states that need to be shared include login user information, APP theme information, and APP language information. Since this information is changed, other Widgets that rely on the information must be notified immediately to update, so we should use it `ChangeNotifierProvider`. In addition, after this information is changed, the Profile information needs to be updated and persisted. In summary, we can define a `ProfileChangeNotifier`base class, and then let the Model that needs to be shared inherit from this class, which is `ProfileChangeNotifier`defined as follows:

```
class ProfileChangeNotifier extends ChangeNotifier {
  Profile get _profile => Global.profile;

  @override
  void notifyListeners() {
    Global.saveProfile(); //保存Profile变更
    super.notifyListeners(); //通知依赖的Widget更新
  }
}

```

### user status

The user status is updated when the login status changes, and its dependencies are notified. We define it as follows:

```
class UserModel extends ProfileChangeNotifier {
  User get user => _profile.user;

  // APP是否登录(如果有用户信息，则证明登录过)
  bool get isLogin => user != null;

  //用户信息发生变化，更新用户信息并通知依赖它的子孙Widgets更新
  set user(User user) {
    if (user?.login != _profile.user?.login) {
      _profile.lastLogin = _profile.user?.login;
      _profile.user = user;
      notifyListeners();
    }
  }
}

```

### APP theme status

The theme status is updated when the user changes the APP theme, and its dependencies are notified, defined as follows:

```
class ThemeModel extends ProfileChangeNotifier {
  // 获取当前主题，如果为设置主题，则默认使用蓝色主题
  ColorSwatch get theme => Global.themes
      .firstWhere((e) => e.value == _profile.theme, orElse: () => Colors.blue);

  // 主题改变后，通知其依赖项，新主题会立即生效
  set theme(ColorSwatch color) {
    if (color != theme) {
      _profile.theme = color[500].value;
      notifyListeners();
    }
  }
}

```

### APP language status

When the APP language is selected as the follow system (Auto), the APP language will be updated when the communication language changes; when the user selects a specific language in the APP (US English or Simplified Chinese), the APP will always use the user The selected language will no longer change with the system language. The language status class is defined as follows:

```
class LocaleModel extends ProfileChangeNotifier {
  // 获取当前用户的APP语言配置Locale类，如果为null，则语言跟随系统语言
  Locale getLocale() {
    if (_profile.locale == null) return null;
    var t = _profile.locale.split("_");
    return Locale(t[0], t[1]);
  }

  // 获取当前Locale的字符串表示
  String get locale => _profile.locale;

  // 用户改变APP语言后，通知依赖项更新，新语言会立即生效
  set locale(String locale) {
    if (locale != _profile.locale) {
      _profile.locale = locale;
      notifyListeners();
    }
  }
}

```