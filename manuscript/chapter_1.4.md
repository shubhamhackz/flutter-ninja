# 1.4 Introduction to Dart language

We have already introduced the relevant features of the Dart language before. Readers can take a look. If you are already familiar with Dart syntax, you can skip this section. If you don’t know Dart yet, don’t worry. According to the author’s experience, if you have Experience in other programming languages ​​(especially Java and JavaScript) will be very easy to get started with Dart. Of course, if you are an iOS developer, don’t worry. Dart also has some features that are similar to Swift, such as named parameters. When I was learning Dart, it only took an hour to read the Language Tour on Dart’s official website. Start writing Flutter.

In my opinion, Dart's design goal should be to draw on both Java and JavaScript. Dart is very similar to Java in terms of static syntax, such as type definitions, function declarations, generics, etc., and very similar to JavaScript in terms of dynamic features, such as functional features, asynchronous support, etc. In addition to integrating the strengths of Java and JavaScript languages, Dart also has some other expressive syntax, such as optional named parameters, `..`(cascade operator) and `?.`(conditional member access operator) and `??`(empty assignment operator) ). In fact, readers who know more about programming languages ​​will find that Dart actually sees not only the shadows of Java and JavaScript, but also other programming languages. For example, named parameters have long been used in Objective-C and Swift. It is very common, and `??`operators already exist in the PHP 7.0 syntax, so we can see that Google has high hopes for the Dart language and wants to build Dart into a programming language with the best of hundreds of families.

Next, we first give a brief introduction to Dart syntax, and then make a brief comparison between Dart and JavaScript and Java to facilitate readers' better understanding.

> Note: Since this book is not a book that specifically introduces the Dart language, this chapter will mainly introduce the grammatical features commonly used in Flutter development. If you want to learn more about Dart, readers can go to Dart's official website to learn. There are already many Dart related materials on the Internet. Up. In addition, Dart 2.0 has been officially released, so all examples in this book use Dart 2.0 syntax.

## 1.4.1 Variable declaration

1.  **var**
    
    Similar to JavaScript `var`, it can receive any type of variable, but the biggest difference is that once the var variable in Dart is assigned, the type will be determined, and you cannot change its type, such as:
    
    ```
    var t;
    t = "hi world";
    // 下面代码在dart中会报错，因为变量t的类型已经确定为String，
    // 类型一旦确定后则不能再更改其类型。
    t = 1000;
    
    ```
    
    The above code is no problem in JavaScript. Front-end developers need to pay attention. The reason for this difference is that Dart itself is a strongly typed language. Any variable has a certain type. In Dart, when you `var`declare a variable Later, Dart will infer its type based on the type of the first assignment data when compiling, and its type has been determined after the completion of the compilation, and JavaScript is a purely weakly typed scripting language, and var is just a way of declaring variables.
    
2.  **dynamic** and **Object**
    
    `Object`Dart is the foundation class for all objects, meaning that all types are `Object`subclasses (including Function and Null), so any type of data can be assigned to `Object`objects declared. `dynamic`And `var`are the same as keywords can be assigned to the variable declaration Any object. And `dynamic`the `Object`same is that the variables they declare can change the assignment type later.
    
    ```
     dynamic t;
     Object x;
     t = "hi world";
     x = 'Hello Object';
     //下面代码没有问题
     t = 1000;
     x = 1000;
    
    ```
    
    `dynamic`The `Object`difference is that the `dynamic`declared object compiler will provide all possible combinations, and the `Object`declared object can only use the properties and methods of Object, otherwise the compiler will report an error. Such as:
    
    ```
     dynamic a;
     Object b;
     main() {
         a = "";
         b = "";
         printLengths();
     }   
    
     printLengths() {
         // no warning
         print(a.length);
         // warning:
         // The getter 'length' is not defined for the class 'Object'
         print(b.length);
     }
    
    ```
    
    The variable a will not report an error, the variable b compiler will report an error
    
    `dynamic`This feature is very similar `Objective-C`to the `id`role `dynamic`of .This feature makes us need to pay special attention when using it, which can easily introduce a runtime error.
    
3.  **final** and **const**
    
    If you never intend to change a variable, then use `final`or `const`not `var`, nor is it a type. A `final`variable can be set only once, that the difference between the two: `const`the variable is a compile-time constant, `final`variable used when first initialized. For variables that are modified `final`or `const`modified, the variable type can be omitted, such as:
    
    ```
    //可以省略String这个类型声明
    final str = "hi world";
    //final String str = "hi world"; 
    const str1 = "hi world";
    //const String str1 = "hi world";
    
    ```
    

## 1.4.2 Functions

Dart is a true object-oriented language, so even functions are objects and have a type **Function** . This means that functions can be assigned to variables or passed as parameters to other functions, which is a typical feature of functional programming.

1.  Function declaration
    
    ```
    bool isNoble(int atomicNumber) {
      return _nobleGases[atomicNumber] != null;
    }
    
    ```
    
    If the Dart function declaration does not explicitly declare the return value type, it will be `dynamic`handled by default . Note that the function return value does not have type inference:
    
    ```
    typedef bool CALLBACK();
    
    //不指定返回类型，此时默认为dynamic，不是bool
    isNoble(int atomicNumber) {
      return _nobleGases[atomicNumber] != null;
    }
    
    void test(CALLBACK cb){
       print(cb()); 
    }
    //报错，isNoble不是bool类型
    test(isNoble);
    
    ```
    
2.  For functions that contain only one expression, you can use shorthand syntax
    
    ```
    bool isNoble (int atomicNumber)=> _nobleGases [ atomicNumber ] ！= null ;
    
    ```
    
3.  Function as a variable
    
    ```
    var say = (str){
      print(str);
    };
    say("hi world");
    
    ```
    
4.  Function passed as a parameter
    
    ```
    void execute(var callback) {
        callback();
    }
    execute(() => print("xxx"))
    
    ```
    
5.  Optional positional parameters
    
    Wrap a group of function parameters, mark them as optional positional parameters with [], and put them at the end of the parameter list:
    
    ```
    String say(String from, String msg, [String device]) {
      var result = '$from says $msg';
      if (device != null) {
        result = '$result with a $device';
      }
      return result;
    }
    
    ```
    
    The following is an example of calling this function without optional parameters:
    
    ```
    say('Bob', 'Howdy'); //结果是： Bob says Howdy
    
    ```
    
    The following is an example of calling this function with the third parameter:
    
    ```
    say('Bob', 'Howdy', 'smoke signal'); //结果是：Bob says Howdy with a smoke signal
    
    ```
    
6.  Optional named parameters
    
    When defining a function, use {param1, param2, …} at the end of the parameter list to specify named parameters. E.g:
    
    ```
    //设置[bold]和[hidden]标志
    void enableFlags({bool bold, bool hidden}) {
        // ... 
    }
    
    ```
    
    When calling a function, you can use designated named parameters. E.g:`paramName: value`
    
    ```
    enableFlags(bold: true, hidden: false);
    
    ```
    
    Optional named parameters are used a lot in Flutter.
    
    **Note that you cannot use optional positional parameters and optional named parameters at the same time**
    

## 1.4.3 Asynchronous support

The Dart library has a lot of return `Future`or `Stream`object functions. These functions are called **asynchronous functions** : they will only return after setting up some time-consuming operations, such as IO operations. Instead of waiting for this operation to complete.

`async`And `await`keywords support asynchronous programming, allowing you to write asynchronous code that is very similar to synchronous code.

### Future

`Future`It is `Promise`very similar to JavaScript , which represents the final completion (or failure) of an asynchronous operation and its result value. Simply put, it is used to process asynchronous operations. If the asynchronous process succeeds, the successful operation is executed, and if the asynchronous process fails, the error is captured or the subsequent operation is stopped. A Future will only correspond to one result, either success or failure.

Due to its many functions, here we only introduce its commonly used APIs and features. Also, please remember that `Future`the return value of all APIs is still an `Future`object, so chain calls can be made easily.

#### Future.then

To facilitate the example, in this example we `Future.delayed`created a delayed task (the actual scenario will be a real time-consuming task, such as a network request), that is, the result string "hi world!" is returned after 2 seconds, and then we in `then`receiving the result of the asynchronous and print the results, as follows:

```
Future.delayed(new Duration(seconds: 2),(){
   return "hi world!";
}).then((data){
   print(data);
});

```

#### Future.catchError

If an error occurs in the asynchronous task, we can `catchError`catch the error in, we change the above example to:

```
Future.delayed(new Duration(seconds: 2),(){
   //return "hi world!";
   throw AssertionError("Error");  
}).then((data){
   //执行成功会走到这里  
   print("success");
}).catchError((e){
   //执行失败会走到这里  
   print(e);
});

```

In this example, we throw an exception in asynchronous tasks, the `then`callback function will not be executed and replaced by `catchError`the callback function will be called; however, not only `catchError`callback to catch the error, `then`the method as well as an optional Parameters `onError`, we can also use it to catch exceptions:

```
Future.delayed(new Duration(seconds: 2), () {
    //return "hi world!";
    throw AssertionError("Error");
}).then((data) {
    print("success");
}, onError: (e) {
    print(e);
});

```

#### Future.whenComplete

Sometimes, we will encounter scenarios where something needs to be done regardless of the success or failure of the asynchronous task execution, such as popping up a loading dialog box before the network request, and closing the dialog box after the request is over. In this scenario, there are two methods. The first is to close the dialog box in `then`or respectively `catch`. The second is to use `Future`the `whenComplete`callback. Let's change the above example:

```
Future.delayed(new Duration(seconds: 2),(){
   //return "hi world!";
   throw AssertionError("Error");
}).then((data){
   //执行成功会走到这里 
   print(data);
}).catchError((e){
   //执行失败会走到这里   
   print(e);
}).whenComplete((){
   //无论成功或失败都会走到这里
});

```

#### Future.wait

Sometimes, we need to wait for multiple asynchronous tasks to complete before performing some operations. For example, if we have an interface, we need to obtain data from two network interfaces. After the acquisition is successful, we need to perform specific operations on the two interface data. What should I do if it is displayed on the UI interface after processing? The answer is `Future.wait`that it accepts an `Future`array of parameters. The success callback `Future`will be triggered only after all of the arrays are executed successfully `then`. As long as one `Future`execution fails, the error callback will be triggered. Below, we use simulation `Future.delayed`to simulate two asynchronous tasks for data acquisition. When the two asynchronous tasks are executed successfully, the results of the two asynchronous tasks are spliced ​​and printed out. The code is as follows:

```
Future.wait([
  // 2秒后返回结果  
  Future.delayed(new Duration(seconds: 2), () {
    return "hello";
  }),
  // 4秒后返回结果  
  Future.delayed(new Duration(seconds: 4), () {
    return " world";
  })
]).then((results){
  print(results[0]+results[1]);
}).catchError((e){
  print(e);
});

```

After executing the above code, you will see "hello world" in the console after 4 seconds.

### Async/await

`async/await`The `async/await`functions and usage of Dart and JavaScript are exactly the same. If you already know `async/await`the usage of JavaScript , you can skip this section directly.

#### Callback Hell

If there is a lot of asynchronous logic in the code, and a large number of asynchronous tasks depend on the results of other asynchronous tasks, there will inevitably be a `Future.then`set of callbacks in the callback. For example, for example, there is a demand scenario where the user first logs in, and after the login is successful, the user ID will be obtained, and then the user ID will be used to request the user's personal information. After obtaining the user's personal information, we need to use it for convenience. Cached in the local file system, the code is as follows:

```
//先分别定义各个异步任务
Future<String> login(String userName, String pwd){
    ...
    //用户登录
};
Future<String> getUserInfo(String id){
    ...
    //获取用户信息 
};
Future saveUserInfo(String userInfo){
    ...
    // 保存用户信息 
};

```

Next, execute the entire task flow:

```
login("alice","******").then((id){
 //登录成功后通过，id获取用户信息    
 getUserInfo(id).then((userInfo){
    //获取用户信息后保存 
    saveUserInfo(userInfo).then((){
       //保存用户信息，接下来执行其它操作
        ...
    });
  });
})

```

You can feel that if there are a large number of asynchronous dependencies in the business logic, the above situation will occur in the callback inside the callback. Too much nesting will cause the code readability to decrease and the error rate to increase, and it is very difficult Maintenance, this problem is vividly called **Callback Hell (Callback Hell)** . The problem of callback hell was very prominent in JavaScript before, and it was also the point where JavaScript was the most complained about. However, with the release of ECMAScript6 and ECMAScript7 standards, this problem has been solved very well, and the two artifacts to solve the callback hell are the introduction of ECMAScript6 `Promise`, And introduced in ECMAScript7 `async/await`. In Dart, the two in JavaScript are almost completely translated: `Future`equivalent `Promise`, without `async/await`even changing the name. Next, let's look at the adoption `Future`and `async/await`how to eliminate the nesting problem in the above example.

##### Use Future to eliminate Callback Hell

```
login("alice","******").then((id){
      return getUserInfo(id);
}).then((userInfo){
    return saveUserInfo(userInfo);
}).then((e){
   //执行接下来的操作 
}).catchError((e){
  //错误处理  
  print(e);
});

```

As mentioned above, _" `Future`The return value of all APIs is still an `Future`object, so it can be easily chained."_ If there is one returned `Future`in then, it `future`will be executed, and the following will be triggered after execution `then`Callbacks, in this order downwards, avoid layers of nesting.

##### Use async/await to eliminate callback hell

By `Future`callback back `Future`the way though to avoid nested layers, but still with a layer of a callback, is there a way to allow us to write synchronization code can be like manner as to perform asynchronous tasks without using the callback? The answer is yes, it is necessary to use `async/await`, let's look at the code directly, and then explain, the code is as follows:

```
task() async {
   try{
    String id = await login("alice","******");
    String userInfo = await getUserInfo(id);
    await saveUserInfo(userInfo);
    //执行接下来的操作   
   } catch(e){
    //错误处理   
    print(e);   
   }  
}

```

-   `async`Used to indicate that the function is asynchronous, the defined function returns an `Future`object, you can use the then method to add a callback function.
-   `await`Followed by a `Future`showing wait for the asynchronous task is completed, the asynchronous completion will go down; `await`must appear `async`within the function.

As you can see, we have `async/await`represented an asynchronous stream with synchronous code.

> In fact, whether in JavaScript or Dart, it `async/await`is just a syntactic sugar, and the compiler or interpreter will eventually convert it into a Promise (Future) call chain.

## 1.4.4 Stream

`Stream`It is also used to receive asynchronous event data, and the `Future`difference is that it can receive the results (success or failure) of multiple asynchronous operations. In other words, when performing asynchronous tasks, you can pass result data or error exceptions by triggering multiple success or failure events. `Stream`It is often used in asynchronous task scenarios where data is read multiple times, such as network content downloading, file reading and writing, etc. for example:

```
Stream.fromFutures([
  // 1秒后返回结果
  Future.delayed(new Duration(seconds: 1), () {
    return "hello 1";
  }),
  // 抛出一个异常
  Future.delayed(new Duration(seconds: 2),(){
    throw AssertionError("Error");
  }),
  // 3秒后返回结果
  Future.delayed(new Duration(seconds: 3), () {
    return "hello 3";
  })
]).listen((data){
   print(data);
}, onError: (e){
   print(e.message);
},onDone: (){

});

```

The above code will output in turn:

```
I/flutter (17666): hello 1
I/flutter (17666): Error
I/flutter (17666): hello 3

```

The code is very simple, so I won't repeat it.

> Question: Since Stream can receive multiple events, can Stream be used to implement a subscriber-mode event bus?

## 1.4.5 Comparison of Dart, Java and JavaScript

Through the above introduction, I believe you should have a preliminary impression of Dart. Since the author also uses Java and JavaScript at ordinary times, I will talk about my views based on my own experience and combining Java and JavaScript.

> The reason why Dart is compared with Java and JavaScript is that they are typical representatives of strongly typed languages ​​and weakly typed languages ​​respectively, and many places in Dart's grammar also draw on Java and JavaScript.

### Dart vs Java

Objectively speaking, Dart is indeed more expressive than Java at the grammatical level; at the VM level, Dart VM has been repeatedly optimized for memory recovery and throughput. However, for specific performance comparisons, the author did not find relevant test data, but In my opinion, as long as the Dart language is popular, there is no need to worry about the performance of the VM. After all, Google already has many technologies on Go (VM is not used but GC), JavaScript (v8), and Dalvik (Java VM on Android). Accumulation. It is worth noting that Dart can already achieve GC within 10ms in Flutter, so comparing Dart and Java, the decisive factor will not be performance. At the grammatical level, Dart is more expressive than Java. The most important thing is that Dart's support for functional programming is much stronger than Java (currently only staying at Lambda expressions). The real shortcoming of Dart is **ecology** , but the author I believe that with the gradual popularity of Flutter, it will go back and push the accelerated development of the Dart ecosystem. For Dart, it takes time now.

### Dart vs JavaScript

JavaScript's weak types have been caught short, so TypeScript, CoffeeScript and even Facebook's flow (although not a superset of JavaScript, but also provide static type checking through annotation and packaging tools) have a market. Among the scripting languages I have used (I have used Python and PHP), JavaScript is undoubtedly the scripting language with the best **dynamic** support. For example, in JavaScript, any object can be dynamically extended at any time. For those who are proficient in JavaScript For masters, this is undoubtedly a sharp sword. However, everything has two sides. The powerful dynamic feature of JavaScript is also a double-edged sword. You can often hear another voice, thinking that the dynamic nature of JavaScript is terrible. Too flexible makes the code difficult to predict. , Cannot restrict undesired modifications. After all, some people are always worried about the code written by themselves or others. They want to make the code controllable and expect a static type checking system to help them reduce errors. For this reason, in Flutter, Dart almost gave up the dynamic features of the scripting language, such as not supporting reflection or dynamically creating functions. And Dart forced to open the type check (Strong Mode) in 2.0, the original checked mode (checked mode) and optional type (optional type) will fade out, so in terms of type safety, Dart and TypeScript, CoffeeScript are similar , So from this point of view alone, Dart does not have any obvious advantages, but on the whole, Dart can perform server-side scripting, APP development, and web development, which has advantages!

In summary, the author is still very optimistic about the future of the Dart language. The reason for expressing this attitude is because in the early stage of the development of new technologies, many people may still be swayed and hesitate, so it is necessary to give everyone a boost. Needle, of course, this is a matter of opinion, everyone can express their own opinions.