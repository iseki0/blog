---
date: 2020/5/31 21:00:00
updated: 2020/5/31 21:00:00
title: 解决 CLion + MSVC 下的字符编码问题

---

# 解决 CLion + MSVC 下的字符编码问题

第一次这么用，上来字符编码就炸了，不出意外 log 中会出现如下内容

```
warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
```

然后就是诡异的编译失败语法错误，比如换行符、分号等等

原因是 CLion 默认使用 UTF-8 编码，MSVC 继承了 MS 家族的一贯传统，除非明确指定否则要么 UTF-8 with BOM 要么当前代码页。

解决办法也简单，加上命令行开关就行了： `/utf-8` 或者 `source-charset:utf-8 /execution-charset:utf-8` [参见MSVC文档>>](https://docs.microsoft.com/en-us/cpp/build/reference/utf-8-set-source-and-executable-character-sets-to-utf-8)

默认创建的项目是 CMake 的，在 `CMakeList.txt` 中加入如下内容即可

```cmake
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")
```

括号中表达式语法具体参见 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7))