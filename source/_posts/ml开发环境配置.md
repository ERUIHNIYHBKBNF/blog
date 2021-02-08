---
title: ml开发环境配置
date: 2021-02-08 23:01:47
tags:
math: true
---

# 一些概念

## 人工智能相关

### 人工智能

* 模糊逻辑、计算机视觉、自然语言处理。
* 机器学习、推荐系统
* 弱人工智能：应用于特定环境，如小爱同学。
* 强人工智能：各种适应性，如科幻电影的智能机器人。

### 机器学习

* 人工智能的一种实现方法
* 设定任务，用算法从大量数据中学习如何完成任务。

### 深度学习

* 机器学习的一种技术
* 深度神经网络 + 大量数据
* 深度神经网络：多层的神经网络
* 根本是矩阵运算与向量运算的数学问题。

### 神经网络

* 多个“神经元”连接在一起
* 神经元：一个函数，一系列输入得到输出。

## 深度学习与C++与Py

### 语言特性

* 生产效率：Python
  * 动态类型语言，生产效率高，跨平台语言。
  * 运行效率低。
* 运行效率：C++
  * 运行效率基本达到C水平，可内嵌汇编，底层优化。
  * 静态类型语言，函数库稀烂，不好跨平台。

### 语言选择

* 面向应用者：生产效率优先，例如监控识别、自动驾驶等。
* 面向系统开发者：运行效率优先，开发深度学习底层系统例如TensorFlow。

> 同学们学C++有什么用呢？不管其它用处怎么样，至少我们可以看TensorFlow的核心代码对吧？
>
> ——北邮MOOC C++ 崔毅东老师

## 数学相关

### 向量vector

* 平面向量是二维平面内有大小(magnitude)和方向(direction)的量。
* 二维向量代数表示是一个数对 $ \vec{a} = (x, y) $，是最简单的向量。
* 一般表示方法 $ \vec{a} = (a_1, a_2, a_3, ..., a_n) $   $ \vec{a} = \left[\begin{array}{c} a_1\\a_2\\...\\a_n\end{array}\right]   $ $\vec{a} = \left[\begin{array}{cccc} a_1 & a_2 & ... & a_n\end{array}\right]$

### 矩阵matrix

* $m$ 行 $n$ 列元素排列成矩形阵列，元素可以是数字、符号、数学式。
* C语言使用数字表示，C++用 `std::array` 表示。
* 一列或一行矩阵是向量。

### 张量tensor

* 流行的深度学习系统TensorFlow
* 多维矩阵组合在一起，矩阵直接具有关系，二阶是矩阵，一阶是向量，*理解为矩阵是张量的退化形式*。
* 线性代数库Eigen基本数据单元是Matrix

## 机器学习相关

### 类别

> 嘛..个人理解什么的emmm

* 监督学习

  ​	进行学习，程序被告知正确答案，通过设计一个能得出正确答案的模型，解决其它问题，分训练数据与测试数据。

* 无监督学习

  ​	给出一个复杂难题，自行求解。

* 强化学习

  ​	通过性能评估，强化相关模型。

### 工作流程

通过建立数学模型来理解数据，其具有自行调整内部参数的能力。

机器学习实际上是关于数据的，要对数据进行**预处理**，转换成易于解析的形式。

* 预处理：
  * 特称选取：选取有用的特性，例如颜色，SURF或HOG特征什么的（不懂qwq）
  * 特征提取：将原始数据变换到所期望的特征空间。

* 训练阶段：在训练数据集上，通过机器学习算法，训练一个机器学习模型。
* 测试阶段：在测试数据集上（不同于训练数据集的新数据），评估模型，关注模型对新数据的泛化能力。

# 开发环境配置

为避免接口不同，采用与书上一样的版本（现在都老了好像），**可能涉及多余的操作，但至少能保证稳定使用**。

## Step1：安装anaconda

> Anaconda是一个可用于科学计算的Python发行版blabla...

看不懂，，总之anaconda是一个挺牛逼的各种东西的集成，比如装包，管理环境什么的，并且**可以创建一个python的虚拟环境**。*这样理解大概没问题叭qwq*

因为要使用旧版的python和轮子，用anaconda会方便一些。

~~这..有手就行叭qwq~~

https://www.anaconda.com/

## Step2：创建环境安装所需包

* 创建环境：打开Anaconda Navigator，选择Environments，Create，Python版本选择3.6（为保证与书上接口一致）。

<img src = "https://s3.ax1x.com/2021/02/07/yNnKKO.png" width = "750px">

* 添加 `conda-forge` 通道并安装相应包。

  * 选中刚刚创建的环境，Channels，Add...，输入conda-forge **回车**，Update。

    <img src = "https://s3.ax1x.com/2021/02/07/yNnaM8.png" width = "500px">

  * 下列出所需库与版本：

    ```yaml
    name: OpenCV-ML
    channels:
      - conda-forge
    dependencies:
      - python==3.6
      - numpy==1.16.1
      - scipy==1.1.0
      - scikit-learn==0.20.1
      - matplotlib
      - jupyter==1.0
      - notebook==5.7.4
      - pandas==0.23.4
      - theano
      - keras==2.2.4
      - graphviz
      - mkl-service==1.1.2
      - opencv==4.1.0
    ```

  * 搜索所需的包，在方框处鼠标右键选择对应版本，选中安装。<img src="https://s3.ax1x.com/2021/02/07/yNK39I.png" width="200px">不来点科学的话挺慢的，，用的时候现装也行？

* 好像之后就可以直接在anaconda里用了emm

## Step3：安装验证

* Anaconda中选择对应环境 `Open Terminal`

* 命令窗口启动IPython(运行Python命令的一个交互式shell)，尝试查看几个库的版本：

  ```python
  import sys
  print(sys.version)
  
  import cv2
  cv2.__version__
  ```

* 查看OpenCV的ml模块（看看...看不懂Orz

  ```python
  import cv2
  dir(cv2.ml)
  ```

* 在anaconda里打开Jupyter 或者 输入命令 `jupyter notebook` 瞧瞧，大概是一个能在web浏览器上使用对应python环境运行本地代码的工具，看上去挺好用的。