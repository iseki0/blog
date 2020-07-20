---
date: 2020-07-20 19:30:00
title: 浅扯 Coroutine
tags: [kotlin, coroutine]
---
# 浅扯 Coroutine

关于 Kotlin Coroutine 的使用就不多说了，大家都已经很熟悉emmm，这里简单探索下 Coroutine 内部的实现，来尽量规避Coroutine 与其他库和框架协同使用时的坑。

本文编写时使用 Kotlin 版本：`1.4-M3`，JDK 版本：`openjdk 11.0.7`

Coroutine 的核心原理 [KEEP][KEEP] 中的 coroutine 提案已经写得很清楚了，详见提案的实现详情章节： [KEEP/proposals/coroutines.md#implementation-details](https://github.com/Kotlin/KEEP/blob/master/proposals/coroutines.md#implementation-details) ，这里补充一点点提案中没说的，和实现高度相关的内容。

本质就是对 suspend 函数进行CPS变换，将代码转换为[延续体传递风格][Continuation-passing style]。简单来说就是为每个函数增加了一个隐式的参数 `$completion`，它的类型是 `Continuation` 即延续体；函数本体则被编译成状态机，状态存储于上述的 `Continuation` 中，suspend 函数每次 resume 时都会被调用，其根据 `Continuation` 中的状态信息直接调转到对应的位置继续执行。

为了便于分析，这里使用 [CFR][CFR] 反编译器对 kotlinc 编译的 suspend 函数进行反编译，当前版本为 `0.150`

我们从下面代码段开始：

```kotlin
class Demo {
    suspend fun demo1(s: String) {
        println("1==========")
        delay(100)
        println("2==========")
    }
}
```

注意，Kotlin 编译器对其进行编译后我们看到两个 class `Demo$demo1$1.class` `Demo.class` 前面那个是后面的匿名内部类，不用管它，用 cfr 反编译后面的就行了。以下我将只节选部分源码，注意这不是标准的Java代码，没法直接编译

```java
public final class Demo {
    /*
     * Unable to fully structure code
     * Enabled aggressive block sorting
     * Lifted jumps to return sites
     */
    @Nullable
    public final Object demo1(@NotNull String s, @NotNull Continuation<? super Unit> $completion) {
        if (!($completion instanceof demo1.1)) ** GOTO lbl-1000
            // 这个 demo1.1 就是下面new出来的那个ContinuationImpl
        var6_3 = $completion;
        if ((var6_3.label & -2147483648) != 0) {
            var6_3.label -= -2147483648;
        } else lbl-1000:
        // 2 sources

        {
            $continuation = new ContinuationImpl(this, $completion){
                /* synthetic */ Object result;
                int label;
                final /* synthetic */ Demo this$0;
                Object L$0;
                Object L$1;

                @Nullable
                public final Object invokeSuspend(@NotNull Object $result) {
                    this.result = $result;
                    this.label |= Integer.MIN_VALUE;
                    return this.this$0.demo1(null, (Continuation<? super Unit>)this);
                }
                {
                    this.this$0 = demo;
                    super(continuation);
                }
            };
        }
        $result = $continuation.result;
        var7_5 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
        switch ($continuation.label) {
            case 0: {
                ResultKt.throwOnFailure((Object)$result);
                var3_6 = "1==========";
                var4_7 = false;
                System.out.println((Object)var3_6);
                $continuation.L$0 = this;
                $continuation.L$1 = s;
                $continuation.label = 1;
                v0 = DelayKt.delay((long)100L, (Continuation)$continuation);
                if (v0 == var7_5) { // if result == COROUTINE_SUSPENDED
                    return var7_5;  //     return COROUTINE_SUSPEND
                }
                ** GOTO lbl27       // else goto lbl27
            }
            case 1: {
                // 状态机在这里 resume，先恢复局部变量
                s = (String)$continuation.L$1;
                this = (Demo)$continuation.L$0;
                ResultKt.throwOnFailure((Object)$result);
                v0 = $result;
lbl27:
                // 2 sources
                // 如果一开始就没有挂起，那自然也就不需要恢复咯
                var3_6 = "2==========";
                var4_7 = false;
                System.out.println((Object)var3_6);
                return Unit.INSTANCE;
            }
        }
        throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
    }
}
```

上面这个很清晰了，这个 suspend 真正被调用时会创建一个 ContinuationImpl 的子类，里边存放了状态机的状态 `label` 和局部变量，还有最关键的，传进来的调用者的延续体也被包含在了里面，这个稍后会用到。

当协程挂起时，状态机函数会返回 `COROUTINE_SUSPEND` 这个对象，这也就是 suspend 函数编译后函数返回值必然为 Any 的原因，实际的返回值是 返回值 T 和 COROUTINE_SUSPEND 之一，显然这是在 Java 和 Kotlin 类型系统中均无法表达的。局部变量等状态信息，都会在调用下一个 suspend 函数前保存进延续体 (45~47) 

 接下来看一看 CoroutineImpl 这个类到底干了什么，这个类的代码很多，我不全粘过来了，只节选有意义的部分

```kotlin
public final override fun resumeWith(result: Result<Any?>) {
        // This loop unrolls recursion in current.resumeWith(param) to make saner and shorter stack traces on resume
        var current = this
        var param = result
        while (true) {
            // Invoke "resume" debug probe on every resumed continuation, so that a debugging library infrastructure
            // can precisely track what part of suspended callstack was already resumed
            probeCoroutineResumed(current)
            with(current) {
                val completion = completion!! // fail fast when trying to resume continuation without completion
                val outcome: Result<Any?> =
                    try {
                        val outcome = invokeSuspend(param)
                        if (outcome === COROUTINE_SUSPENDED) return
                        Result.success(outcome)
                    } catch (exception: Throwable) {
                        Result.failure(exception)
                    }
                releaseIntercepted() // this state machine instance is terminating
                if (completion is BaseContinuationImpl) {
                    // unrolling recursion via loop
                    current = completion
                    param = outcome
                } else {
                    // top-level completion reached -- invoke and return
                    completion.resumeWith(outcome)
                    return
                }
            }
        }
    }
```

其实这个函数做的事很简单，不停的循环并调用 current 指针的 invokeSuspend 函数来恢复协程的执行，如果返回 COROUTINE_SUSPEND 那就意味着又是暂停，返回；否则说明当前函数执行完了，从 current 指针指向的延续体中拿出它上一级的延续体，继续 invoke，直到回到根，结束退出。

总结下，传入每个 suspend 函数的延续体在初始时都是调用者的延续体，当 resume 时会传入本函数的延续体，并根据里边的 label 去往相应的状态，同时协程暂停时会返回那个特殊的 COROUTINE_SUSPEND 对象。





[Continuation-passing style]: https://en.wikipedia.org/wiki/Continuation-passing_style "[CPS] 延续体传递风格（WikiPedia[英文]）"

[KEEP]: https://github.com/Kotlin/KEEP "Kotlin Evolution and Enhancement Process"

[CFR]: https://www.benf.org/other/cfr/ "CFR - another java decompiler"