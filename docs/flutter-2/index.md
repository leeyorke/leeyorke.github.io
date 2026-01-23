# Flutter开发之路-组件学习


先来一段代码
```dart
import 'package:flutter/material.dart';

class ExpandedWidget extends StatelessWidget {
    const ExpandedWidget({super.key});

    @override
    Widget build(BuildContext context) {
        return MaterialApp(
        // 根 Widget：提供 Material 设计主题
        home: Scaffold(
            // 骨架 Widget：提供应用基本结构
            appBar: AppBar(
            title: const Text('我的第一个 Flutter App'),
            backgroundColor: Colors.teal,
            ),
            body: Center(
            // 居中 Widget
            child: Row(
                children: <Widget>[
                    Container(color: Colors.red, width: 100),
                    Expanded( // 占据剩余空间
                    child: Container(color: Colors.blue, height: 50),
                    ),
                    Expanded( // 也占据剩余空间，且与上一个 Expanded 平均分配
                    flex: 2, // 占据的空间是上一个 Expanded 的两倍 (1:2 比例)
                    child: Container(color: Colors.green, height: 50),
                    ),
                ],
                )

            ),
        ),
        );
    }
}
```
Widget: 部件。是 `flutter` 页面的基本单位。一切皆 `Widget`。
StatelessWidget：无状态 `Widget`, 一旦创建，意味着状态也固定。可以理解为静态页面。
StatefulWidget：有状态 `Widget`, 创建之后，页面可以有状态，flutter会根据状态来渲染页面如何显示。
MaterialApp：根 Widget, 提供 Material 设计主题。
Scaffold：骨架 Widget，提供应用基本结构。一个脚手架由appbar、home、body组成。

## 布局
Expanded： 强制子部件占满父容器的剩余空间。
Flexible：

