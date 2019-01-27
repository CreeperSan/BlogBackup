# 【Android】Toolbar中app:showAsAction=”always”无效的解决方法

## 起源

在给`Toolbar`加载菜单的时候，发现菜单中`Item`设置的`app:showAsAction="always"`不生效，`Item`始终都是隐藏起来的，也就是说要点击三个点的按钮才能显示出来

## 过程

网上有人说需要把命名空间从`android`修改成`app`，即可生效，但是我这里本来就是`app`的命名空间呀，要不反过来试试(笑)？把`app`修改为`android`后直接报 `Should use app:showAsAction with the appcompat library with xmlns:app="http://schemas.android.com/apk/res-auto"` 既然这里修改没生效，可以尝试在代码中修改掉这个标志位，尝试一下，成功了。虽然这个方法解决的不是很优雅，但是也在这里记下来把

## 解决方法

```kotlin
videoPlayerToolbar.apply {
    inflateMenu(R.menu.video_player)
    menu.findItem(R.id.menuVideoPlayerLock).setShowAsAction(MenuItem.SHOW_AS_ACTION_ALWAYS)
    menu.findItem(R.id.menuVideoPlayerRotate).setShowAsAction(MenuItem.SHOW_AS_ACTION_ALWAYS)
    menu.findItem(R.id.menuVideoPlayerScale).setShowAsAction(MenuItem.SHOW_AS_ACTION_ALWAYS)
}
```

