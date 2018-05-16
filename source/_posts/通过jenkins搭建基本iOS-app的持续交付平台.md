---
title: 通过jenkins搭建基本iOS-app的持续交付平台
date: 2016-08-10 16:10:49
tags: [macOS,iOS]
---

平时我们开发完成IOS项目，需要打包给测试人员进行测试。其中的过程需要重复进行：修改配置项--编译---连接设备--运行打包--debug进设备中--然后交给等待的测试人员。现有成熟的持续集成Jenkins解决方案，并且该方案也提供了Xcode插件的支持，可以讲上述过程封装成一键解决方案。

<!-- more -->

# 安装jenkins
##	java环境准备
jenkins主要依赖的java环境 所以确保电脑中java环境准确就绪支撑软件
检测是否已经部署了java环境
在终端输入

```
$ java -version
```

如果出现类似于如下的字样 则标识您已经安装了java环境

```
$ java version "1.8.0_77"
```
若出现如下

```
$ -bash: java: command not found
```

则表示没有安装java环境 则需要下载java安装
下载地址 (1.8版本,后续下载版本下载地址可能会有变化)
[java 1.8下载地址](http://sdlc-esd.oracle.com/ESD6/JSCDL/jdk/8u101-b13/jre-8u101-macosx-x64.dmg?GroupName=JSC&FilePath=/ESD6/JSCDL/jdk/8u101-b13/jre-8u101-macosx-x64.dmg&BHost=javadl.sun.com&File=jre-8u101-macosx-x64.dmg&AuthParam=1469437384_c779c63859c8f9e7a246e00f1e58a803&ext=.dmg)

下载安装后重复上述步骤 确保java环境成功搭建

##	在mac上安装jenkins
###	安装HomeBrew
jenkins依赖HomeBrew包管理,所以我们要先安装homebrew,
若已安装则跳转,查看方法

```
$ brew -v
```
若正确显示版本信息则代表安装成功
安装homebrew参考网站
```
http://brew.sh/index_zh-cn.html
```
安装完成之后
homebrew的源在国外,所以我们需要配置一下使之下载的快一点
参考网站 http://ban.ninja
在终端中输入  

```
$ vim ~/.bashrc
```
在文件最后一行 添加七牛cdn加速

```
$ export HOMEBREW_BOTTLE_DOMAIN=http://7xkcej.dl1.z0.glb.clouddn.com
```
注:以上工作都是前期准备,因为大部分包管理(包括brew和gem)的源都在国外,所以可能下载速度会很感人
###	开始安装jenkins
在终端输入
```
$ brew install jenkins
```
根据提示操作即可完成 大概需要不到5分钟(排除网络原因)
至此:jenkins环境搭建成功

#	安装fastlane和fir-CLI
## fastlane和fir-CLI工具介绍
fastlane和fir-CLI是一组工具套件,旨在实现iOS应用发布的自动化,并且提供一个良好的持续集成和部署流程,只需要一个点击或者一个命令就可以触发这个流程
项目地址

[fastlane(github)](https://github.com/fastlane/fastlane)

[fir-CLI(github)](https://github.com/FIRHQ/fir-cli/)
shenzhen已经由一年以上没有更新了,很多功能都并不是很好用了,所以在这里我们才会安装fastlane这个工具,截止到2016年07月,fastlane工具还是可以上传到itc的

##	rubygems环境搭建
fastlane 和 shenzhen都是ruby编写的 所以要用到gem安装
查看当前的ruby版本

```
$ ruby –v
```

代表ruby gem安装成功已经安装
若没有安装rubygems 可以利用我们刚刚搭建好的brew包管理来安装gem
```
$ brew install ruby
```

注:gems其实是ruby的包管理,ruby自1.9.2以后已经安装gems
gem安装完成后我们还要修改ruby的下载源到国内
参考网站[ruby-china](http://gems.ruby-china.org/)(只需要了解到该网站下的`如何使用`就可以了)
##	安装fastlane
安装fastlane和fir-cli的前提是确保已经安装了xcode-tool-chain工具链,如果本地没有安装这个工具链,需要下载并安装,最简单的办法就是下载xcode

若已经安装了这个工具链(一般本地安装了xcode软件,会自动安装这个工具链的),键入如下命令
```
$ sudo gem install fastlane
```
可能需要输入密码,密码为现在mac登入账户下的登录密码

如果出现如下错误
```
while executing gem ... (Errno::EPERM)
```
键入

```
$ export GEM_HOME=~/.ruby;sudo nvram boot-args="rootless=0"; sudo reboot
```

等待重启完成后继续安装fastlane


```
$ sudo gem install –n /usr/local/bin fastlane  --no-ri --no-rdoc

$ sudo gem install –n /usr/local/bin fir-cli  --no-ri --no-rdoc
```

#	为项目初始化fastlane
##	检查初始化
查看项目code文件夹下,与xcodeproj同级目录下是否存在fastlane文件夹
如果存在,可以跳过这一步,直接进行jenkins设置和环境搭建
如果不存在,则要为项目初始化fastlane了
##	开始
在终端中cd 到项目所在的文件夹下,注意:与xcodeproj文件同级)
执行下面的命令

```
$ fastlane init
```
按照提示完成输入
1.	初始化输入appleid(该appleid 必须是对应着该项目的,而且在本地xcode中已经登录并存在与钥匙串中,并确保证书已经下载)

2.核对信息下载文件
fastlane初始化结束后,会在本地生成一个名为fastlane的文件夹
文件夹内包含三个文件和两个额外的文件夹


```
Appfile
Deliverfile
Fastfile
metadata
		copyright.txt
		primary_category.txt
		primary_first_sub_category.txt
		primary_second_sub_category.txt
		secondary_category.txt
		secondary_first_sub_category.txt
		secondary_second_sub_category.txt
		zh-Hans
				description.txt
				keywords.txt
				marketing_url.txt
				name.txt
				privacy_url.txt
				release_notes.txt
				support_url.txt
screenshots
```

这其中最重要的也是和上传相关最大关联的文件就是**Fastfile**和**metadata**文件夹中的**zh-Hans**文件夹中的内容
其中:
**description.txt**:描述这个app的文本文档

**keywords.txt**:appstore ASO(搜索关键字)

**marketing_url.txt** :销售url

**name.txt**: 应用名称,一般不会修改

**release_notes.txt**: 应用更新内容, 一般在发布版本的时候都会修改

到现在为止,所有的准备工作已经就绪,现在我们就要进入jenkins的后台去进行持续集成了.
注意:fastlane的init工作一般只进行一次,如果发现存在fastlane文件夹,就不要进行这个工作了,而且这个文件会随着svn或者git一起上传上去
我已经减少了不必要下载的文件和样例,只保留了两个文件 fastlane/metadata/zh-Hans/下的description.txt和release_notes.txt 因为在我看来,这个文件才是有可能经常会改动的,其他的如果想要修改,可以去 https://itunesconnect.apple.com 上登录去修改

#	jenkins的配置和自动打包脚本的编写
##	启动jenkins
在终端输入
```
$ jenkins –h
```
输出结果:

如果看到最后一行  **Jenkins is fully up and running**  则代表jenkins启动成功

在浏览器(safiri或chrome中输入地址 127.0.0.1:8080),默认jenkins的启动端口是8080

如果是第一次进入到后台,会有一些基本的配置需要来完成.按照提示来就可以了
到最后一步,会让我们去安装插件,如果网络环境不好,可能会出现安装失败的情况,但是不会影响主要功能的使用
##	jenkins界面

如果所有的配置都是在正确的,那么你会看到主界面

顾名思义:


|  |  |
:----------- | :-----------
|**新建** 	|	可以新建一个持续集成的任务|
|**用户** 	|		管理用户|
|**任务历史** |		可以查看持续集成的历史|
|**系统管理** |		类似于系统设置,一般我们使用的较多的功能则是插件管理.|
|**Credentials**| 管理所有的通行证,可以是git的也可以是svn|

构建view界面

构建视图view可以查看当前的构建的队列和构建任务的进行情况

##	搭建基于jenkins的iOS_mvs_beta项目
###	项目目的
建立该项目的目的主要是为了一键能够将iOS应用`优服365`能够快速方便的提供给测试人员测试,并且能够将项目一键打包成三个版本(dev,qa,prod)和分发给测试人员测试
###	新建任务
回到主面板,点击新建

###	输入信息
名称我们定为iOS_mvs_beta
紧接着选择  构建一个只有风格的软件项目  最后点击ok


###	源码管理
随后进入一个新的页面 找到源码管理


如果发现源码管理中没有找到你想要的  可能是没有安装对应的插件,安装插件的过程如下
主界面系统管理管理插件可选插件找到 `GIT plugin`或者`Subversion Plug-in`勾选后点击最下面的按钮直接安装安装完后重启即可


可能会由于网络的原因,安装失败或根本不能访问到jenkins-ci.org
提供一个subversion.hdi的国内私有下载地址,如果失效,请另行想办法下载
```
$ http://7xrlqi.com1.z0.glb.clouddn.com/subversion.hpi
```
下载完成后,上传到jenkins安转即可
主界面系统管理管理插件高级上传插件选择文件后上传完成后重启

插件安装完成后,进入到项目配置页面,找到源码管理
这个项目我们使用的是svn 输入svn的地址和用户名密码

###	添加构建命令
找到 `构建` 这一栏点击`增加构建步骤` 点击 `Execute shell`

输入如下命令
```
#1.usr/bin/env bash
source ~/.bash_profile

#发布qa环境
en="qa"
#打包文件存放的目录
di="${HOME}/jenkins/app/${JOB_NAME}/${en}/"
#打包的文件的名称
name="mvs_${en}-${BUILD_NUMBER}-`date \"+%m月%d日%H时%M分\"`.ipa"
#ci环境变量路径
cienvpath="${WORKSPACE}/MVS/ci.env"
#修改ci.env的值
echo "{\"env\":\"${en}\"}">$cienvpath
#修改应用名称
sed -i '' -e "s/优服365/优服365_${en}/g" ${WORKSPACE}/MVS/Info.plist
mkdir -p $di
#打包

#编译环境
scheme="MVS"
configuration="Debug"
export_method='ad-hoc'
#指定项目地址
in_path="${WORKSPACE}/MVS.xcodeproj"
#指定输出归档文件地址
archive_path="$di/archive/mvs_${en}-${BUILD_NUMBER}-`date \"+%m月%d日%H时%M分\"`.xcarchive"
#先清空前一次build在发布
gym --project ${in_path} --scheme ${scheme} --clean --configuration ${configuration} --archive_path ${archive_path} --export_method ${export_method} --output_directory ${di} --output_name ${name}

#ipa build -c Release -d $di --ipa $name
#发布到蒲公英
curlfileurl="@${di}${name}"
curl -F "file=${curlfileurl}" -F "uKey=5bffb7613006a47ea75bbc1b05720c16" -F "_api_key=8b461209f404caa7121e1374627f004b" http://www.pgyer.com/apiv1/app/upload

#还原名称
sed -i '' -e "s/优服365_${en}/优服365/g" ${WORKSPACE}/MVS/Info.plist


#发布production环境 ali环境(测试用) 不是发布到App Store
en="prod"
di="${HOME}/jenkins/app/${JOB_NAME}/${en}/"
name="mvs_${en}-${BUILD_NUMBER}-`date \"+%m月%d日%H时%M分\"`.ipa"
echo "{\"env\":\"${en}\"}">$cienvpath
sed -i '' -e "s/优服365/优服365_${en}/g" ${WORKSPACE}/MVS/Info.plist
mkdir -p $di
#打包
#编译环境
scheme="MVS"
configuration="Debug"
export_method='ad-hoc'
#指定项目地址
in_path="${WORKSPACE}/MVS.xcodeproj"
#指定输出归档文件地址
archive_path="$di/archive/mvs_${en}-${BUILD_NUMBER}-`date \"+%m月%d日%H时%M分\"`.xcarchive"
#先清空前一次build在发布
gym --project ${in_path} --scheme ${scheme} --clean --configuration ${configuration} --archive_path ${archive_path} --export_method ${export_method} --output_directory ${di} --output_name ${name}

#发布到testin
curlfileurl="@${di}${name}"
#上传到testin
curl -F "file=${curlfileurl}" -F "user_key=d2a20900910cf09b4dd7bf039d72a7eb" -F "update_notify=1" http://api.pre.im/api/v1/app/upload


#还原名称
sed -i '' -e "s/优服365_${en}/优服365/g" ${WORKSPACE}/MVS/Info.plist

#发布dev环境
en="dev"
di="${HOME}/jenkins/app/${JOB_NAME}/${en}/"
name="mvs_${en}-${BUILD_NUMBER}-`date \"+%m月%d日%H时%M分\"`.ipa"
echo "{\"env\":\"${en}\"}">$cienvpath
sed -i '' -e "s/优服365/优服365_${en}/g" ${WORKSPACE}/MVS/Info.plist
mkdir -p $di
#发布到fir.im 需要安装gem插件 fir-cli #gem install fir-cli
fir upgrade
fir build_ipa ${WORKSPACE} -o $di$name -p -T 99a2677b35d3e1b1037791a8e6c3034a
```


点击保存
###	开始第一次构建
现在我们回到主面板,发现已经多了一个名为  `iOS_mvs_qa ` 的项目,点击进入后 点击立即构建开始第一次构建

#	配置一键上传iTunesConnect
##	项目目的
建立该项目的目的主要是为了一键能够将iOS应用`优服365`能够快速方便的上传到苹果iTunesConnect的服务器,节省开发人员和上传人员的时间
##	建立任务
如4.3.1所示,新建一个名为iOS_mvs_appstore的任务
svn地址和描述都和上一个一样

##	编写脚本
如4.3.4所示:脚本更改内容如下
```
#1.usr/bin/env bash
 cd ${WORKSPACE}
chmod +x upload2Appstore.sh
./ upload2Appstore.sh
```

upload2Appstore.sh 这个文件是随着svn一起down下来的
附: upload2Appstore.sh 源码
```
#1.usr/bin/env bash

cienvpath="MVS/ci.env"
#修改ci.env的值
echo "{\"env\":\"production\"}">$cienvpath

rm -f MVS.ipa
rm -f MVS.app.dSYM.zip

fastlane ios appstore

```
