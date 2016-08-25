---
title: 解决gem安装Operation not permitted错误
date: 2016-08-10 14:23:11
tags: [macOS,gem]
---

简单来说,这个问题应该出现在10.11 El Captain上的几率比较大
原因呢 就是因为apple在El Captain引入了**rootless**机制

# rootless

###### 一句话总结
即使是root用户，将无法对以下路径有写和执行权限：

* /System
* /bin
* /sbin
* /usr (except /usr/local)

只有Apple自身签名的软件（含命令行工具）可以。

###### 详细说明：

先简单介绍一下OS X的权限系统吧

OS X 的内核XNU,一个由Mach，BSD，IOKit混合的内核，所以它的权限管理系统类似UNIX的BSD

1. 一个用户(user)对于一个文件有三种状态，可读(r)、可写(w)、可执行(x)，一个文件会有一个所属用户，所属用户组。还会有文件属主权限、同组用户权限、其他用户权限这三种标识符用来定义一个文件对一个用户的权限集合。

2. 为了达到多个用户的权限管理，每个用户会在一个或者多个用户组(group)中，一个用户组可以有多个用户

3. root用户是一个特殊用户（超级用户），拥有对所有文件的rwx权限（可读可写可执行）

4. UNIX系统是纯粹基于文件的，换言之你的网络读取，驱动程序，分区表什么的其实都会以文件的形式存在

###### 再复杂点

OS X 内置会有staff wheel admin这三种常见的用户组，分别介绍一下

1. staff : 所有创建用户都会属于这个用户组，提供最基本的对该用户目录 ~/*/User/(*))的rwx权限，其他的一般只有r或者rx权限。比如我建立一个用户叫做lizhuoli，那么我会在一个staff组里面，对/User/someone/* 有rwx权限。

2. admin :默认创建的所有用户也会属于这个admin用户组，在它里面的用户可以通过
su
或者
sudo
切换到root用户，只要执行以后输入这个用户的密码即可，而不需要知道root密码。

3. wheel :唯一只拥有一个用户root，意思是root用户的专属用户组。

OS X 10.11 加入了Rootless，默认创建的用户还是属于admin用户组，也能切换到root用户，但是加以了限制，结果是一旦你执行su 或者 sudo切换到root用户

**你的这个root用户不再是真正的root用户(对所有文件有rwx)**

**你的这个root用户不再是真正的root用户(对所有文件有rwx)**

**你的这个root用户不再是真正的root用户(对所有文件有rwx)**

重要的话说三遍！

```
bash-3.2# whoami
root
touch /usr/test
touch: test: Operation not permitted
```

现在你虽然用 whoami 看到自己明明是root用户了，但是不能有对/System /usr等文件夹的写入权限，甚至将一些原来的所属用户组为admin的文件夹的权限全部改为了wheel（比如对/usr/local，在10.11之前都是属于当前用户:admin用户组的，启用Rootless会变成root:wheel）。

这样防止了黑客入侵，病毒修改，一些恶意软件的篡改文件权限的行为。（不过话说Homebrew — The missing package manager for OS X应该也要更新了，因为Rootless下面Homebrew的安装路径/usr/local都是root:wheel的）

当然，你也可以禁用Rootless，不过这就多了一份被恶意软件攻击的潜在威胁。建议此时打开Mac App Store 和被认可的开发者或者纯粹的Mac App Store限制。


# 说正事
如果发生了上面的错误

告诉你就是因为rootless的原因

怎么解决

既然没有权限我换一个地方存着这个就好了

例如我要装openssl

```
#macOS下
$ sudo gem install –n /usr/local/bin openssl  --no-ri --no-rdoc
```

正事说完了