# 【Android】 SimpleDateFormat在Android设备上的一个坑

### 前言

SimpleDateFormat是Java中一个用于格式化时间的工具是很好用的一个类，但是在Android设备上使用时却存在一个坑。

### 过程

某天被告知存在一个Bug，某个功能完全用不了，一点击就闪退。当时就觉得有点疑惑，在开发时还是好好的呀？难道是打包打错了？于是乎，抓取了相关日志，查看到报的错误是

```
java.lang.IllegalArgumentException：Unknown pattern character ‘Y’
```

定位到的崩溃的地方是这个

```
val data = SimpleDateFormat("YYYY-MM-dd")
```

在我手机上市好好的呀，为什么在他的手机上就有问题呢？经过查询，才发现是`SimpleDateFormat`在 API24( 7.0 ) 一下的手机时，使用了大写字母 Y 作为格式化时间的问题

`YYYY`代表的是 Weak Year，按照 [BugFree_张瑞](https://blog.csdn.net/u011489043) 的博客上的解释为

> y 是 Year、Y 是 Week Year。至于 Week Year 是什么？按周计年的第一周是指：以星期一为周始，以星期日为周末，第一个包含该年度四天以上的周。

因此，准确的来说对应的正确的格式化应该是 `yyyy-MM-dd`

### 参考资料

[SimpleDateFormat Y 与 y 区别 及 崩溃 Unknown pattern character 'Y' - BugFree_张瑞](https://blog.csdn.net/u011489043/article/details/88877908)

[Android Developer - SimpleDateFormat](https://developer.android.com/reference/java/text/SimpleDateFormat.html)