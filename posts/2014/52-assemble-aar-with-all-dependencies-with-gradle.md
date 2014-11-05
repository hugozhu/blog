---
date: 2014-11-05
layout: post
title: 使用Gradle生成包含所有依赖库(.jar或.aar)的aar包
description: Assemble aar package with all dependencies using Gradle 2.1
categories:
- Blog
tags:
- Android


---

Android Library项目中如果使用Android Gradle plugin打aar包，通过maven依赖的库，或者是local依赖的aar都不会包含在生成的aar包里，如果项目是发布一个SDK，为了方便开发者使用，我们倾向于生成一个包含所有依赖库以及.so等文件的aar包。

通过反复研究和测试，以下Gradle脚本能满足需求，如果需要对代码运行ProGuard混淆，则需要使用Gradle 2.1

方法是为项目增加一个sub project（如`pkg_project`）专门用于打包，该项目中`build.gradle`内容如下：

```
apply plugin: 'java'
version = 1.0

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:19.1.0'
    }
}

repositories {
    mavenCentral()
}

dependencies {
    compile project(':<your_library_project>') //此处填写需要打包的Android Library Project name
}

task sync_jars() << {
	 //把所有依赖的.jar库都拷贝到build/aar/libs下
    copy {
        into buildDir.getPath() +"/aar/libs"
        from configurations.compile.findAll {
            it.getName().endsWith(".jar")
        }
    }
}

task sync_aars(dependsOn:':<your_library_project>:assembleRelease') << {
	 //把所有依赖的.aar库里包含的classes.jar都拷贝到build/aar/libs下，并重命名以不被覆盖
    def jar_name
    def aar_path
    def dest_dir = buildDir.getPath()+"/aar"
    configurations.compile.findAll {
        it.getName().endsWith(".aar")
    }.collect {
        aar_path = it.getPath()
        jar_name = "libs/"+it.getName().replace(".aar",".jar")
        copy {
            from zipTree(aar_path)
            into dest_dir
            include "**/*"
            rename 'classes.jar', jar_name
        }
    }
}

task fataar(dependsOn:[sync_aars, sync_jars]) << {
    task (obfuse_classes_jar, type: proguard.gradle.ProGuardTask) {
    	//把build/aar/libs/*.jar混淆后生成build/aar/classes.jar
        configuration "proguard.cfg"
        injars buildDir.getPath()+"/aar/libs"
        outjars buildDir.getPath()+"/aar/classes.jar"
        libraryjars "${System.getProperty('java.home')}/lib/rt.jar"
        libraryjars "${System.getProperty('java.home')}/Contents/Classes/classes.jar"
        libraryjars System.getenv("ANDROID_HOME")+"/platforms/android-19/android.jar"
    }.execute()

    task (gen_aar, type: Zip) {
    	//把生成最终的aar包，注意libs目录需要被排除
        def dest_dir = buildDir.getPath()+"/aar/"
        baseName = "mysdk-all"
        extension = "aar"
        destinationDir = file(buildDir.getPath())
        from dest_dir
        exclude "libs"
    }.execute()
}
```

最后就可以使用`gradlew pkg_project:fataar`来打包了

