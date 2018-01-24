---
layout: post
title: iOS自动化打包分发
tag: [iOS, 自动化]https://manajay
date: 2017-05-18 22:44:46 +09:00
---


### 为什么要自动化
* 节省时间,快速迭代: 减少重复繁琐的过程,本地继续编码,使用工具自动拉取远程库代码后打包
* 纠错: 打包出错,会自动查找到编译错误
* 快速分发多个版本: 配置好不同分支的打包策略,可以将打包任务移交测试,分工更明确

### 工具
* Jenkins CI
* Flow CI 
* Travis CI
* Hudson CI
* Circle CI

本文只关注 Jenkins 的使用 

#### Jenkins 的三种安装方式
* **war** 
* **pkg** 
* **homebrew**

推荐使用 `homebrew`
#### jenkins 安装
1、[正确安装Homebrew的方式](https://manajay.com/2017/01/homebrew-clean-install/)
2、安装:  `brew install jenkins` 
3、终端启动命令: `jenkins` 
4、浏览器访问`jenkins`地址: ``` http://localhost:8080/ ``` , 如果不能正常访问,要么**Java**环境出问题,要么`jenkins`没有启动; Java环境的去官网下载最新的**jdk**安装;[jenkins开关命令](http://damien.co/general/how-to-start-stop-restart-or-reload-jenkins-mac-osx-8022)
5、 正常的浏览器启动页面是如下的 

![浏览器启动页面](/assets/post/unlock-jenkins.jpg)

根据提示修改文件夹权限,获取密钥,登录**jenkins**
6、安装插件 ,**Jenkins**功能很多以来相应的插件,简化了接入的难度
> 最好先跳过这一步,因为电脑环境问题,有些插件是需要翻墙安装,所以导致jenkins安装插件的时候,会卡在一个地方,然后就一直卡着,我安装过多次jenkins,因为这个问题,__我习惯于跳过插件安装,先登录配置好环境,然后再手动安装插件__!!!!!!!!!

![install plugins](/assets/post/custom-jenkins.png)

如果出现一只卡死在安装插件的界面,关机重启,重新启动**Jenkins**后,登录`http://localhost:8080` 进入管理员注册页面
7、 管理员注册
要牢记这个名称,如果是自己测试用,直接用 `admin admin` 这种更简单的组合.
![管理员注册](/assets/post/create-jenkins-admin.png)

8、 进入首页,首先将需要安装的插件 再次补充全
![补全插件](/assets/post/fix-jenkins-plugins.png)

需要的插件如下
 * `Git` , `Gitlab`,`SVN` , `SSH Credentials`用于授权后拉取远程库的代码
 * `Keychains and Provisioning Profiles Management`: 证书与描述文件的管理
 * `Xcode integration` Xcode打包的插件,所以iOS的打包只能部署在Mac系统
 * `Cocoapods` 如果项目使用了cocoapod插件 来获取依赖库
 * `Mailer Plugin` 用来发送通知邮件
 * `fir-plugin` 用来将`ipa`包分发到 `fir.im`上面,或者使用蒲公英(只能是脚本)也可以
 * `Post-Build Script Plug-in` 脚本插件 
 
9、配置项目的访问**ssh**私钥 

![SSH-private-key](/assets/post/jenkins-configuration.png)

根据图上的路径 

![SSH](/assets/post/jenkins-ssh.png)


添加`SSH`的私钥, 一般你项目的访问私钥是 `~/.ssh/id_rsa` 这个文件,如果没有配置,则询问你的源代码的管理员

![id_rsa](/assets/post/jenkins-ssh-key.png)

如果私钥是错误的,则配置项目的时候会出现下面👇的错误

![wrong private key](/assets/post/jenkins-git-ssh.png)

所以务必要明白,项目的私钥是如何配置的. 需不需要口令码!!!
如果是公司项目,询问运维,当然一般运维会搭建jenkins(iOS必须要在Mac电脑上面搭建,如果是给JAVA使用,一般用linux,不能打包iOS).
自己的项目,不论是在`Coding,Github,Gitlab` 都可以在页面上查找对应的SSH添加,以明确将要在`jenkins`使用的`ssh`私钥是什么

10、配置项目依赖的证书与描述文件

![login key](/assets/post/jenkins-login-key.png)

进入后的界面是

![upload login-keychains](/assets/post/jenkins-ios-keychains.png)

主要有两个步骤,
①是 上传钥匙串的 `login.keychain` , mac地址```~/Library/Keychains/```

![login-keychains location](/assets/post/login-keychains-location.png)


② 设置参数

![login-keychains-password](/assets/post/login-keychains-password.png)

**注意** 证书的名称就是本机钥匙串,安装后的证书简介的 **常用名称**

![common name](/assets/post/ios-p12-name.png)

描述文件的地址 一般是 ```~/Library/MobileDevice/Provisioning Profiles```
不过我多次尝试 发现配置项目的时候 并没有得到描述文件,后面只能用脚本自己打包的

11、创建新的项目
![jenkins-new-job](/assets/post/jenkins-new-job.png)
选择项目的类型
![jenkins-new-job-setting](/assets/post/jenkins-new-job-setting.png)

进入项目的配置页面

![configuration](/assets/post/jenkins-job-config.png)

![all setting items](/assets/post/jenkins-item-config.png)

丢弃旧的构建 ,可以自己定义策略

![drop old builds](/assets/post/jenkins-build-settings.png)

设置 源码的 拉取, 这一步 主要是可能卡在 私钥的配置上面,所以一定要明确SSH的配置(见上面的说明)

![fetch origin code](/assets/post/jenkins-job-git-config.png)

构建触发器 , 这里主要是 定时去 自动化打包项目

![triger](/assets/post/jenkins-job-build-time.png)


构建环境, 主要配置的是 证书与描述文件

![keychain](/assets/post/jenkins-keychains-codesign.png)

下面是正确的环境[链接在此](http://www.jianshu.com/p/3b43776ed73f),我不知道是不是xcode8之后才有的这问题,还是我使用`homebrew`确实获取不到. 

![others keychains](/assets/post/jenkins-build-job-setting-config.png)

Xcode的配置 

![Xcode](/assets/post/jenkins-codesign-keychains.png)

具体配置

![Xcode build settings](/assets/post/jenkins-ios-job-general-settings.png)

钥匙串 选择配置好的钥匙串

![Xcode code sign](/assets/post/jenkins-keychain-ios.png)

其他的编译打包参数 , 如果使用了cocoapods还需要指定具体的一些参数,并且执行脚本,拉取依赖的远程库

![Xcode Project setting](/assets/post/jenkins-ios-archive-config.png)

最最重要的来了,打包的脚本

![build shell](/assets/post/jenkins-job-shell.png)
https://manajay.com
如果使用了 `cocoapods` 则需要提供拉取依赖库的代码 否则请忽略这一步(比如我们叮叮暂时没有)
分别是 指定这是一个脚本(截图有问题,应该是`#bin/bash -l`), `podfile`文件的 中文格式编码, 切换到`podfile`的路径下,拉取依赖的`pod`
![pod shell](/assets/post/jenkins-ios-job-build-shell.png)

![archive build](/assets/post/jenkins-job-ios-shell.png)

具体的代码如下

脚本①  样本 `xcodebuild -workspace "demo.xcworkspace" -sdk iphoneos -scheme "targetName" -configuration 'Release Adhoc' CODE_SIGN_IDENTITY="keychain中证书代号名称" SYMROOT='$(PWD)`  这里 `-workspace "demo.xcworkspace"`是使用`cocoapods`的样本, 没有使用就像下面的 `-project "project的路径"`

```
if [ -d "${WORKSPACE}/builds" ]; then rm -rf ${WORKSPACE}/builds; fi;
mkdir ${WORKSPACE}/builds;
if [ -d "${WORKSPACE}/builds/${BUILD_NUMBER}" ]; then rm -rf ${WORKSPACE}/builds/${BUILD_NUMBER}; fi;
mkdir ${WORKSPACE}/builds/${BUILD_NUMBER};
xcodebuild -project ${WORKSPACE}/ProjectName/ProjectName.xcodeproj 
-scheme "ProjectName" 
-sdk iphoneos 
archive -archivePath ${WORKSPACE}/builds/${BUILD_NUMBER}/archive 
CODE_SIGN_IDENTITY="iPhone Developer: xxxx"

```

注意 scheme 一定还要勾选 分享 

![scheme of project](/assets/post/ios-scheme.png)

![share scheme](/assets/post/jenkins-ios-archive-share.png)

并且注意 不同的打包`method` 要对应好. 

脚本②  [xcodebuild官方文档](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html)

```
xcodebuild -exportArchive 
-archivePath ${WORKSPACE}/builds/${BUILD_NUMBER}/archive.xcarchive 
-exportOptionsPlist ${WORKSPACE}/ProjectName/ProjectName/ExportOptions_development.plist 
-exportPath ${WORKSPACE}/builds/${BUILD_NUMBER}/${JOB_NAME}_${BUILD_NUMBER}.ipa 
PROVISIONING_PROFILE="iPhone Developer: xxxx"
```

这里有一个注意点 就是 `exportOptionsPlist` ,需要自己在项目中配置 相应的信息 

![exportOptionsPlist](/assets/post/jenkins-ios-exportplist.png)

一切顺利就可以正常打包了 
后面就是打包后 上传到 fir.im或者是蒲公英 给测试团队 .

![fir.im plugin](/assets/post/jenkins-fir.im.png)

还有就是 邮件通知 

![email plugin](/assets/post/jenkins-email.png)

12、 正常使用

![build by hand](/assets/post/jenkins-build.png)

点击进入控制台输出,查看运行的细节

![log](/assets/post/jenkins-log.png)

#### 团队的使用
1、 设置一个 局域网的固定访问地址

![custom jenkins url](/assets/post/jenkins-url.png)

公司内其他同事就可以通过这个地址,访问`jenkins` 自己去配置项目,进行打包
2、 设置jenkins运行电脑的安装工作目录 为 分享目录

![share files](/assets/post/mac-share-file-location.png)

![configurate share files](/assets/post/mac-share-01.png)

![jenkins' workspace](/assets/post/mac-share.png)

同事可以通过**Finder** 访问共享的电脑,找到`manajay`名称的电脑,选择连接,可以设置密码,我直接让同事可以以客人的身份,无密码访问`jenkins`下的 `workspace`目录.
这样即使 上传失败,自己可以获取到ipa包,自己分发.
3、 jenkins 设置成,开机自启的程序 [brewed-jenkins小插件](https://github.com/fastlane/brewed-jenkins)
![brewed-jenkins](/assets/post/jenkins-start-with-mac.png)

#### 补充另一种打包方式 fastlane
如果你的项目使用的是 fastlane那就简单很多了. 
* match 配置证书与描述文件
* gym 负责打包
* pgyer 负责分发测试, fastlane有个蒲公英的插件,亲测有bug,所以我还是用的脚本

整个打包上传脚本,注意如果使用了fastlane就不需要配置证书的钥匙串,上面的xbuild脚本了.

```
#bin/bash -l
pwd
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
export LC_ALL=en_US.UTF-8

echo "pod 更新项目依赖 ----开始----"
/usr/local/bin/pod update --verbose --no-repo-update
echo "pod 更新项目依赖 ----结束----"

cd fastlane
pwd
echo "fastlane 打包命令-----开始-----"
fastlane development
echo "fastlane 打包命令-----结束-----"

echo "pyger 上传命令-----开始----"
sh ./deploy_pgyer.sh
echo "pyger 上传命令-----结束----"
```

#### 待完成的
* 测试蒲公英分发 `curl -F "file=@/tmp/example.ipa" -F "uKey=" -F "_api_key=" https://qiniu-storage.pgyer.com/apiv1/app/upload
`
* 添加`webhooc`自动触发构建

### 参考链接

* [iOS持续集成:Jenkins+GitLab+蒲公英](http://www.jianshu.com/p/3b43776ed73f)

* [史上最全Jenkins+SVN+iOS+cocoapods环境搭建及其错误汇总](http://www.jianshu.com/p/7a2efc7c69fe)

* [使用Jenkins搭建iOS/Android持续集成打包平台](http://debugtalk.com/post/iOS-Android-Packing-with-Jenkins/)

* [关于持续集成打包平台的Jenkins配置和构建脚本实现细节](http://debugtalk.com/post/iOS-Android-Packing-with-Jenkins-details/)

* [牛人总结的自动化打包脚本](https://github.com/debugtalk/JenkinsTemplateForApp)

* [webhooc自动触发打包](http://www.jianshu.com/p/ad018160aff9)

* [iOS 持续集成系列 - 自动化 Code Review](http://www.jianshu.com/p/c8b3b515ccf3)

* [丢弃xcodebuild,用fastlane打包](http://www.jianshu.com/p/0b22d6a8969d)

* [蒲公英上传app](https://www.pgyer.com/doc/api#uploadApp)

* [fir.im Jenkins 插件使用方法](http://blog.fir.im/jenkins/)

* [flow.ci iOS 证书设置指南](http://blog.flow.ci/ios-certificate-settings/)

* [iOS 项目 Build 失败的常见原因](http://blog.flow.ci/ios-build-failure-reasons/)

* [iOS自动构建以及打包命令](http://blog.csdn.net/skylin19840101/article/details/54406080)

* [记录一次 xcodebuild 无法生成 dSYM 文件 的解决步骤](http://www.jianshu.com/p/7a79a6ad5df4)

* [iOS构建自动化打包脚本](http://gcblog.github.io/2016/11/21/iOS%E6%9E%84%E5%BB%BA%E8%87%AA%E5%8A%A8%E5%8C%96%E6%89%93%E5%8C%85%E8%84%9A%E6%9C%AC/)



