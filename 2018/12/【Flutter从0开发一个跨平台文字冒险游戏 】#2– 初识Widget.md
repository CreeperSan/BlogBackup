# 【Flutter从0开发一个跨平台文字冒险游戏 】#2– 初识Widget

## 前言

经过前面的折腾，我们的Flutter总算是运行起来啦，其实这段时间，Flutter也推出了 1.0 正式版，这里插一句话，如果要升级的话可以执行下面的命令来对Flutter进行升级

```
flutter upgrade
```

执行了这个命令后，稍等一段时间（我是等了好长一段时间）就升级好了，来试一试吧

## 开篇

由于我打算做的是一个简单的基于文字和少量图片的RPG游戏，游戏中不需要大量的渲染也不需要大量的主动刷新，大部分都是基于玩家的操作然后给予反馈。因此，为了省电就不使用其他的游戏引擎了，就直接使用Flutter自带的组件库来完成。

## 关于Dart

Flutter采用了Dart作为其编程语言，Dart是由Google主导开发的，目标成为下一代结构化Web的开发语言。如果想了解更多有关于Dart语言的其他东西，可以参考 [起步 | Dart](http://dart.goodev.org/guides/get-started) 官方网站，在这里我就不详述啦（因为我也不懂🤣，我还是边写游戏边感悟吧，感觉和Java有点像）

## 最简单的Hello World

回到我们上面的工程，工程里面已经默认有一个小的应用程序了，尽管它只是一个简单的计数器，但是里面也包含了不少的代码，为了快速了解了解其本事，我们还是看看一个最简单的Flutter应用程序的代码是怎么样的吧。如下

```dart
import 'package:flutter/material.dart'

void main(){
    runApp(
    	new Center(
        	child: new Text("Hello World"!),
            textDirection: TextDirection.ltr,
        )
    );
}
```

就像上面代码描述的，这个就是一个最简单的 Hello World啦，让我们开始下手吧

首先，我们先从Flutter程序的入口`void main()`开始看起

main函数里面很简单，里面就执行了一个语句`runApp`，顾名思义，这个函数就是把App给运行起来，并给它传递了一个参数 Center 类的实例。而这个Center其实就是一个Widget，这个Widget是一个容器，在他的child下面的Widget将会居中显示。而在他们的里面child下面有一个Widget —— Text，这个Widget的作用就是显示文本在屏幕上，同时我们看到这里也同时定义了 textDirection 为 TextDirection.ltr 其作用就是让文本的显示方向为从左到右显示（默认也是这样，可以省略）。综上，其实上面的程序将会在屏幕的中央显示 Hello World! 的文字。

