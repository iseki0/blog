---
title: 用 AC 自动机给汉字注上假名
date: 2020-02-22 13:50:24
tags: [Aho–Corasick,AC自动机]
---
# 用 AC 自动机给汉字注上假名

呐，本来想撸一个歌词站，觉得应该加上一个自动注音的功能，当然这个注音肯定是不精确的（一词多音现象太严重了），特殊地方就只能手动修改了。

[AC 自动机](https://zh.wikipedia.org/wiki/AC%E8%87%AA%E5%8A%A8%E6%9C%BA%E7%AE%97%E6%B3%95) 的构建并不是太复杂，理解其原理后并不难懂，这里就简单说一下，详细内容可以在 [WikiPedia](https://zh.wikipedia.org/wiki/AC%E8%87%AA%E5%8A%A8%E6%9C%BA%E7%AE%97%E6%B3%95) 和 [OI Wiki](https://oi-wiki.org/string/ac-automaton/) 上找到。

简单来说：首先构建一个字典树，然后对每个节点添加一个失配指针（根节点的失配指针指向自己），使用时按照字典树的方式匹配字符，失配时按照失配指针回溯，直到找到一个能够匹配的节点（该节点包含待匹配的字符）；到达失配指针指向的节点的字符必然与到达当前节点的字符相同。

构建失配指针的方式非常简单，首先根节点的失配指针指向自己，然后从根节点开始对字典树进行 BFS 遍历，枚举每个节点的子节点，从当前节点向上回溯直到根节点，找到第一个同样包含指向该子节点字符的子节点，将失配指针指向它；如果没找到，就指向根节点。
```kotlin
// 构建 fail 指针
root.fail = root // root 节点的 fail 指针指向自己
root.bfs{ current -> // 从 root 节点开始做 BFS 遍历
    current.childs.forEach{ char, node -> // 枚举每个节点的子节点
        // 从当前节点的 fail 指针开始向上回溯
        // 直到找到一个儿子节点中同样拥有该字符的节点
        // 如果没找到就指向 root
        var failTo = current.fail
        while(true){
            if(failTo.hasChild(char)){
                failTo = failTo.childs[ch]
                break
            }
            if(failTo == root) break
            failTo = failTo.fail
        }
        node.fail = failTo
    }
}
```

匹配时每次都要检查 __当前节点__ 以及 __从当前节点的失配指针回溯到根途径的所有节点__ 是否是一个 word 的终结点，如果是就要考虑要不要加入到结果集中。

因为本文描述的是一个为汉字标注假名的工具，并非要求统计匹配次数的 OI 题目，并不需要保留所有结果，最长匹配原则即可。由于精确的搜索最佳方案的复杂度可能过高，也没必要，所以这里简单的用类似贪心的方式找到最长匹配即可。

（~ 待续