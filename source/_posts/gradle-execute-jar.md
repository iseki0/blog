---
title: Gradle 里执行 Jar 的几种办法
date: 2020-04-11 20:44:00
tags: [gradle]
---
> Gradle 这个东西真是让人头大······ 这里权当留个笔记备忘了，改日要好好学习下 Gradle

之前只用过 `application` 插件，结果用了 Kotlin MPP 后那个插件似乎失效了，虽然应该有办法通过手动配置继续使用那个插件，但是考虑到自己完全不熟悉 Gradle 还是不多折腾了···

以下内容来自 StackOverflow......

```kotlin
tasks.create("execute") {
    dependsOn(tasks.getByName("jvmJar"))
    doLast {
        javaexec {
            this.main = "-jar"
            val file = File(buildDir.absolutePath + "/libs/project-jvm-0.0.1.jar")
            println("Execute Jar: " + file.absolutePath)
            this.args(file.absolutePath)
        }
    }
}
```
要注意的是要写好 `manifest` 的 `Main-Class` 还要依赖的问题，最粗暴的办法无非就是打包成 Fat-Jar (x

