---
title: 使用 gradle 构建 Kotlin React 应用
date: 2020-01-10 00:13:14
tags: [kotlin,react,kotlin-js,js,gradle]
---
# 使用 gradle 构建 Kotlin React 应用

之前 JetBrain 官方提供了一个 `create-react-kotlin-app` 工具，这个东西可用来创建可使用 npm 构建的应用…现在社区似乎又打了一层包，切换到 gradle 了，但看起来底层还是极大的依赖 npm… （毕竟那一坨库还是要用的）…

似乎官方相关的文档还没有出来，那么这里就整理一点自己搜集到的东西。

### 关于 build.gradle.kts

要引入的插件和依赖倒是不多(?)：
```kotlin
plugins {
    // 这里的版本最好和下方统一
    id("org.jetbrains.kotlin.js") version "1.3.60"
}
dependencies {
    implementation(kotlin("stdlib-js"))
}
```

### React 相关依赖

要注意 npm 依赖不能在顶层的 dependencies 里引用，而是：
```kotlin
kotlin {
    sourceSets["main"].dependencies {
        // 一定要引入 React 的一系列依赖
        implementation(npm("react", "16.9.0"))
        implementation(npm("react-dom", "16.9.0"))
        implementation(npm("core-js", "3.4.8"))
    }
}
```


Maven 版的 Kotlin-React-Wrapper 依赖:
```kotlin
    implementation("org.jetbrains:kotlin-react:16.9.0-pre.89-kotlin-1.3.60")
    implementation("org.jetbrains:kotlin-react-dom:16.9.0-pre.89-kotlin-1.3.60")
```
以上依赖并未发布在中央仓库，所以一定引入
```kotlin
repositories {
    maven("https://dl.bintray.com/kotlin/kotlin-eap/")
    maven("https://dl.bintray.com/kotlin/kotlin-js-wrappers/")
}
```

可在如下地址找到版本信息等：
- https://www.npmjs.com/package/@jetbrains/kotlin-react
- https://www.npmjs.com/package/@jetbrains/kotlin-react-dom

可使用 `gradlew run / gradlew browserRun` 拉起开发服务， `gradlew browserWebpack` 开始生产构建
