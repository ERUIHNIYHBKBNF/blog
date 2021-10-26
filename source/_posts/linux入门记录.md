---
title: "linux入门记录"
date: 2021-10-26 16:43:22
tags: [linux]
---

# 记录双系统装好ubuntu之后做了什么

打算一点一点地把工作环境迁移到linux上，在此记录过程，Ubuntu20.04。

纯linux萌新，对linux的了解仅限于搭过mc服务器的程度，欢迎围观咱蠢蠢的操作，嘤～

## 换源

网上搜换源出来一堆看不懂的命令QAQ

好像在 `软件和更新` 这个应用程序里直接找就好了，按同学说换了utsc的镜像站。

## 体验感优化

轻薄本小屏幕看着字太小好难受，想办法先放大一点叭 (>\_<)

```shell
sudo apt install gnome-tweaks
```

之后就是有图形界面的一些设定了，功能好丰富的样子QwQ

大概知道了这些东西：

apt是个包管理器，之前用过npm感觉；比较类似..?

`sudo` 是在最高权限下执行指令，用 `su` 直接切换到管理员模式。

## 更换浏览器

老chrome用户，可能用久了chrome就不太习惯firefox之类的了。

拿到chrome的安装包，好像是 `.deb` 结尾的，cd到那个位置就好了诶

```shell
sudo apt install google-chrome***.deb
```

之后的设定还挺智能的样子（不用担心东西被装到C盘真舒服owo）

大概知道了这些东西：

`.deb` 是某种安装包的样子，惊奇地发现apt还可以安装离线包（傻～）

## 科学

> 没有科学还怎么网上冲浪qwq

试来试去找到最舒服的一篇教程： [Linux下安装&配置Clash以实现代理上网](https://zhuanlan.zhihu.com/p/369344633)

### 安装clash

[Github地址](https://github.com/Dreamacro/clash/releases) 一般是下载 clash-linux-amd64.tar.gz

```bash
su
gunzip ***.gz
chmod +x clash***
./clash***
```

会自动在 `~/.config/clash/config.yaml` 中生成 `config.yaml` 文件。

导入配置：

```bash
cd ~/.config/clash/
curl [订阅连接] >> config.yaml #替换掉原来的文件
#另一种姿势
wget -P ~/.config/clash/ -O config.yaml [订阅链接]
```

打开系统设置 -> 网络 -> 网络代理，改为手动：

```text
HTTP代理 127.0.0.1 7890
HTTPS代理 127.0.0.1 7890
FTP代理 空
Socks主机 127.0.0.1 7891
忽略主机 localhost,127.0.0.0/8,::1
```

再运行clash，访问 [https://clash.razord.top/](https://clash.razord.top/)，测试连接即可。

### 配置开机启动与自动更新订阅

开机启动：

习惯性地把clash的二进制文件移动到 `/opt/clash/` 目录下并重命名为 `clash` ：

```bash
mv ./clash*** /opt/clash/clash
```

创建service文件：

```bash
vim /etc/systemd/system/clash.service
# 写入以下内容
[Unit]
Description=clash daemon

[Service]
Type=simple
User=root
ExecStart=/opt/clash/clash -d /opt/clash/
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload #重新加载systemctl deamon
systemctl start clash.sevice #启动clash
systemctl enable clash.service #开机启动
```

重启测试没什么问题。

自动更新订阅：

利用cron执行计划任务：

```bash
crontab -e#写入以下内容，前面的时间是 分 时29 21 * * * pgrep clash | xargs kill -s 9 30 21 * * * mv ~/.config/clash/config.yaml ~/.config/clash/configbackup.yaml 31 21 * * * wget -P ~/.config/clash/ -O config.yaml [订阅链接]32 21 * * * nohup /opt/clash/clash -d /opt/clash/
```

```bash
systemctl restart crond.service #重启crontab，有的地方也叫cron.service
```

大概知道了这些东西：

`~` 大概是当前用户的根目录，里面有好多配置文件qwq

systemd（不懂QAQ），总之好像可以用来做很多东西。

crontab可以用来配置定时任务。

## 网易云等QT应用缩放问题

有linux版就很舒服，安装之后遇到了些小问题...非常小的那种..指字体非常小（躺）

搜到了这个：

> Linux下的高分屏在Gnome、KDE中有缩放因子一说，但是对QT程序（常用如 WPS、网易云音乐等）无效，这里只是简记修改QT程序的缩放方法。

解决方法：

```bash
vim /etc/profile#在最后添加，测试一波感觉2.25倍看上去还算正常的样子qwqexport QT_SCALE_FACTOR=2.25#不用vim的另一种姿势echo "export QT_SCALE_FACTOR=2.25" >> /etc/profile#保存退出注销重新登录
```

## 配置Github免密SSH连接

基本根Windows一样叭，但是弄了两次，因为root跟普通用户好像不在一个地方存SSH-Key

```bash
#配置本地用户信息git config --global user.name "your nickName“git config --global user.email "your email"#生成公钥ssh-keygen -t rsa#查看公钥，这里不同用户好像要重新弄一边cat ~/.ssh/id_rsa.pub
```

之后在Github的Setting里边添加这个公钥就好啦QwQ

## 使用nvm安装管理多版本nodejs

[菜鸟教程](https://www.runoob.com/w3cnote/nvm-manager-node-versions.html)

### 安装与配置nvm

Github仓库地址：[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

按仓库的步骤来就好了唔：

```bash
cd ~/git clone https://github.com/nvm-sh/nvm.git .nvmvim ~/.bashrc#添加环境变量export NVM_DIR="$HOME/.nvm"[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm#保存退出，重载环境变量source ~/.bashrcnvm -v #就好像没什么问题了
```

### 管理node版本

常用命令：

```bash
nvm list #查看已安装版本nvm ls-remote --lts #查看远程lts版本nvm install 12.21 #安装v12.21的nodenvm use 12.21 #激活版本
```

## 安装多版本mysql

### 单版本安装配置

先装一个8.0版本：

```bash
sudo apt updatesudo apt install mysql-serversudo systemctl status mysql #查看服务是否在运行sudo mysql #竟然可以不输密码直接进qwq
```

查看初始随机账户密码：

```bash
sudo cat /etc/mysql/debian.cnf
```

还是改个root密码叭：

> MySql 从8.0开始修改密码有了变化，在user表加了字段authentication_string，修改密码前先检查authentication_string是否为空。

所以网上搜到的 `set authentication_string` 大概会报错qwq

```mysql
use mysql;alter user 'root'@'localhost' identified with mysql_native_password by '123456';
```

之后 `mysql -uroot -p` 就可以登录了的样子。

## 集成开发环境安装配置

### vscode

#### 安装

这个直接去官网下载deb包，然后 `apt install code***` 就好了诶。

[https://code.visualstudio.com/Download](https://code.visualstudio.com/Download)

或者直接装：

```bash
sudo apt updatesudo apt install code
```

#### 配置C++编译调试

```bash
sudo apt install gccsudo apt install g++sudo apt install gdb
```

* 安装C/C++插件

* 开个文件夹，写一个Hello World

  * 此处include出红色波浪线看这个：[如何解决vscode检测到#include错误，请更新includePath。](https://blog.csdn.net/weixin_44621617/article/details/107818700)

  * 进终端输入 `gcc -v -E -x c -` ，把这里这几条路径复制出来：

    ```bash
    #include <...> search starts here: 复制下面这几条 /usr/lib/gcc/x86_64-linux-gnu/9/include /usr/local/include /usr/include/x86_64-linux-gnu /usr/includeEnd of search list.
    ```

  * 进vscode，`ctrl + shift + p` ，搜 `c/c++:Edit Configurations（JSON）`

  * 在 `configurations.includePath` 数组里添加上面的几条路径，重启vsc。

* `.vscode` 文件夹新建两个文件粘进去：

  * tasks.json:

    ```json
    {  "version": "2.0.0",  "tasks": [    {      "label": "compile",      "command": "g++",      "args": [        "-g",        "${file}",        "-o",        "${fileDirname}/${fileBasenameNoExtension}"      ],      "problemMatcher": {        "owner": "cpp",        "fileLocation": ["relative", "${workspaceRoot}"],        "pattern": {          "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",          "file": 1,          "line": 2,          "column": 3,          "severity": 4,          "message": 5        }      },      "group": {        "kind": "build",        "isDefault": true      }    }  ]}
    ```

    使用 `ctrl + shift + b` 进行编译。

  * launch.json:

    ```json
    {  "version": "0.2.0",  "configurations": [    {      "name": "C/C++",      "type": "cppdbg",      "request": "launch",      "program": "${fileDirname}/${fileBasenameNoExtension}",      "args": [],      "stopAtEntry": false,      "cwd": "${workspaceFolder}",      "environment": [],      "externalConsole": false,      "MIMode": "gdb",      "preLaunchTask": "compile",      "setupCommands": [        {          "description": "Enable pretty-printing for gdb",          "text": "-enable-pretty-printing",          "ignoreFailures": true        }      ]    }  ]}
    ```

    使用 `F5` 进行调试。

#### 以root启动vscode以收拾各种插件



### IDEA

#### 安装

下载对应版本压缩包：[https://www.jetbrains.com/idea/download/#section=linux](https://www.jetbrains.com/idea/download/#section=linux)

```bash
sumkdir /opt/ideacp idea***.tar.gz /opt/idea/cd /opt/idea/tar -zxvf idea***.tar.gzrm idea***.tar.gzcd ./idea***/bin/#这里最好切换回非root用户qwq./idea.sh #以后都是用这个来启动
```

#### 创建快捷方式

直接在应用内tools选项下创建快捷方式，不用新建 `.desktop` 文件。





咕咕咕～
