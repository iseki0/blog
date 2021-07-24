---
date: 2018-09-07 22:29:00
updated: 2018-09-07 22:29:00
title: 第一次用ANTLR呢
tags: [antlr,java]
---
觉得既然是第一次尝试ANTLR嘛，那就来个简单点的，parse Windows 的 hosts 文件吧，结果......

先上一段 grammar:

```antlr
grammar hosts;

hostfile: line* EOF;
line : item
     | comment
     ;
item: IPADDRESS HOSTNAME ;
comment: COMMENT;
HOSTNAME: [A-z0-9.]+;
IPADDRESS: NUM '.' NUM '.' NUM '.' NUM;
NUM: [0-9] +;
COMMENT: '#' .*? '\r'? '\n';
NL: ('\r\n'|'\n\r'|'\r'|'\n') ->skip;
BLANK: (' '|'\t') ->skip;
```
乍一看好像没什么问题，挺好的，然而`item`那条文法规则就是不起作用，前面注释都正常处理了，结果ip地址出不来。
网上搜了半天也搜不到，于是去StackOverflow上找了找，有个现成的。
[StackOverflow: antlr-4-5-mismatched-input-x-expecting-x](https://stackoverflow.com/questions/29777778/antlr-4-5-mismatched-input-x-expecting-x)
解决办法就是把`IPADDRESS`挪到`HOSTNAME`上面就行了，原因是ANTLR生成的 parser 和 lexer 几乎是独立工作的，在不考虑内嵌动作的情况下parser不能影响lexer；lexer会简单的按照最长匹配的原则，一样长的就看grammar中出现的顺序。。。

所以之前那个文法把IP地址都匹配成了`HOSTNAME`...

问题解决，还是没好好看书，书上应该都写了的...