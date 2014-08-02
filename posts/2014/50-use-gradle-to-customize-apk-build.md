---
date: 2014-08-03
layout: post
title: 使用Gradle生成一个App的不同版本，且可以同时安装在一个手机上
description: Customizing your Android App build with Gradle
categories:
- Blog
tags:
- Android

---


## 背景
开发一个App一般会生成内测版和正式版，甚至还会有不同渠道的版本，不同版本的配置可能会不一样，比如内测版会需要记录完整的日志。

Android手机对于同样的Application Id的App只能安装一个版本，如果我们需要同时安装内测版和正式版，就必须修改其中一个版本的Application Id。

## 解决方案

Gradle支持buildTypes和productFlavors两种定制方法，这里只介绍通过buildType的解决方案。通过productFlavors则可有效解决渠道包，arm，x86等分平台以及付费版和广告版的打包问题。


### 修改debug版的包名

配置如下：

```
android {
    buildTypes {
        release {
            ...
        }

        debug {
            applicationIdSuffix '.debug'
            ...
        }
    }
}
```

### 修正资源文件里的包名

如果你的项目里使用了自定义的View，且有自定义的属性时，会需要修正一下xml命名空间里的包名。

```
android.applicationVariants.all { variant ->
    def buildType = variant.buildType
    def encoding = java.nio.charset.Charset.defaultCharset().toString()
    if (buildType.applicationIdSuffix) {
        def defaultPackageId = variant.packageName.replaceAll(buildType.applicationIdSuffix,'')
        variant.mergeResources.doLast {
            def dir = file("${buildDir}/intermediates/res/${variant.dirName}/layout")
            dir.listFiles().each { f->
                String content = f.getText(encoding)
                content = content.replaceAll("res/"+defaultPackageId, "res/"+variant.packageName)
                f.write(content, encoding)
            }
        }
    }
}
```

### 定制APK的应用名称

如果同时安装两个版本，那么最好能从应用名称上来区别一下，一般我们在`AndroidManifest.xml`中使用String resource来命名，如下：

```
    <application
    	...
        android:label="@string/app_name" >
```        

在`build.gradle`里增加下面的代码就可以为debug版一个特殊的命名了

```
android.applicationVariants.all { variant ->
    def buildType = variant.buildType
    def encoding = java.nio.charset.Charset.defaultCharset().toString()
    if (buildType.applicationIdSuffix) {
        def defaultPackageId = variant.packageName.replaceAll(buildType.applicationIdSuffix,'')
        variant.mergeResources.doLast {
            def f = file("${buildDir}/intermediates/res/${variant.dirName}/values/values.xml")
            String content = f.getText(encoding)
            content = content.replaceAll('来往','来往Beta')
            f.write(content,encoding)
        }
    }
}
```

### 修改ContentProvider Authority

如果你的应用里还提供ContentProvider的话，如下：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
   package="com.example.android.app"
   android:versionCode="1"
   android:versionName="1">

   <application>

       <provider
           android:name=".provider.Provider1"
           android:authorities="com.example.android.app.debug.provider"
           android:exported="false" />

   </application>
</manifest>
```
也可以用类似的方法修改。


通过在`build.gradle`里增加上面的方法后，可以在`gradle build`后生成debug和release两个包，且可以同时安装到一个手机上。


# 参考链接

1. http://toastdroid.com/2014/03/28/customizing-your-build-with-gradle/
2. http://stackoverflow.com/questions/16777534/using-build-types-in-gradle-to-run-same-app-that-uses-contentprovider-on-one-dev



