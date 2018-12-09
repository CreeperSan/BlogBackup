# 【Flutter】运行指令出现 Waiting for another flutter command to release the startup lock…的解决方法

## 前言

在用Flutter写文字RPG游戏的时候，运行了升级flutter的依赖包的命令，本以为很快能完成的，没想到等了好久都没反应，于是不管了，想直接热更新到机器上试试，结果终端上出现了这条命令。

嗯，好吧，因为后台有flutter在升级依赖包，所以得等待它结束后才能执行热更新。但是这个更新我都等了好一会了，一点反应都没有，这样等下去不知道到等到猴年马月，还是直接结束了这个更新吧。

于是结束了Android Studio，重新打开工程，运行程序，也发出了同样的提示。。

## 产生原因

在上面👆的情境下主要是2个原因导致的

1. **有Flutter程序在运行，锁未释放**
2. **Flutter进程异常结束，锁未能释放**

## 解决方法

针对第一种情况，解决方法也很简单

要么是等待Flutter程序运行完成正常退出，然后就能执行其他命令啦。

如果是等不及的话强行结束就会导致第二种情况的发生，其实也是有解决方法的·



针对第二种情况，

只需要吧对应的锁删除掉就可以啦，cd到Flutter的目录下，

对应的锁是在这个路径下

```
flutter/bin/cache/lockfile
```

这个文件就是了，删掉就好啦，参考命令如下，记得替换成你的路径哦

```
rm flutter/bin/cache/lockfile
```

## 参考链接

[liu_520 - flutter问题锦集](https://www.jianshu.com/p/7507d087e9f1)