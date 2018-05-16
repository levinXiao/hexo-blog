---
title: UITableView 空数据或者错误时候的提示 empty Table
date: 2016-08-06 21:14:52
tags: iOS,oc,UITableView
categories: iOS
---
https://github.com/dzenbot/DZNEmptyDataSet

DZNEmptyDataSet是一个下拉式的UITableView/UICollectionView父类，在没有内容要显示时使用Empty DataSet模式。

<!-- more -->

大多数应用程序都会显示内容列表（datasets），但是某些情况下可能会是空的，尤其是新用户的账户信息。一旦产生错误或bug，空白屏幕会使用户困惑，不知道要做什么，所以Empty DataSet模式应该能够给用户提示相关信息。

Empty DataSet模式的好处：
避免白屏，告诉用户屏幕为何为空；
提示用户下一步的操作；
避免其它中断机制，比如显示错误提醒；
保持连续性，改善用户体验；
展示品牌标示。

功能：
可自定义背景颜色和视图以及垂直和水平对齐，多种布局和外观，支持点击手势和自动旋转，兼容UITableView和UICollectionView以及Storyboard。

你可以不使用UITableView或UICollectionView设计该库，可以使用UITableViewController或UICollectionViewController。只要符合DZNEmptyDataSetSource＆DZNEmptyDataSetDelegate，你就可以完全自定义你的应用程序的Empty DataSet的内容和外观。

代码下载地址：
https://github.com/dzenbot/DZNEmptyDataSet

更多详细内容可参考：
https://www.cocoacontrols.com/controls/dznemptydataset
