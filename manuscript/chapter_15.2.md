# 15.2 Flutter APP code structure

Let's first create a brand new Flutter project, named "github_client_app"; the steps to create a new project depend on the editor used by the reader, and are relatively simple, so I won't repeat it here. After creation, the project structure is as follows:

```
github_client_app
├── android
├── ios
├── lib
└── test

```

Since we need to use external images and Icon resources, we create folders "imgs" and "fonts" in the root directory of the project. The former is used to save images and the latter is used to save Icon files. Regarding pictures and icons, readers can refer to the corresponding content in Chapter 3.

Because in network data transmission and persistence, we need to transfer and save data through Json; but we need to convert Json into Dart Model class during application development, now we use "Json to Model" in Chapter 11 Therefore, we need to create another "jsons" folder in the root directory for saving Json files.

Multi-language support. We use the solution introduced in Chapter 13 "Internationalization", so we also need to create a "l10n" folder in the root directory to save the arb files corresponding to various languages.

Now the project directory becomes:

```
github_client_app
├── android
├── fonts
├── l10n-arb
├── imgs
├── ios
├── jsons
├── lib
└── test

```

Since our Dart codes are all under the "lib" folder, the author created the following directories under the lib file based on technical selection and experience:

```
lib
├── common
├── l10n
├── models
├── states
├── routes
└── widgets

```

| Folder  | Effect                                                                                                                        |
| ------- | ----------------------------------------------------------------------------------------------------------------------------- |
| common  | Some tool classes, such as general method classes, network interface classes, static classes that save global variables, etc. |
| l10n    | Internationalization related classes are in this directory                                                                    |
| models  | The Dart Model class corresponding to the Json file will be in this directory                                                 |
| states  | Save state classes that need to be shared across components in the app                                                        |
| routes  | Store all routing page classes                                                                                                |
| widgets | Some Widget components encapsulated in the APP are in this directory                                                          |

Note that using different frameworks or technology selections will have different organization methods for the code. Therefore, the code organization structure introduced in this section is not fixed or "optimal". In actual combat, readers can adjust the source code structure according to the situation. . But no matter what kind of source code organization structure is adopted, clarity and decoupling are a common principle. We should make our code structure clear for communication and maintenance.