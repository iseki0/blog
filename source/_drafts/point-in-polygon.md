---
date: 2020-09-06 17:59:00
updated: 2020-09-06 17:59:00
title: 判断多边形内点
tags: []
---
# 判断多边形内点

> 本文节选翻译自：http://geomalgorithms.com/a03-_inclusion.html ，部分图片重绘，文字亦加入了一些个人理解

有两种算法，卷绕数(Winding Number)和交叉数(Crossing Number)

- Crossing Number: 点P画一条射线，计算其与多边形边界交叉数，当交叉数是奇数时，点在内部；
- Winding Number: 计算多边形绕过点P的次数，只有卷绕数为0时点在内部

如果一个多边形是简单的（无自交），两种方法对所有的点给出相同结果。但是对于一些复杂多边形，两种方法可能对一些点给出不同的结果，比如：当一个多边形叠加自身之上时，对于重叠区域的点，如果使用wn，则被认为是在多边形之外的，而cn则不是。见下图：

![Crossing Number](./cn0.svg)![Winding Number](./wn0.svg)

在这个例子中，重叠区域的点 圈绕数 wn = 2，暗示着它被环绕在多边形中两次。明显，wn给出的答案更加直观。

尽管如此，cn还是被使用得更加广泛，这是因为cn被错误的认为比wn更快（20倍）[O'Rourke, 1998]
但事实并非如此，通过带符号的计数，wn可以获得与cn同等的效率。事实上，我们的实现的 `wn_PnPoly()` 比[Franklin, 2000], [ Haines, 1994] 和 [O'Rourke, 1998]给出的cn更快！尽管[Moller & Haines, 1999]实现的 `PointInPolygon()` 与我们的wn是差不多的，但综合考虑正确性和性能，wn是应该被首先推荐的。

## 交叉数(The Crossing Number)

这个算法通过计算由点P发出的射线跨越多边形边界的次数来决定点在内部还是外部。点是奇数则在内部，反之则在外部，这非常容易理解，见下图：

![](./cn1.svg)

在实现 cn 法时，必须遵守的一点是只统计影响“进入”和“离开”多边形的交叉。特别是，穿过顶点的情况必须被正确处理，存在如下几种交叉：

![](./cn2.svg)

此外，必须决定当一个点处于边界线上时，这个点是在多边形的内部还是外部。依照惯例，我们认为在左下边的点在内部，而右上边的点在外部。这是因为，如果不同的两个多边形共享一条边，点将只出现在其一内部，而不会同时出现在两个多边形内部，这样做避免了很多潜在的问题。

一个朴素的 Crossing Number 算法实现是，点P向X轴的正方向延伸出一条与X轴平行的射线，使用这条特别的射线可以容易地计算其与多边形边的交叉，甚至当不可能存在交叉时，判断更加容易。要计算总的交叉数 $cn$ ，算法简单地枚举多边形的所有边并测试射线与之的交叉，并在交叉时改变 $cn$ 。特别的，交叉测试必须正确处理点在边上和其他特殊情况，遵守如下规则：

### 边交叉规则(Edge Crossing Rules)

1. 向上的边包含起始点而排除结束点
2. 向下的边排除起始点而包含结束点
3. 排除水平边
4. 交点必须严格的出现在点P的右侧

可以使用这些规则计算交叉，要注意第 4 条规则导致处在右侧边缘的点被认为是在多边形的外部，而左侧边缘上的点被认为是内部。

### 伪代码

```

typedef struct {int x, y;} Point;

cn_PnPoly( Point P, Point V[], int n )
{
    int    cn = 0;    // the  crossing number counter

    // loop through all edges of the polygon
    for (each edge E[i]:V[i]V[i+1] of the polygon) {
        if (E[i] crosses upward ala Rule #1
         || E[i] crosses downward ala  Rule #2) {
            if (P.x <  x_intersect of E[i] with y=P.y)   // Rule #4
                 ++cn;   // a valid crossing to the right of P.x
        }
    }
    return (cn&1);    // 0 if even (out), and 1 if  odd (in)

}
```

需要注意的是，这个验证交叉的方式基于[若尔当曲线定理][Jordan curve theorem]，它描述了一个简单的封闭曲线把一个二维平面恰好分割成两个相连的部分：一个有界的部分包裹在另一个无界的部分中。关键在于曲线必须是简单的（即没有自交）；否则，就可能存在两个部分，之后，通过跨越边界进行in-out校验就失去了保障。并且具有共享边界的两个有界分量可能都在内部，并且越过它们的共享边界不会改变进出奇偶性。

## ~~卷绕数(The Winding Number)~~

~~另一方面，卷绕数可以精确的判断一个点是否存在于一个复杂的封闭多边形。它通过计算多边形围绕点的次数来进行判断。只有当多边形没有围绕点时（即卷绕数 $wn = 0$）点才存在于多边形之外。一般来说，定义点P在2D平面上被连续封闭曲线C围绕的次数为$wn(P,C)$，令封闭的连续2D曲线C由点$C(u)=C(x(u),y(u))$定义，对$0\le u \le 1$有$C(0)=C(1)$，点P不在C上。然后，定义向量$w(P,u)=c$~~

[Jordan curve theorem]: https://zh.wikipedia.org/wiki/%E8%8B%A5%E5%B0%94%E5%BD%93%E6%9B%B2%E7%BA%BF%E5%AE%9A%E7%90%86	"若尔当曲线定理 (Jordan curve theorem)"

~~*待续~*~~ 不会续了，楼主补数学去了，总之，这是一种优化过的转角法，它的性能并不差劲，数学证明部分我实在看不懂了 :cry:哭了，谁来救救我呢:cry: