---
title: 在 Windows 上使用 OpenPGP 签署 Git 提交的一点点注意事项
date: 2020-06-11 00:00:00
updated: 2020-06-11 00:00:00
tags: [pgp,git,window]
---

# 在 Windows 上使用 OpenPGP 签署 Git 提交的一点点注意事项

可能和 Git for Windows 在安装时的选项有关，需要注意默认情况下 git 可能不会使用 Windows 中安装的 gpg.exe，这必然会导致找不到key。（要不是因为看到了gpg的初始化输出我打死也想不到是这个问题）

`git config --global gpg.program "c:/Program Files (x86)/GnuPG/bin/gpg.exe"` 即可，注意看准了自己机器上的路径是啥。

另 `git config commit.gpgsign true` 即可设置该仓库默认签名提交，IDEA里 commit 时就会弹出 Git for windows 的密钥窗口了。