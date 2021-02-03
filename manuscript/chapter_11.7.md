# 11.7 Json to Dart Model class

In actual combat, the backend interface often returns some structured data, such as JSON, XML, etc., as in the previous example when we requested the Github API, the data it returns is a string in JSON format. In order to facilitate us to manipulate JSON in the code, we First convert the JSON format string into a Dart object. This can be achieved by `dart:convert`the built-in JSON decoder json.decode(). This method can convert the JSON string to a List or Map according to the specific content of the JSON string, so that we can Use them to find the required value, such as:

```
//一个JSON格式的用户列表字符串
String jsonStr='[{"name":"Jack"},{"name":"Rose"}]';
//将JSON字符串转为Dart对象(此处是List)
List items=json.decode(jsonStr);
//输出第一个用户的姓名
print(items[0]["name"]);

```

The method of converting JSON string to List/Map through json.decode() is relatively simple. It has no external dependencies or other settings, which is very convenient for small projects. But when the project becomes larger, this kind of manual serialization logic may become unmanageable and error-prone. For example, the following JSON:

```
{
  "name": "John Smith",
  "email": "john@example.com"
}

```

We can `json.decode`decode JSON by calling the method, using the JSON string as a parameter:

```
Map<String, dynamic> user = json.decode(json);

print('Howdy, ${user['name']}!');
print('We sent the verification link to ${user['email']}.');

```

Since `json.decode()`only one is returned , this means that we don't know the type of the value until runtime. In this way, we lose most of the statically typed language features: type safety, auto-completion, and most importantly, compile-time exceptions. As a result, our code may become very error-prone. For example, when we visited the or field, we entered very quickly, which caused the field name to be mistyped. But because this JSON is in the map structure, the compiler does not know the field name of this error, so it will not report an error during compilation.`Map<String,  dynamic>``name``email`

In fact, this problem will be encountered on many platforms, and there has been a good solution for a long time, namely "Json Modeling". The specific method is to predefine some Model classes corresponding to the Json structure, and then request the data Then dynamically create instances of the Model class based on the data. In this way, in the development stage, we use instances of the Model class instead of Map/List, so that spelling errors will not occur when accessing internal properties. For example, we can solve the aforementioned problem by introducing a simple Model class, which we call it `User`. Inside the User class, we have:

-   A `User.fromJson`constructor for constructing a map from an `User`example map Structure
-   A `toJson`method of the `User`instance is converted to a map.

In this way, the calling code can now have type safety, auto-complete fields (name and email), and compile-time exceptions. If we treat the misspelled field as a `int`type instead of `String`, then our code will not compile instead of crashing at runtime.

**user.dart**

```
class User {
  final String name;
  final String email;

  User(this.name, this.email);

  User.fromJson(Map<String, dynamic> json)
      : name = json['name'],
        email = json['email'];

  Map<String, dynamic> toJson() =>
    <String, dynamic>{
      'name': name,
      'email': email,
    };
}

```

Now, the serialization logic is moved inside the model itself. Using this new method, we can deserialize users very easily.

```
Map userMap = json.decode(json);
var user = new User.fromJson(userMap);

print('Howdy, ${user.name}!');
print('We sent the verification link to ${user.email}.');

```

To serialize a user, we just `User`pass the object to the `json.encode`method. We don't need to manually call `toJson`this method, because `JSON.encode will automatically call it internally.

```
String json = json.encode(user);

```

In this way, the calling code does not have to worry about JSON serialization, but the Model class is still necessary. In practice, both `User.fromJson`and `User.toJson`methods require unit testing in place to verify correct behavior.

In addition, in actual scenarios, JSON objects are rarely that simple, and nested JSON objects are not uncommon. If there is something that can automatically handle JSON serialization for us, it will be very good. Fortunately, there is!

### Automatically generate Model

Although there are other libraries available, in this book, we introduce the officially recommended [json_serializable package](https://pub.dartlang.org/packages/json_serializable) . It is an automated source code generator that can generate JSON serialization templates for us during the development phase. In this way, since the serialization code is no longer handwritten and maintained by us, we reduce the risk of JSON serialization exceptions during runtime. To the lowest.

### Set json_serializable in the project

To be included `json_serializable`in our project, we need a regular and two **development dependencies** . In short, a **development dependency** is a **dependency** that is not included in the source code of our application. It is some auxiliary tools and scripts in the development process, similar to the development dependency in node.

**pubspec.yaml**

```
dependencies:
  # Your other regular dependencies here
  json_annotation: ^2.0.0

dev_dependencies:
  # Your other dev_dependencies here
  build_runner: ^1.0.0
  json_serializable: ^2.0.0

```

Run in your project root folder `flutter packages get`(or click "Packages Get" in the editor) to use these new dependencies in the project.

### Create model class in json_serializable way

Let's see how to convert our `User`class into one `json_serializable`. For simplicity, we use the simplified JSON model in the previous example.

**user.dart**

```
import 'package:json_annotation/json_annotation.dart';

// user.g.dart 将在我们运行生成命令后自动生成
part 'user.g.dart';

///这个标注是告诉生成器，这个类是需要生成Model类的
@JsonSerializable()

class User{
  User(this.name, this.email);

  String name;
  String email;
  //不同的类使用不同的mixin即可
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);  
}

```

With the above settings, the source code generator will generate JSON code for serialization `name`and `email`fields.

If needed, it is easy to customize the naming strategy. For example, if the API we are using returns an object with _snake_case_ , but we want to use _lowerCamelCase_ in our model , then we can use the @JsonKey annotation:

```
//显式关联JSON字段名与Model属性的对应关系 
@JsonKey(name: 'registration_date_millis')
final int registrationDateMillis;

```

### Run the code generator

`json_serializable`When you first create the class, you will see an error similar to Figure 11-4.

![ide_warning](https://pcdn.flutterchina.club/imgs/11-4.png)

These errors are completely normal, because the generated code for the Model class does not yet exist. In order to solve this problem, we must run the code generator to generate serialization templates for us. There are two ways to run the code generator:

#### One-time generation

By running in our project root directory:

```
flutter packages pub run build_runner build

```

This triggers a one-time build. We can generate json serialization code for our Model when needed. It uses our source files to find out the source files (including @JsonSerializable annotations) that need to generate the Model class to generate the corresponding. g.dart file. A good suggestion is to put all Model classes in a separate directory, and then execute commands in that directory.

Although this is very convenient, it would be better if we don't need to manually run the build command every time we make changes in the Model class.

#### Continuous generation

Using _watcher_ can make our source code generation process more convenient. It will monitor the changes of the files in our project and automatically build the necessary files when needed. We can `flutter packages pub run build_runner watch`start the _watcher_ by running it in the project root directory . You only need to start the observer once, and then it will run in the background, which is safe.

### Automatically generate templates

One of the biggest problems with the above method is to write a template for each json, which is relatively boring. If there is a tool that can directly generate templates based on JSON text, then we can completely free our hands. The author has implemented a script with dart, which can automatically generate a template and directly convert JSON into a Model class. Let’s see how to do it:

1.  Define a "template template" named "template.dart":
    
    ```
    import 'package:json_annotation/json_annotation.dart';
    %t
    part '%s.g.dart';
    @JsonSerializable()
    class %s {
        %s();
    
        %s
        factory %s.fromJson(Map<String,dynamic> json) => _$%sFromJson(json);
        Map<String, dynamic> toJson() => _$%sToJson(this);
    }
    
    ```
    
    The "%t" and "%s" in the template are placeholders, which will be dynamically replaced with appropriate import headers and class names when the script is running.
    
2.  Write a script (mo.dart) that automatically generates templates. It can traverse and generate templates according to the specified JSON directory. We define some rules when generating:
    
    -   If the JSON file name starts with an underscore "_", the JSON file is ignored.
    -   Complex JSON objects tend to be nested, and we can manually specify the nested objects through a special flag (example later).
    
    The script is written by Dart, and the source code is as follows:
    
    ```
    import 'dart:convert';
    import 'dart:io';
    import 'package:path/path.dart' as path;
    const TAG="\$";
    const SRC="./json"; //JSON 目录
    const DIST="lib/models/"; //输出model目录
    
    void walk() { //遍历JSON目录生成模板
      var src = new Directory(SRC);
      var list = src.listSync();
      var template=new File("./template.dart").readAsStringSync();
      File file;
      list.forEach((f) {
        if (FileSystemEntity.isFileSync(f.path)) {
          file = new File(f.path);
          var paths=path.basename(f.path).split(".");
          String name=paths.first;
          if(paths.last.toLowerCase()!="json"||name.startsWith("_")) return ;
          if(name.startsWith("_")) return;
          //下面生成模板
          var map = json.decode(file.readAsStringSync());
          //为了避免重复导入相同的包，我们用Set来保存生成的import语句。
          var set= new Set<String>();
          StringBuffer attrs= new StringBuffer();
          (map as Map<String, dynamic>).forEach((key, v) {
              if(key.startsWith("_")) return ;
              attrs.write(getType(v,set,name));
              attrs.write(" ");
              attrs.write(key);
              attrs.writeln(";");
              attrs.write("    ");
          });
          String  className=name[0].toUpperCase()+name.substring(1);
          var dist=format(template,[name,className,className,attrs.toString(),
                                    className,className,className]);
          var _import=set.join(";\r\n");
          _import+=_import.isEmpty?"":";";
          dist=dist.replaceFirst("%t",_import );
          //将生成的模板输出
          new File("$DIST$name.dart").writeAsStringSync(dist);
        }
      });
    }
    
    String changeFirstChar(String str, [bool upper=true] ){
      return (upper?str[0].toUpperCase():str[0].toLowerCase())+str.substring(1);
    }
    
    //将JSON类型转为对应的dart类型
     String getType(v,Set<String> set,String current){
      current=current.toLowerCase();
      if(v is bool){
        return "bool";
      }else if(v is num){
        return "num";
      }else if(v is Map){
        return "Map<String,dynamic>";
      }else if(v is List){
        return "List";
      }else if(v is String){ //处理特殊标志
        if(v.startsWith("$TAG[]")){
          var className=changeFirstChar(v.substring(3),false);
          if(className.toLowerCase()!=current) {
            set.add('import "$className.dart"');
          }
          return "List<${changeFirstChar(className)}>";
    
        }else if(v.startsWith(TAG)){
          var fileName=changeFirstChar(v.substring(1),false);
          if(fileName.toLowerCase()!=current) {
            set.add('import "$fileName.dart"');
          }
          return changeFirstChar(fileName);
        }
        return "String";
      }else{
        return "String";
      }
     }
    
    //替换模板占位符
    String format(String fmt, List<Object> params) {
      int matchIndex = 0;
      String replace(Match m) {
        if (matchIndex < params.length) {
          switch (m[0]) {
            case "%s":
              return params[matchIndex++].toString();
          }
        } else {
          throw new Exception("Missing parameter for string format");
        }
        throw new Exception("Invalid format string: " + m[0].toString());
      }
      return fmt.replaceAllMapped("%s", replace);
    }
    
    void main(){
      walk();
    }
    
    ```
    
3.  Write a shell (mo.sh) to link the generated template and the generated model:
    
    ```
    dart mo.dart
    flutter packages pub run build_runner build --delete-conflicting-outputs
    
    ```
    

So far, our script is written. We create a json directory in the root directory, then move user.json into it, and then create a models directory in the lib directory to save the final generated Model class. Now we only need one command to generate the Model class:

```
./mo.sh

```

After running, everything will be executed automatically, now it's much better, isn't it?

#### Nested JSON

We define a person.json content to be modified as:

```
{
  "name": "John Smith",
  "email": "john@example.com",
  "mother":{
    "name": "Alice",
    "email":"alice@example.com"
  },
  "friends":[
    {
      "name": "Jack",
      "email":"Jack@example.com"
    },
    {
      "name": "Nancy",
      "email":"Nancy@example.com"
    }
  ]
}

```

Each Person has `name`, `email`, `mother`and `friends`four fields, as `mother`is a Person, a friend is more Person (array), so we expect to generate the Model is the following:

```
import 'package:json_annotation/json_annotation.dart';
part 'person.g.dart';

@JsonSerializable()
class Person {
    Person();

    String name;
    String email;
    Person mother;
    List<Person> friends;

    factory Person.fromJson(Map<String,dynamic> json) => _$PersonFromJson(json);
    Map<String, dynamic> toJson() => _$PersonToJson(this);
}

```

At this time, we only need to simply modify the JSON, add some special flags, and re-run mo.sh:

```
{
  "name": "John Smith",
  "email": "john@example.com",
  "mother":"$person",
  "friends":"$[]person"
}

```

We use the dollar sign "$" as the special identifier (if it conflicts with the content, you can modify the `TAG`constants in mo.dart , custom identifiers), the script will first convert the corresponding field to the corresponding object after encountering the special identifier Or an array of objects. The array of objects needs to add the array symbol "[]" after the identifier, and the symbol is followed by the specific type name, in this case it is person. Other types Similarly, we added to the User to add a Person type of `boss`field:

```
{
  "name": "John Smith",
  "email": "john@example.com",
  "boss":"$person"
}

```

Re-run mo.sh, the generated user.dart is as follows:

```
import 'package:json_annotation/json_annotation.dart';
import "person.dart";
part 'user.g.dart';

@JsonSerializable()

class User {
    User();

    String name;
    String email;
    Person boss;

    factory User.fromJson(Map<String,dynamic> json) => _$UserFromJson(json);
    Map<String, dynamic> toJson() => _$UserToJson(this);
}

```

As you can see, the `boss`field has been automatically added and "person.dart" has been automatically imported.

### Json_model package

It is obviously very troublesome to build a script like the above for each project. For this reason, we have encapsulated the above script and generated template into a package, which has been published on Pub. The package name is [Json_model](https://github.com/flutterchina/json_model) . The developer adds the package to the development dependency. Then, you can use a command to generate Dart classes based on the Json file. In addition, [Json_model is](https://github.com/flutterchina/json_model) in iterative process and its functions will gradually improve, so readers are recommended to use this package directly (instead of manually copying the above code).

### Use IDE plugin to generate model

At present, Android Studio (or IntelliJ) has several plug-ins that can convert json files into Models, but the quality of the plug-ins is uneven, and some are even infected with plagiarism. Therefore, the author does not make a priority recommendation here. Readers are interested. You can understand by yourself. However, we still have to understand the pros and cons of IDE plug-ins and [Json_model](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fflutterchina%2Fjson_model) :

1.  [Json_model](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fflutterchina%2Fjson_model) needs to maintain a separate folder for storing Json files. If there is a change, only need to modify the Json file to regenerate the Model class; and IDE plug-ins generally require the user to manually copy the Json content into an input box, so that after the Json is generated The files are not archived, and later changes need to be done manually.
2.  [Json_model](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fflutterchina%2Fjson_model) can manually specify other Model classes referenced by a certain field, which can avoid generating duplicate classes; and IDE plug-ins generally generate a Model class for all nested objects in each Json file, even if these nested objects may be in other It has been generated in the Model class.
3.  [Json_model](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fflutterchina%2Fjson_model) provides a command line conversion method, which can be easily integrated into non-UI environment scenarios such as CI.

### FAQ

Many people may ask if there is a Json serialization library like Gson/Jackson in Java development in Flutter? The answer is no! Because such libraries need to use runtime reflection, this is disabled in Flutter. Runtime reflection will interfere with Dart's _tree shaking_ . With _tree shaking_ , unused code can be "removed" in the release version, which can significantly optimize the size of the application. Since reflection is applied to all codes by default, _tree shaking_ will be difficult to work, because it is difficult to know which code is not used when reflection is enabled, so redundant code is difficult to strip off, so Dart's reflection function is disabled in Flutter, and Because of this, the function of dynamically transforming the Model cannot be realized.