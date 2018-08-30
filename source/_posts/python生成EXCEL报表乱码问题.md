---
title: python生成EXCEL报表乱码问题
date: 2018-07-30 11:21:51
tags:
    - python
    - 编码
categories: python
---

### 乱码问题
从数据库导出数据，整理后使用jaro.jaro_winkler_metric 进行文本距离的计算，计算之前需要使用decode("utf-8")方法进行编码转换，但是在结果数据存储成CSV是遇到了乱码问题，使用EXCEL打开就会乱码，以往都是直接使用file.open()方法进行txt文本的输出，不会出现编码问题。

感觉问题应该在excel端，需要加个header之类的

### 解决问题
后来在知乎找到了满意的答案。
使用csv写入内容，并在头部写入BOM信息 csvfile.write(codecs.BOM_UTF8) 顺利解决问题

```
#!/usr/bin/env python
# -*- coding: UTF-8 -*-
import csv
import codecs
with open('test.csv', 'wb') as csvfile:
    csvfile.write(codecs.BOM_UTF8)    
    spamwriter = csv.writer(csvfile, dialect='excel')    spamwriter.writerow(['测试'] * 5 + ['Baked Beans'])    spamwriter.writerow(['Spam', 'Lovely Spam', 'Wonderful Spam'])
```
转自
作者：Nice2
链接：https://www.zhihu.com/question/34201726/answer/150221584