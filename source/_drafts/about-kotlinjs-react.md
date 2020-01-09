---
title: 使用 gradle 构建 Kotlin React 应用
date: 2020-01-10 00:13:14
tags:
---
# 使用 gradle 构建 Kotlin React 应用

之前 JetBrain 官方提供了一个 `create-react-kotlin-app` 工具，这个东西可用来创建可使用 npm 构建的应用…现在社区似乎又打了一层包，切换到 gradle 了，但看起来底层还是极大的依赖 npm… （毕竟那一坨库还是要用的）…

似乎官方相关的文档还没有出来，那么这里就整理一点自己搜集到的东西。

### 关于 build.gradle.kts

要引入的插件倒是不多：
```gradle
 plugins {
     id 'org.jetbrains.kotlin.js' version '1.3.61'
}
```
要注意 npm 依赖不能在顶层的 dependencies 里引用，而是：
```kotlin
kotlin{sourceSets{main{dependencies{implementation(npm("",""))}}}}
```

