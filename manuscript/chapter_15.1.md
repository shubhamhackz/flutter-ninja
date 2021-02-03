# 15.1 Github client example

This chapter creates a new Flutter project to implement a simple Github client. The main goals of this example are two:

1.  Lead readers to understand how to use Flutter to develop a complete APP, understand the Flutter application development process and engineering structure, etc.
2.  An application and summary of the content learned in the previous chapters.

It should be noted that since Github itself has many functions, our focus is not to implement all the business functions of Github. Therefore, we only need to implement an APP skeleton to achieve the above two points. The following functions we want to achieve are as follows:

1.  Realize Github account login and logout functions
2.  You can view your project homepage after logging in
3.  Support skinning
4.  Support multiple languages
5.  The login status can be persistent;

To achieve the above functions will involve the following technical points:

1.  Network request; need to request Github API.
2.  Json to Dart Model class;
3.  Global status management; language, theme, login status, etc. need to be shared globally.
4.  Persistent storage; save login information, user information, etc.
5.  Support internationalization, use of Intl package

Now that the goal has been determined, in the following chapters, we will implement the above functions step by step in modules.