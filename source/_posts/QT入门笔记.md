---
title: QT入门笔记
date: 2021-07-13 07:49:54
tags: [杂项, QT]
---

 ~~入门，跑路，不会再碰第二次了~~

### 基础操作

#### 一些快捷键

```
快捷键
注释ctrl + l
运行ctrl + r
编译ctrl +b
字体缩放ctrl + 鼠标滚轮
查找 ctrl +f
整行移动 ctrl + shift + ↑或者↓
帮助文档 F1
自动对齐 ctrl + i
同名之间的.h和.cpp切换 F4
```

#### 帮助文档使用

摸了好久才发现**帮助文档就在QTCreator里面**QAQ，正确姿势就是在文档里直接搜相关组件。

比如QPushButton：

```cpp
QPushButton Class
The QPushButton widget provides a command button. More...

Header: //所要包含的头文件
#include <QPushButton> 
qmake:
QT += widgets //配置文件里要加上这东西
Inherits: //父类
QAbstractButton
Inherited By: //子类
QCommandLinkButton

List of all members, including inherited members
Obsolete members 
```

#### 项目基础结构

构成：一个.pro基本配置，一个.ui设计ui，一个main.cpp，一堆h和cpp。

main.cpp

```cpp
#include "mainwindow.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    //创建一个窗口对象
    MainWindow *w = new MainWindow();

    //窗口重置
    int height = 600, width = 400;
    w -> resize(height, width);
    //固定窗口大小 w -> setFixedSize();
    w -> setWindowTitle("qwqwq");

    //显示窗口
    w -> show();
    return a.exec();
}
```

mainwindow.cpp

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QPushButton>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent) //注意继承关系，有的地方可能需要手改
    , ui(new Ui::MainWindow)
{
    //创建一个按钮
    QPushButton *btn = new QPushButton;

    //btn -> show(); //顶层窗口形式弹出

    //使btn对象依赖在mainwindow窗口中
    btn -> setParent(this);

    //设置大小
    btn -> resize(200, 100);

    //显示文本
    btn -> setText("qwq");

    //创建另一个按钮，重载函数参数：文本，父亲
    QPushButton *btn2 = new QPushButton("text", this);

    //调整位置，跟css的top和left挺像..?或者传一个QPoint对象
    btn2 -> move(200, 100);

    //ui->setupUi(this); //这句自己生成的，先注释掉qwq
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

#### 对象树

通过`setParent()`会使对象之间连接为对象树，父对象析构时，会把整颗子树的对象析构掉。（**这里所说的对象树跟继承树不一样**），但一个神奇的地方是，**析构时的顺序**：执行当前对象的析构函数 -> 寻找当前对象有没有儿子 -> 执行儿子析构函数 -》 .... -> 释放儿子 -> .... -> 释放当前对象。也就是说**QObject的后代不用管内存释放**，一定程度上简化了内存回收机制。

~~什么？你跟一个竞赛选手讲内存释放？~~

### QT的灵魂——信号和槽

#### 建立连接

建立连接的语法：

`connect(信号发送者，发送的信号（函数的地址），信号接收者，处理的槽函数);`

实例：点按钮，关闭窗口：

```cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    //创建按钮
    QPushButton *btn = new QPushButton("关闭窗口", this);

    //建立连接：四个参数：信号发送者，发送的信号（函数的地址），信号接收者，处理的槽函数
    connect(btn, &QPushButton::clicked, this, &MainWindow::close);
}
```

断开 `disconnect(*, *, *, *);`

#### 自定义信号和槽

新建Hoto类和Kafuu类，派生自QObject。（直接用QT生成就好）

Hoto.h:

```cpp
#ifndef HOTO_H
#define HOTO_H

#include <QObject>

class Hoto : public QObject
{
    Q_OBJECT
public:
    explicit Hoto(QObject *parent = nullptr);

signals:
    /** 自定义信号写在这里
     *  返回值是void，只需要声明，不需要实现
     *  可以有参数，可以重载
     **/
    void mofumofu();
};

#endif // HOTO_H
```

Kafuu.h:

```cpp
#ifndef KAFUU_H
#define KAFUU_H

#include <QObject>

class Kafuu : public QObject
{
    Q_OBJECT
public:
    explicit Kafuu(QObject *parent = nullptr);
    /** 槽函数
     *  返回值void，需要声明和实现
     *  可以有参数，可以重载
     **/
    void pushAway();

signals:

};

#endif // KAFUU_H
```

Kafuu.cpp:

```cpp
#include "kafuu.h"
#include <Qdebug>

Kafuu::Kafuu(QObject *parent) : QObject(parent)
{

}

void Kafuu::pushAway()
{
    //输出到控制台
    qDebug() << "Cocoa was pushed away by Chino when mofumofuing.\n";
}
```

MainWindow.cpp:

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QPushButton>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui -> setupUi(this);
    Hoto *cocoa = new Hoto(this);
    Kafuu *chino = new Kafuu(this);
    connect(cocoa, &Hoto::mofumofu, chino, &Kafuu::pushAway);
    //使用emit触发事件（不用emit也行...?
    emit cocoa -> mofumofu();
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

总之运行之后控制台打印了"Cocoa was pushed away by Chino when mofumofuing."

ex：存在重载时，connect使用函数指针。

```cpp
//假设有个带参QString的pushAway
void (Kafuu:: *push)(QString) = &Kafuu::pushAway;
connect(cocoa, &cocoa::mofumofu, chino, push);
```

**顺带一提**，`QString`转`char*`，`qstr.toUtf8().data()`

**顺带一提信号也可以连接信号**

### 窗口相关

鸽了，，有事看文档qwq

有个好用的东西是Qbrush：

```cpp
void MainWindow::paintEvent(QPaintEvent *)
{
    QPainter painter(this);

    //画线
    painter.drawLine(QPoint(0, 0), QPoint(200, 200));

    //设置填充色
    QBrush brush(Qt::blue);
    painter.setBrush(brush);

    //画矩形：左上角坐标 宽度 高度（画出来是蓝色填充）
    painter.drawRect(QRect(10, 10, 500, 500));

}
```

~~因为做完课设就会一直咕咕咕下去所以直接把这篇推上去了~~

