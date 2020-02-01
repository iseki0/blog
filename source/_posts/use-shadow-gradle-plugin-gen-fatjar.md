---
title: 使用 Gradle Shadow 插件生成 Fat-Jar
date: 2019-12-08 18:27:59
tags: [gradle,kotlin,jar]
---

# 使用 Gradle Shadow 插件生成 Fat-Jar

在网络上找了好一阵子，最终发现了 [Gradle Shadow] 这个插件，用起来很方便：

首先需要修改 gradle 的 `buildscript` 来引入这个依赖，这个东西 IDEA 默认生成的 Kotlin 项目没有，自己加进去:
```kotlin
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // 这里目前的版本号是 5.2.0 ，对应 Gradle 版本 5.x ，较低的版本可能无法使用
        classpath("com.github.jengelman.gradle.plugins:shadow:5.2.0")
    }
}
```
然后在 `plugins` 中加入这个插件就行了：
```kotlin
plugins{
    id("com.github.johnrengelman.shadow") version "5.2.0"
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
<!-- **Note:** 需要注意的是不建议代码中在顶层包进行任何声明，这在一些插件和库下可能出现问题。  
（比如 kapt 可能不能正确处理注解，同时由于操作系统 locale 和字符编码问题，你无法在 IDEA 中看到可以理解的错误信息） -->

最后就可以执行 `.\gradlew shadowjar` 来生成 fat-jar 了

这些项也可从 `shadowJar` 配置

**Note:** 由于目前版本 `5.2.0` 尚未支持 Kotlin DSL, 如需访问 `shadowJar` 进行更详细的配置（如：[过滤 jar 包中的内容](https://imperceptiblethoughts.com/shadow/configuration/filtering/)）
目前的 Workaround 如下： [>>](https://github.com/johnrengelman/shadow/issues/533#issue-541921197)
```kotlin
val shadowJar: com.github.jengelman.gradle.plugins.shadow.tasks.ShadowJar by tasks
```

更多的内容建议参考官方 User Guide: [https://imperceptiblethoughts.com/shadow/](https://imperceptiblethoughts.com/shadow/)

项目地址：[https://github.com/johnrengelman/shadow](https://github.com/johnrengelman/shadow)

**Note:** 本文涉及的 Gradle 代码均为 Gradle Kotlin DSL ，视情况可能需要自行修改




[Gradle Shadow]: https://github.com/johnrengelman/shadow