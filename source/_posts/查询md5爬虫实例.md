---
title: '[杂项] python查询md5爬虫和简单excel表格处理'
date: 2021-06-07 10:30:35
tags: [python, 爬虫, 杂项]
---

## 背景

报名了学校"2021小学期网络攻防实践创新课程"，然后这样公布了名单。。

<img src="https://z3.ax1x.com/2021/06/07/2dyBz6.png" width="500px">

该说不愧是搞安全的ヘ(_ _ヘ)，~~攻防开始~~。拿一堆md5加密过的东西放上去，倒是能查到自己的，不过还有点好奇同专业的同学还有谁选上这门课了，写个爬虫玩玩。

> md5: 一种不可逆的加密算法

## 思路

目前手中的资料有：

* 全专业同学的学号与姓名名单
* 入选同学的学号姓名（上图，加密过的）
* 随手找的一个md5加密网站：https://www.qvdv.com/tools/qvdv-md5（api：https://www.qvdv.com/tools/qvdv-md5-_index.html）

打算拿学号那一栏去对比。查询每个人学号加密后的结果，如果在入选名单中出现就是被选上啦，，

具体请求方式是这样的，POST加上form参数：

<img src="https://z3.ax1x.com/2021/06/07/2dgdij.png" width="500px">

## 代码实现

大部分时间花在了excel表格的处理上，不太会玩QAQ

提前准备了 全部人员信息表格 `total.xlsx` ，通过人员md5加密的表格 `pass.xlsx`

```python
import requests as rq
from bs4 import BeautifulSoup
import json
import csv
from openpyxl import load_workbook as readxl
from openpyxl import Workbook

#设置要爬取的url
url = "https://www.qvdv.com/tools/qvdv-md5-_index.html"

#读取全部信息表格，第一个工作表
table = readxl('./total.xlsx')
totSheet = table.worksheets[0]
#print(totSheet[1][1].value)

#读取通过者名单表格
table = readxl('./pass.xlsx')
passSheet = table.worksheets[0]
#稍微打包一下方便查找
passerMd5 = []
for passer in passSheet:
  passerMd5.append(passer[0].value)

#准备输出到这里
#创建一个Workbook对象
wb = Workbook()
#新建一张表
wb.create_sheet("sheet1") 
# 获取当前活跃的sheet，默认为第一张sheet
table = wb.active
#tot记录总人数，顺带当作表格的索引，顺带这个是从1开始计数的
tot = 1

#循环所有人名单查找通过者
for stu in totSheet:
  #打包要post的form数据
  data = {
  #这里通过学号的md5来找
  "code": stu[0].value,
  "codeType": "x32"
  }
  #post请求获取md5值
  res = rq.post(url, data)
  #返回一个字符串，转成json再找data就是学号对应的md5值
  md5Value = json.loads(res.text)["data"]
  #当前对象在通过名单中，加入表格
  if (md5Value in passerMd5):
    table.cell(tot, 1, stu[0].value)
    table.cell(tot, 2, stu[1].value)
    tot += 1

#保存表格
wb.save("passers.xlsx")
```

