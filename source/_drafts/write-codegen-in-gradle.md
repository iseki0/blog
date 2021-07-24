---
title: 在 Gradle 里写 Codegen 为 Kotlin 的每个类生成 xxxOf()
date: 2020-04-12 21:09:42
updated: 2020-04-12 21:09:42
tags: [gradle]
---
# 在 Gradle 里写 Codegen 为 Kotlin 的每个类生成 xxxOf()
这两天也不知道咋回事，和 codegen 杠上了，也未必节省多少时间，复杂的 codegen 写起来挺累的······ 上次在 Vert.x 里看到了 `classnameOf(params...)` 这种写法，例如 `httpOptionsOf(port = 8080,...)` 顿时感觉非常舒服，决定也给自己的代码加上。

codegen 的方案无非就这几种：

1. 独立的一个生成器，用起来麻烦，不能很舒服的自动化，但是写起来简单啊（x
2. 利用 Java 的 APT 注解处理器，缺点是首先只能支持 Java，Kotlin/JVM 虽然有 KAPT 可以用，但并不舒服，而且有暗坑；其次最关键的是这东西是完全在编译时生成代码，一般的设计方式都不会保留编译时生成的源码文件。这意味着 IDE 的提示会爆炸，一般的解决方案是再写一个 IDE 插件······
3. 编译器插件······ 这个，虽然解决了注解处理器仅支持 JVM 的问题，但依旧不能解决 IDE 提示的问题，而且工作量会爆炸······
4. 独立的 codegen，利用 gradle 的 task，在恰当的时候触发，同时生成的代码包含在项目的源集中，不用写 IDE 插件了（

当然，可以粗暴的开一个子模块单独编译单独运行，但是实在是太臃肿且没必要了

参考下 Gradle 的官方文档，插件的存在主要有两种形式，一种是最简单的，直接一个 `*.gradle(.kts)?` 文件；另一种是正经的插件，官方的说法是 Binary Plugin，后者能提供更强大的能力，前者充其量就是个脚本。
平时使用的插件基本都是二进制插件，如 `java` `kotlin全家桶` `maven-*` 等等，

就写一个临时用的 codegen 实在没必要花费太大精力研究 Gradle 的官方文档，那么就从一个简单的文件写起。

最简单的办法：
```codegen.gradle.kts[kotlin]
println("Hello world")
```
```build.gradle.kts[kotlin]
apply(from = "codegen.gradle.kts")
```
不出意外，每次执行 `gradlew` 时都能看到控制台中打出的 Hello world。那么其实剩下的事情就很简单了。（感觉最困难的是完全不熟悉的 Gradle 😥

需要做的东西并不多，Kotlin 的文法文件有现成的，用 antlr 生成下就可以用了。这里用的是来自 antlr 官方仓库的文件：[https://github.com/antlr/grammars-v4/tree/master/kotlin/kotlin-formal](https://github.com/antlr/grammars-v4/tree/master/kotlin/kotlin-formal)

把它丢进 `buildSrc/src/main/[java|kotlin|groovy]` 然后就可以在合适的 `gradle.kts` 文件中使用了。当然如果觉得这不够好，也可以在完全独立的项目中构建一个 jar，把他包含进 `buildscript{}` 就像那些插件一样。

待续······