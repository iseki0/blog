---
title: 第一次使用 Kotlin MPP
date: 2020-04-11 14:18:15
tags: [kotlin,kotlin-mpp,gradle]
---
Kotlin 多平台也推出了好一阵子了，却一直没用过，今天第一次用，官方文档不多，记录下遇到的坑。

> 本文 Kotlin Multiplatform Gradle 插件版本 **1.3.71**

## Gradle 配置
不知道是不是我漏掉了什么开关，idea 默认生成的项目 gradle 是 `build.gradle` 而不是 `build.gradle.kts` 。一看到陌生的语法就难受，决定手动改成 Kotlin DSL。有关 DSL 的语法在这里[可以](https://www.kotlincn.net/docs/reference/building-mpp-with-gradle.html)找到。

注意如果项目同时存在 Java 代码，要开启 `withJava`
```kotlin
kotlin{
    jvm { withJava() }
}
```

## JVM-Jar 的注意事项
默认给出的 jar 的 `manifest` 文件中并不包含 `Main-Class` 项目，需要写进 gradle ：
```kotlin
kotlin{
    tasks.withType(Jar::class){
        this.manifest{
            attributes["Main-Class"] = "MainKt"
        }
        // 加入下面动作后会生成 Fat-Jar
        doFirst{
            from(configurations["runtimeClasspath"].files
            .filter { it.name.endsWith("jar") }.map { zipTree(it) })
        }
    }
}
```

待续······