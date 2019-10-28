# win10搭建android阅读源码环境

## 一、下载源码

下载清华大学开源软件镜像站的AOSP：[https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar](https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar)
下载完成后用户管理员权限(必须管理权权限，否则无效)启动cmd：解压
>>start winrar x -y aosp-latest.tar D:\WorkSpace

## 二、安装repo

因为repo在win10下有很多问题，遂采用Ubuntu子系统来操作。
Ubuntu子系统直接在win10商店搜索安装即可
repo需要使用python2.7，Ubuntu子系统和Python具体安装过程和配置参考这篇文章[windows下Ubuntu子系统配置python](https://www.cnblogs.com/howxcheng/p/10497555.html)
Ubuntu是自带git的，无需另外安装

下载 repo 工具:

>>mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

## 三、同步

在Ubuntu下使用命令行进入解压源码的目录，执行repo sync同步最新源码
同步完成后，进到.repo/manifests目录中
git branch -a 查看所有分支
找到想看的分支，切换过去，比如 git checkout android-9.0.0_r1
如果切换失败，出现fatal bad revision 'head'错误，只需commit一次，错误就会消失：
>>git commit -m "initial commit"
切换好分支，回到解压根目录，执行 repo sync -l 即可检出对应分支的源码

## 四、放入编译文件

as配置文件git地址： [github.com/difcareer/A…](https://github.com/difcareer/AndroidSourceReader)
下载对应版本的编译文件放到项目的根目录即可

## 五、导入Android studio

使用Android studio打开下载下来的编译文件里的<font color=red>.ipr</font>文件即可

## 参考资料

[AndroidStudio不用编译，阅读Android源码](https://juejin.im/post/5bd5c42ce51d457a9b6c8387)
[Windows 10安装Repo](https://chy.mobi/tool-used/windows-10-install-repo.html)
[清华大学开源镜像软件站Aosp](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
[windows解压文件时，出错：不能创建符号链接...客户端没有所需取得特权](https://blog.csdn.net/y601500359/article/details/80418765)
[AOSP之REPO操作](https://www.jianshu.com/p/358ea9c1c644)
[windows下Unbuntu子系统配置python](https://www.cnblogs.com/howxcheng/p/10497555.html)