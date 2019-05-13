---
layout:     post
title:      学习工作环境：机器学习与知识管理平台
subtitle:   云服务器+Pylab+Github.io
date:       2018-06-05
author:     马骋
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 工作方法
---

---

# 引言

近期基本摸索出了一套高效的线上工作方法：

- 阿里云服务器**机器学习**环境
- 用github管理代码和项目
- 用github.io搭建博客 

如此，可以边工作，边记笔记、写文章，成果保存在github上，博客直接通过git发布到网上。

# 1. 机器学习环境

## 配置方法

自己搭建云服务器+JupyterLab，基本可以取代阿里云的DSW产品，并且具有更强的通用性、可配置。

- 服务器ECS选型，4核32G即可
**内存型 r5**即可满足需求，费用成本约500/月，与DSW差不多。
考虑长周期的使用强度并不大，内存16G也能基本满足要求。
- 本地cygwin中ssh到ecs服务器，在tmux中启动jupylab
```
ssh $ip
tmux 
jupyter lab --ip 127.0.0.1 --no-browser --port=8889 --allow-root
```
- 本地local映射到浏览器
```
ssh -N -L localhost:8889:localhost:8889 $ip
```
- 由于远程在tmux中启动，不用担心连接中断的问题。可以在不同终端上启动服务


其余操作的效果与dsw是一致的。

## 使用体验

优势：

- 通用服务器，网络连接无限制
    - 正常使用git，用github维护notebook仓库
    - 支持配置自动通知，可以更高效掌握进度，利用后台机器时间
- 可配置性强，各种软件包随便按照，可以与自己的常用配置对齐，支持更复杂的文件操作以及后台的运算
- 故障可控，不用总掉线，连接断开等问题
- 界面主题色可调整，terminal 有渲染，对眼睛舒适

劣势：

- 4核CPU，性能略差于DSW，可接受
实际测速，计算密集型的任务，速度差距在2倍作用
- 更多需要自己维护配置的地方，在不同的终端启动，先得配置ssh 登录
常用的终端不多，配置好就行。

总体上，使用ECS+jupyter Lab框架效率会更好。

## pylab高级功能

JupyterPylab是Notebook的高级版，主要的优势功能有：

- 强大的侧边栏目录树，方便项目文件的管理/切换
起到了文件管理系统的作用
![image.png](https://upload-images.jianshu.io/upload_images/845620-ddd797b75d22df04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 支持上传/下载，数据和程序的传输很方便，避免了繁琐的rysnc
![image.png](https://upload-images.jianshu.io/upload_images/845620-6431751c426293df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 强大的tab页面，可以在网页下开N个tab，并随意组合位置
支持多个页面交叉工作，notebook和terminal可以对应起来，例如一边run数据，一边看htop。
![image.png](https://upload-images.jianshu.io/upload_images/845620-79f0744425f2df70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- markdown预览功能，可以起到markdown编辑器的功能，如上。
- session管理，方便关闭不用的session，释放内存。
- multi-workspace，工作环境可以clone到新的页面，并行工作
例如，换了ssh登录，也可以进入同一个lab页面工作
![image.png](https://upload-images.jianshu.io/upload_images/845620-12ed4903f8dee35a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 2. Github代码管理

- 机器学习的代码，直接push到代码仓库
https://github.com/Frank2015a/machine_learning_practice
- 博客markdown发布到
https://github.com/Frank2015a/Frank2015a.github.io

将ECS服务器的ssh key配置到Github，免去密码验证。

# 3. github.io搭建博客 

github博客的最大优势是，**push即发布**，免去了简书等平台繁琐的发布过程，也避免了leannote等笔记工具未来不靠谱的风险。

感谢BYQiu分享的github.io环境，https://www.jianshu.com/p/e68fba58f75c 。
参照此教程配置搭建自己的博客页面：https://frank2015a.github.io/

![image.png](https://upload-images.jianshu.io/upload_images/845620-b54c10fc88d7fe47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

博客规划：

- github作为主要的博客，尤其发布技术方面的文章，有利于专业圈子的交流
- git博客的内容打磨成熟后，再直接搬运到简书
- 简书可以作为图床，链接可以在其他网站打开、显示
比七牛云之类的存储要方便很多。

