---
title: 尝试使用 Gradle Shadow 插件
date: 2019-12-08 18:27:59
tags: [gradle,kotlin,jar]
---

# 尝试使用 Gradle Shadow 插件

个人对 Gradle 实在不太熟悉，可能是因为用 `implementation` 代替 `compile` 的原因吧（不能确定），总之Google到的有关修改 `build.gradle` 的 `jar.from` 来生成 fat-jar 的方法对我无效。  
在网络上找了好一阵子，最终发现了 [Gradle Shadow] 这个插件，用起来很方便：

首先需要修改 gradle 的 `buildscript` 来引入这个依赖，这个东西 IDEA 默认生成的 Kotlin 项目没有，自己加进去:
```kotlin
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // 这里目前的版本号是 5.1.0 ，对应 Gradle 版本 5.x ，较低的版本可能无法使用
        classpath("com.github.jengelman.gradle.plugins:shadow:5.1.0")
    }
}
```
然后在 `plugins` 中加入这个插件就行了：
```kotlin
plugins{
    id("com.github.johnrengelman.shadow") version "5.1.0"
}
```

为了避免生成的 fat-jar 的 `META-INF/MANIFEST.MF` 中缺少 `Main-Class` 项，导致无法直接启动，建议在 `plugins` 中加入 `application`，并配置好相关属性 :
```kotlin
plugins{
    application
}

application {
    mainClassName = "yours.MainKt"
}
```
Note: 需要注意的是不建议代码中在顶层包进行任何声明，这在一些插件和库下可能出现问题。  
（比如 kapt 可能不能正确处理注解，同时由于操作系统 locale 和字符编码问题，你无法在 IDEA 中看到可以理解的错误信息）

最后就可以执行 `.\gradlew shadowjar` 来生成 fat-jar 了

更多的内容建议参考官方 User Guide: [https://imperceptiblethoughts.com/shadow/](https://imperceptiblethoughts.com/shadow/)

项目地址：[https://github.com/johnrengelman/shadow](https://github.com/johnrengelman/shadow)

Note: 本文涉及的 Gradle 代码均为 Gradle Kotlin DSL ，视情况可能需要自行修改




[Gradle Shadow]: https://github.com/johnrengelman/shadow