---
title: Xcode中SVN提交不了.a文件的解决方法
date: 2017-11-25 21:14:52
tags: svn,macOS,Xcode
categories: iOS
---
在Xcode中使用svn的时候 发现.a文件都提交不上去
原因是在svn中 .a文件是默认忽略的
知道这个原理后我们只要在svn的忽略文件中移除.a文件就可以了

输入vi ~/.subversion/config 打开svn配置文件
找到```# global-ignores = *.o *.lo *.la *.al .libs *.so *.so.[0-9]* *.a *.pyc *.pyo
 #*.rej *~ #*# .#* .*.swp .DS_Store```

 <!-- more -->
 
删除 ```*.a```这一串就可以了

当这样使用后发现还是不能够添加上去
所以你还需要手动添加追踪和手动提交

1.打开终端，输入cd，空格，然后将需要上传的.a文件所在的文件夹（不是.a文件）拖拽到终端（此办法无需输入繁琐的路径，快捷方便） ，回车；
2.之后再输入如下命令：svn add **********.a，回车；
3.之后会出现：A  (bin)  libOCMock.a
   表示添加成功刚才添加的.a文件，此时就可以手动上传了。
   另外，在mac 10.8中输入命令行，可能会提示你command not found，因为10.8默认没有安装Command line tools，解决办法：command not found解决方法。

4.输入 svn ci - m ""提交
