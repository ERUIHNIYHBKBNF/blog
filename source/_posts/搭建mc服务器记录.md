---
title: mc服务器搭建记录
date: 2021-09-27 09:06:17
tags: [杂项, 服务器, mc]
---

## 环境准备

### 云服务器相关

* 一台云服务器，可以去嫖各大厂商的学生服务器，顺带装了操作系统ubuntu。
  * 顺带进防火墙把25565（一般习惯都是这个数字qwq）端口打开。
* ftp工具，与服务器传输文件用，随便找的FileZilla。
* SSH工具，远程连接服务器，随便找的PuTTY。

### java环境配置

官网下载：https://www.oracle.com/java/technologies/downloads/

下载一个对应版本的压缩包，一般好像都叫 jdk-xxxx-linux-x64.tar.gz

顺带最新版mc服务器对jdk版本要求好像很高来着qwq

顺带低版本mc服务器不能拿太高版本的jdk跑来着qwq

进去解压，记住路径。

* cd到 `/root` 或者 `home/admin`，看用哪个用户了。
* 文本编辑器打开 `.bashrc` 文件，后面添加几行改成自己的路径

```text
export JAVA_HOME=/usr/local/java/jdk-8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

* 然后 `java -version` 试一下就好了唔，，

## 启动纯净服

> 其实就是个jar包没什么特别的qwq

官网下载： https://www.minecraft.net/zh-hans/download/server

找个地方扔服务器上，跑一下（听说Server端最好把Xmx和Xms设成同一个值）

```shell
java -Xmx1024M -Xms1024M -jar minecraft_server.1.17.1.jar nogui
```

启动失败，然后ls一下，发现多了个 `eula.txt`， 进去把之后一行改成 `eula=true` 。

然后再启动，进mc直接连接 `服务器ip:25565` 应该就可以连上。

关于 ` server.properties` 的一些配置文档： https://minecraft.fandom.com/zh/wiki/Server.properties，改了重启服务器就好了的样子。

关于作弊模式，在服务器启动的时候终端输入 `op 游戏用户名` 就能把该用户设成管理员了。

## 支持mod的服务器

这边用了mcforge，下载对应版本安装（或者直接下载universal再扔个libraries过去）：

https://files.minecraftforge.net/net/minecraftforge/forge/

装好之后跟纯净服一样启动，会多出一个mods文件夹，把需要的mod放进去就好了。

顺带有些mod要求客户端也要有的，本地安装forge，windows的话进入 `%appdata%/.minecraft/mods` 把mod放进去什么的。

## 长期运行

遇到一个问题是，玩着玩着服务器突然断掉了QAQ

看了半天log，怎么看都是服务器正常结束，咱可没手动关掉过这个\_(:з」∠)\_

讲个笑话，咱都不知道"挂起"是什么概念qwq

被大佬嘲讽了呜呜呜..

跑jar的时候加个 `nohup` 就可以了qwq

```shell
nohup java -jar -Xmx1700m -Xms1700m forge....
```

做个记录，溜了溜了QAQ
