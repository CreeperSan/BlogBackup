#【Android】Canvas绘制开启抗锯齿的一些方法

### 前言

在重构头像组件的时候，在Canvas中绘制头像时，发现即使Paint开启抗锯齿，其绘制出来的图形依然有锯齿。经过查找和瞎蒙尝试后发现有下面3种方法开启抗锯齿，在这里简单记录一下

### 方法

1. 给画笔添加抗锯齿

   给画布添加抗锯齿，那么在使用Paint绘制图形的时候，则带有抗锯齿效果

   可以在实例化画笔的时候添加

   ```kotlin
   val paint = Paint(Paint.ANTI_ALIAS_FLAG)
   ```

   也可以在实例化画笔后进行设置

   ```kotlin
   paint.style = Paint.Style.FILL
   ```

   效果对比，左侧为开启抗锯齿的图形，右侧为关闭抗锯齿的图形，图片显示比例为 200%

   ![img01](img/01.png)

2. 给画布添加抗锯齿（没尝试过）

   如果有时候给画笔添加抗锯齿无效的话，可以尝试给Canvas增加抗锯齿

   ```kotlin
   canvas.drawFilter = PaintFlagsDrawFilter(0, Paint.ANTI_ALIAS_FLAG or Paint.FILTER_BITMAP_FLAG)
   ```

3. Bitmap添加抗锯齿

   针对于图片缩放时所产生的锯齿，我们可以在缩放图片时指定抗锯齿即可

   ```kotlin
   val canvas = Bitmap.createBitmap(
                   sourceBitmap,
                   0,
                   0,
                   size,
                   size,
                   matrix,
                   true // 此处 filter 指定为 true 即可
               )
   ```
   类似的方法也是把 filter 传入 true 即可
   
   效果对比，左侧为开启抗锯齿的图像，右侧为关闭抗锯齿的图像，图片显示比例为 100%
   
   ![img02](img/02.png)

### 参考资料

[lazy-ape - bitmap缩放时抗锯齿](https://www.cnblogs.com/lazy-ape/p/5515154.html)

