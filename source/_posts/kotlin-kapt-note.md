---
title: 关于Kotlin注解处理器的一些坑
date: 2019-10-14 00:34:14
tags: [kotlin,kapt,java,idea,gradle]
---

1. kapt 1.3.5 存在bug，不能用，连同 kotlin-gradle-plugin 一同降级到 1.3.41
2. `build.gradle.kts` 中 `dependencies` 需要同时使用 `implementation` 和 `kapt` 引用 `com.google.auto.service:auto-service` ，否则无法识别使用Kotlin编写的注解处理器
```kotlin
dependencies{
    implementation("com.google.auto.service:auto-service:$googleAutoServiceVersion")
    kapt("com.google.auto.service:auto-service:$googleAutoServiceVersion")
}
```
3. 如果使用注解的类、函数签名、注解参数等包含顶级包声明的内容，javac可能出现找不到符号异常。不确定是不是bug。由于Windows下jdk可能使用中文locale，idea中Build中文报错可能显示成乱码，可以在Terminal中运行 `gradlew build` 查看错误原因（Terminal的文字编码是正确的）。