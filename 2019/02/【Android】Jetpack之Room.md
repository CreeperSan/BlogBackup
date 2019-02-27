#【Android】Jetpack之Room

## 前言



## 引入

首先，引入google maven仓库

```maven
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
```

然后添加room依赖

```
implementation "android.arch.persistence.room:runtime:1.1.1"
annotationProcessor "android.arch.persistence.room:compiler:1.1.1"
```

## 使用

