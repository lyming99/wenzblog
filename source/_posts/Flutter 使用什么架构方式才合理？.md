---
title: Flutter 用什么架构方式才合理？
date: 2024-03-23 11:25:35
tags: Flutter
---
## 前言

刚入门 Flutter 编程时，差点被 Flutter 的嵌套地狱吓走，不过当我看到 Flutter 支持 Windows 稳定后，于是下定决心尝试接受 Flutter，因为 Flutter 真的给的太多了：跨平台、静态编译、热加载界面。

Flutter 代码是写到文件夹中的，通过文件夹来管理代码，像是 c++ 语言那样，一个文件，即可以写类，也可以直接写方法😠。

不像 java 那样，全部都是类，整齐划一，通过包名来管理，但也支持类似的“导包”😆。

那么怎样才能像 Java 那样，有个框架优化代码，让项目看起来更整洁好维护呢？

我目前的答案是 MVC 🐷，合适自己的架构才是最好的架构，用这个架构，我感觉找到了家，大家先看看我的代码，然后再做评价。



## 使用部分

结合GetX, 使用方式如下：

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'package:wenznote/commons/mvc/controller.dart';
import 'package:wenznote/commons/mvc/view.dart';

class CustomController extends MvcController {
  var count = 0.obs;

  void addCount() {
    count.value++;
  }
}

class CustomView extends MvcView<CustomController> {
  const CustomView({super.key, required super.controller});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        children: [
          Obx(() => Text("点击次数：${controller.count.value}")),
          TextButton(
            onPressed: () {
              controller.addCount();
            },
            child: Text("点我"),
          ),
        ],
      ),
    );
  }
}
```

简单粗暴，直接在 CustomView 中设计 UI, 在 CustomController 中编写业务逻辑代码，比如登录注册之类的操作。

至于 MVC 中的 Model 去哪里了？你猜猜😘。



## 代码封装部分

代码封装也很简洁，封装的 controller 代码如下

```dart
import 'package:flutter/material.dart';

class MvcController with ChangeNotifier {
  late BuildContext context;

  @mustCallSuper
  void onInitState(BuildContext context) {
    this.context = context;
  }

  @mustCallSuper
  void onDidUpdateWidget(BuildContext context, MvcController oldController) {
    this.context = context;
  }

  void onDispose() {}
}
```

封装的  view 代码如下

```dart
import 'package:flutter/material.dart';
import 'controller.dart';

typedef MvcBuilder<T> = Widget Function(T controller);

class MvcView<T extends MvcController> extends StatefulWidget {
  final T controller;
  final MvcBuilder<T>? builder;

  const MvcView({
    super.key,
    required this.controller,
    this.builder,
  });

  Widget build(BuildContext context) {
    return builder?.call(controller) ?? Container();
  }

  @override
  State<MvcView> createState() => _MvcViewState();
}

class _MvcViewState extends State<MvcView> with AutomaticKeepAliveClientMixin{
  @override
  bool get wantKeepAlive => true;

  @override
  void initState() {
    super.initState();
    widget.controller.onInitState(context);
    widget.controller.addListener(onChanged);
  }

  void onChanged() {
    if (context.mounted) {
      setState(() {});
    }
  }

  @override
  Widget build(BuildContext context) {
    super.build(context);
    widget.controller.context = context;
    return widget.build(context);
  }

  @override
  void didUpdateWidget(covariant MvcView<MvcController> oldWidget) {
    super.didUpdateWidget(oldWidget);
    widget.controller.onDidUpdateWidget(context, oldWidget.controller);
  }


  @override
  void dispose() {
    widget.controller.removeListener(onChanged);
    widget.controller.onDispose();
    super.dispose();
  }
}
```

## 结语

MVC 可以很简单快速的将业务代码和 UI 代码隔离开，改逻辑的时候就去找 Controller 就行，改 UI 的话就去找 View 就行，和后端开发一样的思路，完成作品就行。

附上的作品文件结构截图，亲喷哈~

![](assets/04ab4670-d62d-11ee-b1e9-af9546993c52)

感谢大家的关注与支持，后续继续更新更多f lutter 跨平台开发知识，例如：MVC 架构中的 Controller 应该在哪里创建？Controller 中的 Service 应该在哪里创建？

作品地址：[https://github.com/lyming99/wenznote](https://github.com/lyming99/wenznote)
