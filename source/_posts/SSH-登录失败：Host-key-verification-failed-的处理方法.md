---
title: ' SSH 登录失败：Host key verification failed 的处理方法'
date: 2016-08-08 17:00:00
tags: [ssh,error]
categories: linux
---

SSH 登录失败：Host key verification failed
由于公钥不一样了，所以无法登录，提示信息是 KEY 验证失败。
解决方法是：

在 /root/.ssh/known_hosts  文件里面将原来的公钥信息删除即可。
SSH 报 Host key verification failed.
一般来说，出现该错误有这么几种可能:

1. ssh/known_hosts 裡面记录的目标主机 key 值不正确。这是最普遍的情况，只要删除对应的主机记录就能恢复正常。
2. ssh 目录或者 .ssh/known_hosts 对当前用户的权限设置不正确。这种情况比较少，一般正确设置读写权限以后也能恢复正常。
3. /dev/tty 对 other 用户没有放开读写权限。这种情况极为罕见。出现的现象是，只有 root 用户能够使用 ssh client，而所有其他的普通用户都会出现错误。修改 /dev/tty 的权限后，一切正常。

