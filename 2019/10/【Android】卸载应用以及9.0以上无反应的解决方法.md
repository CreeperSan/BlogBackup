# 【Android】卸载应用以及9.0以上无反应的解决方法

## 前言

正在写一个自己使用的文件管理器，里面逻辑牵涉到卸载手机已经安装的应用程序。

## 相关逻辑

卸载一个应用有三种方法

1. 使用Intent弹出提示框让系统卸载

   第一种是最常用的方法，让系统弹出对话框请求用户确认卸载。相关代码如下

   ```kotlin
   val uri = Uri.fromParts("package", "包名", null)
   val intent = Intent(Intent.ACTION_DELETE, uri)
   startActivity(intent)
   ```

2. 使用PackageManager静默卸载

   此方法只有系统应用才能使用，需要的权限如下

   ```
   <uses-permission android:name="android.permission.DELETE_PACKAGES"/>
   ```

   不过这个权限强行添加到`AndroidManifest.xml`也无法编译过，不过可以将应用反编译然后修改完`AndroidManifest.xml`再重新打包。当然应该还有别的办法，只是暂时我还不知道..

   使用方法如下，摘自 [yunterry](https://blog.csdn.net/ta_ab) 的博客

   ```java
   private class PackageDeleteObserver extends IPackageDeleteObserver.Stub {  
       private int position;  
       private int mFlag;  
   
       public PackageDeleteObserver(int index, int flag) {  
           position = index;  
           mFlag = flag;// 0卸载1个包，1卸载N个包 N>1  
       }  
   
       @Override  
       public void packageDeleted(String arg0, int arg1)  
               throws RemoteException {  
           // TODO Auto-generated method stub 
           Message msg;  
           msg = mHandle.obtainMessage();  
           msg.what = FLAG_DELETE_VIRUS;  
           msg.arg1 = position;  
           msg.arg2 = mFlag;  
           msg.sendToTarget();  
       }  
   }
   
   PackageManager pkgManager = mContext.getPackageManager();  
   PackageDeleteObserver observer = new PackageDeleteObserver(currVirus, 1);  
   pkgManager.deletePackage("包名", observer, 0);  
   ```

   最后进行签名

   ```shell
   java -jar signapk.jar platform.x509.pem platform.pk8 test.apk test_signed.apk
   ```

   编译生成APK时，要在manifest文件下添加`Android:sharedUserId=”android.uid.system”`

   ```xml
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"  
        package="com.xieyuan.mhfilemanager"  
        android:versionCode="1"  
        android:versionName="1.0"  
        android:installLocation="internalOnly"  
        android:sharedUserId="android.uid.system" >  
   ```

   

3. 使用pm命令静默卸载

   此方法需要root权限，相关使用代码如下

   ```java
   //pm命令可以通过adb在shell中执行，同样，我们可以通过代码来执行 
   public static String execCommand(String... command) {
       Process process = null;
       InputStream errIs = null;
       InputStream inIs = null;
       String result = "";
       try {
           process = new ProcessBuilder().command(command).start();
           ByteArrayOutputStream baos = new ByteArrayOutputStream();
           int read = -1;
           errIs = process.getErrorStream();
           while ((read = errIs.read()) != -1) {
               baos.write(read);
           }
           inIs = process.getInputStream();
           while ((read = inIs.read()) != -1) {
               baos.write(read);
           }
           result = new String(baos.toByteArray());
           if (inIs != null)
               inIs.close();
           if (errIs != null)
               errIs.close();
           process.destroy();
       } catch (IOException e) {
   
           result = e.getMessage();
       }
       return result;
   }
   ```

   使用时，只需要

   ```java
   execCommand("pm","uninstall", "包名");
   ```

## 遇到的问题

在使用第一种方法时，在我的手机上却无法弹出删除应用的对话框。经过一番检索，原来是在Android 9.0及以上的版本需要生命多一个权限

```
<uses-permission android:name="android.permission.REQUEST_DELETE_PACKAGES" />
```

增加权限即可正确弹出删除对话框

## 尾巴

发现在Android的高版本里，Google逐步收紧了开发者所能自由控制的权限。当然，这也是为了提高Android系统的安全性，减少恶意应用的一个做法，对整个Android系统的生态都是有积极作用的。~~只是苦了开发者~~，觉得有必要对Android1的大版本升级Api变动做一个总结了。。。

## 参考资料

[Android APP中卸载其他APP的三种方法 - yunterry](https://blog.csdn.net/ta_ab/article/details/77949348)

[AndroidPie（9）使用Intent卸载应用无任何反应解决方法 - 阿闯学长](https://www.jianshu.com/p/cb60962b1fff)

