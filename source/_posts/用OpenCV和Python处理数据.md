---
title: 用OpenCV和Python处理数据
date: 2021-02-08 23:03:01
tags:
---

> 所以要先学会处理数据什么的叭QAQ

* 开始一个新的IPython或Jupyter会话
* 使用Numpy, sklearn, Matplotlib处理数据
* 才知道ipython按tab可以自动填词

## Numpy数组

简化Python数组运算而设计的数组。

```python
In [1]: import numpy as np

In [2]: li = [i for i in range(10)]

In [3]: li
Out[3]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [4]: li *= 2 #一般列表乘2会复制一遍

In [5]: li
Out[5]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [6]: li = li[:10]

In [7]: li
Out[7]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [8]: int_li = np.array(li) #创建numpy数组

In [9]: int_li
Out[9]: array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

In [10]: int_li *= 2 #此处乘2即把所有元素*2，简化编写(life is short...

In [11]: int_li
Out[11]: array([ 0,  2,  4,  6,  8, 10, 12, 14, 16, 18])

In [12]: int_li.ndim #维数
Out[12]: 1

In [13]: int_li.shape #每一维大小
Out[13]: (10,)

In [14]: int_li.size #元素总数
Out[14]: 10

In [15]: int_li.dtype #数据类型
Out[15]: dtype('int32')
    
In [16]: int_li[-2] #索引与切片不变
Out[16]: 16

In [17]: int_li[3:5]
Out[17]: array([6, 8])
    
In [2]: arr_2d = np.zeros((3, 5)) #创建二维数组，默认float类型

In [3]: arr_2d
Out[3]:
array([[0., 0., 0., 0., 0.],
       [0., 0., 0., 0., 0.],
       [0., 0., 0., 0., 0.]])

In [4]: arr_3d = np.ones((3, 2, 4)) #3个颜色通道（0-1的32位浮点数表示），2x4像素的小图像

In [5]: arr_3d[0, :, :] #第一个通道的颜色信息
Out[5]:
array([[1., 1., 1., 1.],
       [1., 1., 1., 1.]])

In [8]: arr_3d = np.ones((3, 2, 4), dtype = np.uint8) * 255 #同上0-255的8位整数表示RGB
    
In [9]: arr_3d
Out[9]:
array([[[255, 255, 255, 255],
        [255, 255, 255, 255]],

       [[255, 255, 255, 255],
        [255, 255, 255, 255]],

       [[255, 255, 255, 255],
        [255, 255, 255, 255]]], dtype=uint8)
```

## sklearn加载外部数据

实例是(150个样本，4个特征)的三类鸢尾花。

```python
In [1]: from sklearn import datasets as ds #想起箱社关于少女前线的iris真是不错的作品QAQ

In [2]: iris = ds.fetch_openml('iris', version = 1) #下载数据集from:  http://openml.org
# 好像需要一点科学？
In [3]: iris_data = iris['data'] #数据

In [4]: iris_target = iris['target'] #标签

In [5]: iris_data.shape
Out[5]: (150, 4)

In [6]: iris_target.shape
Out[6]: (150,)

In [7]: import numpy as np

In [8]: np.unique(iris_target) #查看不同标签
Out[8]: array(['Iris-setosa', 'Iris-versicolor', 'Iris-virginica'], dtype=object)
```

## Matplotlib可视化数据

Matplotlib是建立在Numpy数组上的数据可视化库。

* 绘制正弦函数图像：

```python
In [1]: import matplotlib as mpl

In [2]: import matplotlib.pyplot as plt

In [3]: import numpy as np

In [4]: %matplotlib #shell中使用魔术命令，自动显示图形
Using matplotlib backend: Qt5Agg
    
In [5]: x = np.linspace(0, 10, 100) #创建x轴上线性空间，x值范围[0, 10]，100个样本点

In [6]: plt.plot(x, np.sin(x)) #计算每个正弦
Out[6]: [<matplotlib.lines.Line2D at 0x21626e6f940>] #图像会自动显示

#脚本使用plt.show()显示图像。
In [7]: plt.style.available #显示样式
In [8]: plt.style.use(Name) #应用样式
In [17]: plt.xkcd() #神奇的样式owo
```

* 可视化外部数据集的数据：

这里采用sklearn的digits数据集。

```python
In [1]: import numpy as np

In [2]: from sklearn import datasets as ds

In [3]: import matplotlib.pyplot as plt

In [4]: %matplotlib
Using matplotlib backend: Qt5Agg

In [5]: digits = ds.load_digits() #加载digits数据集

In [6]: digits.data.shape
Out[6]: (1797, 64)

In [7]: digits.images.shape #data与images的区别仅在像素的排列方式，一个大向量/一堆小向量
Out[7]: (1797, 8, 8)

In [8]: img = digits.images[0, :, :] #选取第一个数据，用images方便绘制。
    
In [11]: plt.imshow(img, cmap = 'gray') #绘图
    #cmap参数指定一个彩图，默认jet，对灰度图像使用gray更有意义。
Out[11]: <matplotlib.image.AxesImage at 0x18a6ff22518>
```

啊..好糊...小一点就不糊了hhh<img src="https://s3.ax1x.com/2021/02/08/yU5ePK.png" width="30px">

绘制一组样本：

```python
In [18]: plt.figure(figsize=(14, 4)) #设置窗口大小
Out[18]: <Figure size 1400x400 with 0 Axes>
    
In [23]: plt.figure(figsize=(14, 4))
    ...: for index in range(0, 10):
    ...:     subplot_index = index + 1 #显示框从1开始编号的
    ...:     plt.subplot(2, 5, subplot_index) #两行五列第index位置
    ...:     plt.imshow(digits.images[index, :, :], cmap = 'gray')
```

