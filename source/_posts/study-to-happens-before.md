---
date: 2019-12-27 20:08:00
title: 复习有关 happens-before rules
tags: [jls,java,translate]
---
# Java 中的 happens-before 原则

### 以下内容翻译自节选 Oracle Java SE 7 语言规范(JLS) [Link>>>](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html)

> Two actions can be ordered by a *happens-before* relationship. If one action *happens-before* another, then the first is visible to and ordered before the second.  
If we have two actions x and y, we write hb(x, y) to indicate that x *happens-before* y.


可以通过 *happens-before* 关系来排序两个动作。如果一个动作 *happens-before* 另一个，则前者对后者可见，且在后者前发生  
如果我们有两个动作 x 和 y，我们使用 `hb(x,y)` 来代表 x 在 y 之前发生

> If x and y are actions of the same thread and x comes before y in program order, then hb(x, y).

如果 x 和 y 是发生在同一个线程上的动作，且在编程顺序上，x 在 y 之前，那么 `hb(x,y)`

> There is a *happens-before* edge from the end of a constructor of an object to the start of a finalizer [(§12.6)](https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.6) for that object.

一个对象的构造方法结束于它的 finalizer 开始之前

> If an action x synchronizes-with a following action y, then we also have hb(x, y).

如果动作 x 同步于其后的动作 y，则我们也有 `hb(x,y)`

> If hb(x, y) and hb(y, z), then hb(x, z).

如果 `hb(x,y)` 且 `hb(y,z)` 那么`hb(x,z)`

> The `wait` methods of class `Object` [(§17.2.1)](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.2.1) have lock and unlock actions associated with them; their *happens-before* relationships are defined by these associated actions.

`Object` 类的 `wait` 方法具有与之关联的锁定和解锁动作；他们的 *happens-before* 关系由这些关联的动作定义

> It should be noted that the presence of a *happens-before* relationship between two actions does not necessarily imply that they have to take place in that order in an implementation. If the reordering produces results consistent with a legal execution, it is not illegal.

应当指出的是，两个动作之间存在 *happens-before* 关系并不一定意味着在实现中它们必须按照该顺序进行。如果重新排序产生的结果与合法执行相符，则是合法的。

> *For example, the write of a default value to every field of an object constructed by a thread need not happen before the beginning of that thread, as long as no read ever observes that fact.*

*比如，一个线程构造一个对象时，向该对象的每个字段写入默认值并不需要发生于该线程开始前，只要事实上不可被观察到*

> More specifically, if two actions share a *happens-before* relationship, they do not necessarily have to appear to have happened in that order to any code with which they do not share a *happens-before* relationship. Writes in one thread that are in a data race with reads in another thread may, for example, appear to occur out of order to those reads.

更明确的说法，如果两个动作具有 *happens-before* 关系，在与之没有共享 *happens-before* 关系的其他代码看来，他们并不一定具有相同的关系。在一个线程中写入也许会和另一个线程的读取存在数据竞争，比如，这些读取是乱序的。

