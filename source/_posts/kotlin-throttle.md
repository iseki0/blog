---
title: 话说我大Kotlin怎么连个throttle都没有呢
date: 2019-10-23 15:54:00
tags: [kotlin]
---
既然没有，就自己写一个糊上去吧。
总之，这东西的功能就是给函数调用加一个冷却时间。

我这里的场景是，bot要根据接收到的消息来源判断自己是否被加入了未经授权的群组，如果出现了这种群组，就发出请求退群。
那么问题就来了，在发出的请求完成前bot可能接收到大量来自同一个群组的消息，为了避免发出一堆没用的请求，最简单的办法就是加一个冷却时间。

```kotlin
fun throttle(threshhold: Long, lambda: () -> Unit) = run {
    var last = 0L
    {
        val t = System.currentTimeMillis()
        if (t - last > threshhold) {
            lambda.invoke()
            last = t
        }
    }
}
```

单元测试

```kotlin
class UtilKtTest {
    @Test
    fun throttle() {
        var count = 0
        val t = bot.throttle(200) {
            count++
        }
        repeat(8) { t.invoke() }
        assertEquals(count, 1)
        Thread.sleep(500)
        repeat(4) { t.invoke() }
        assertEquals(count, 2)
    }
}
```