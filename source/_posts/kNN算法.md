---
title: '摸一摸k-NN算法'
date: 2021-02-16 09:47:44
tags: [python, 机器学习]
---

## OpenCV实现k-NN

[OpenCV中文官方文档](http://www.woshicver.com/)

### 算法描述

> 如果我们是红队球迷，我们不可能搬到大多数人都认为可能是蓝队球迷的社区。

k-NN算法认为一个数据点可能与其邻居属于同一类。

没了。~~这不比Prim还水？~~

### 训练数据生成

前置：

```python
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt
plt.style.use('ggplot')
np.random.seed(42)
```

随机生成二维点及标签：

```python
single_data_point = np.random.randint(0, 100, 2) #随机生成坐标
print(single_data_point) #[51, 92]
single_label = np.random.randint(0, 2) #随机指定标签
print(single_label) #0
```

大量数据生成：

```python
#生成器
def generate_data(num_samples, num_features=2): #数据个数，特征数
    data_size = (num_samples, num_features) #行，列
    train_data = np.random.randint(0, 100, size=data_size)
    label_size = (num_samples, 1)
    labels = np.random.randint(0, 2, size=label_size)
    return train_data.astype(np.float32), labels
train_data, labels = generate_data(11)
print(train_data[0], labels[0]) #[71., 60.] [1]
```

绘制：

```python
#尝试绘制单点
plt.plot(train_data[0, 0], train_data[0, 1], color='r', marker='^', markersize=10)
plt.xlabel('x-coordinate') #坐标轴描述
plt.ylabel('y-coordinate')
plt.show()
```

```python
#绘制全部点
def plot_data(all_blue, all_red):
    plt.figure(figsize=(10, 6))
    #x坐标列表，y坐标列表，颜色，标记，标记大小
    plt.scatter(all_blue[:, 0], all_blue[:, 1], c='b', marker='s', s=180)
    plt.scatter(all_red[:, 0], all_red[:, 1], c='r', marker='^', s=180)
#numpy列表功能：list.ravel()将列表压成向量(一维数组)，list == 0生成布尔数组bool_list，list[bool_list]选取数组中的一部分(true对应下标元素)。
blue = train_data[labels.ravel() == 0]
red = train_data[labels.ravel() == 1]
plot_data(blue, red)
plt.show()
```

<img src="https://s3.ax1x.com/2021/02/15/y6JYOP.png" width="500px">

### 训练分类器

接上段：

```python
import cv2
knn = cv2.ml.KNearest_create()
knn.train(train_data, cv2.ml.ROW_SAMPLE, labels) #成功会返回true
```

### 预测一个新数据点的标签

生成一个新点并绘制：

```python
newcomer, _ = generate_data(1)
print(newcomer) #[[91., 59.]]
plot_data(blue, red)
plt.plot(newcomer[0, 0], newcomer[0, 1], 'go', markersize=14)
plt.show()
```

<img src="https://s3.ax1x.com/2021/02/15/y6JJyt.png" width="500px">

查看预测结果：

```python
ret, results, neighbor, dist = knn.findNearest(newcomer, 1) #待测点，最近的k个邻居
print("Predicted label:\t{}\nNeighbor's label:\t{}\nDistance to neighbor:\t{}\n".format(results, neighbor, dist))
'''
k = 1
Predicted label:        [[1.]]
Neighbor's label:       [[1.]]
Distance to neighbor:   [[250.]]
k = 2
Predicted label:        [[1.]]
Neighbor's label:       [[1. 1.]]
Distance to neighbor:   [[250. 401.]]
k = 3
Predicted label:        [[1.]]
Neighbor's label:       [[1. 1. 0.]]
Distance to neighbor:   [[250. 401. 784.]]
'''
#另一种姿势
knn.setDefaultK(3)
print(knn.predict(newcomer)) #(1.0, array([[1.]], dtype=float32))
```

## k-NN实现手写数字识别

### 数据准备

这里采用sklearn的datasets中的digits。

```python
In [1]: import numpy as np
   ...: from sklearn import datasets as ds
   ...: digits = ds.load_digits()
```

包含内容：

```python
In [2]: digits.keys()
Out[2]: dict_keys(['data', 'target', 'frame', 'feature_names', 'target_names', 'images', 'DESCR'])
```

data和images和target：

```python
In [13]: digits.images[0]
Out[13]: #每个元素是 8×8像素的二维数组
array([[ 0.,  0.,  5., 13.,  9.,  1.,  0.,  0.],
       [ 0.,  0., 13., 15., 10., 15.,  5.,  0.],
       [ 0.,  3., 15.,  2.,  0., 11.,  8.,  0.],
       [ 0.,  4., 12.,  0.,  0.,  8.,  8.,  0.],
       [ 0.,  5.,  8.,  0.,  0.,  9.,  8.,  0.],
       [ 0.,  4., 11.,  0.,  1., 12.,  7.,  0.],
       [ 0.,  2., 14.,  5., 10., 12.,  0.,  0.],
       [ 0.,  0.,  6., 13., 10.,  0.,  0.,  0.]])

In [10]: digits.data[0]
Out[10]: #images展成 64个元素的一维数组
array([ 0.,  0.,  5., 13.,  9.,  1.,  0.,  0.,  0.,  0., 13., 15., 10.,
       15.,  5.,  0.,  0.,  3., 15.,  2.,  0., 11.,  8.,  0.,  0.,  4.,
       12.,  0.,  0.,  8.,  8.,  0.,  0.,  5.,  8.,  0.,  0.,  9.,  8.,
        0.,  0.,  4., 11.,  0.,  1., 12.,  7.,  0.,  0.,  2., 14.,  5.,
       10., 12.,  0.,  0.,  0.,  0.,  6., 13., 10.,  0.,  0.,  0.])

In [11]: type(digits.data[0, 0])
Out[11]: numpy.float64 #每个像素是一个float64表示的颜色
    
In [20]: digits.target_names
Out[20]: array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) #每个图片对应标签就是数字
    
In [23]: len(digits.target)
    ...: len(digits.data)
Out[23]: 1797 #数据集包含1797个样本
```

### 实现思路

直接把图片展开，看成64维向量（或者该叫64维空间内的点?）。

距离表示图片相似程度，相同的数字距离较近。

套k-NN板子。

### 具体实现

> Life is short, I use Python.

```python
import numpy as np
import cv2
from sklearn import datasets as ds
from sklearn import metrics as mts
#数据准备
digits = ds.load_digits()
knn = cv2.ml.KNearest_create()
train_data = digits.data[:1000].astype(np.float32) #前999个用来训练，转为float32便于读取
train_labels = digits.target[:1000]
test_data = digits.data[1000:].astype(np.float32)
test_labels = digits.target[1000:]
#模型训练
knn.train(train_data, cv2.ml.ROW_SAMPLE, train_labels)
#测试
_, res, _, _ = knn.findNearest(test_data, k=10)
print(mts.accuracy_score(res, test_labels)) #正确率0.9560853199498118
```

