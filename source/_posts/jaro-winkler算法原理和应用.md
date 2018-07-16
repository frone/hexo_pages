---
title: Jaro–Winkler算法原理和应用
date: 2018-07-16 16:21:51
tags:
    - Jaro–Winkler
    - blog
    - NexT
categories: 算法
mathjax: true
---
### 算法简介
The Jaro–Winkler distance (Winkler, 1990)是计算2个字符串之间相似度的一种算法。它是Jaro distance算法的变种。主要用于record linkage/数据连接（duplicate detection/重复记录）方面的领域，Jaro–Winkler distance最后得分越高说明相似度越大。Jaro–Winkler distance 是适合于串比如名字这样较短的字符之间计算相似度。0分表示没有任何相似度，1分则代表完全匹配。

### 算法定义
1. The Jaro distance算法最后得分公式：
$$ d_j = \frac{1}{3}\left(\frac{m}{|s1|} + \frac{m}{|s2|} + \frac{m-t}{m} \right) $$
其中:   
- s1、s2 是要比对的两个字符
- $d_j$是最后得分
- m是匹配的字符数
- t  是换位的数目

2. Match Window(匹配窗口)计算公式
$$MW=\left(\frac{MAX(|s1|,|s2|)}{2}\right) - 1$$
其中：
- s1、s2 是要比对的两个字符
- MW是匹配窗口值

3. 上述公式解释
- 字符串s1与字符串s2在做匹配计算时，当两个字符的距离不大于公式二的最后结果(匹配窗口)即认为是匹配的。
- 当s1、s2中字符相匹配但是字符位置不一样时发生换位操作、而公式一中换位的数目t为不同顺序的匹配字符的数目的一半。比如:两个字符串CRATE和TRACE做匹配操作，字符串中仅有'R' 'A' 'E'三个字符是匹配的，即m=3。为什么'C', 'T'不算做是匹配的呢。因为虽然'C', 'T'都出现在两个字符串中，但是通过公式二得出匹配窗口值为 (5/2)-1=1.5。而两个字符串中'C', 'T'字符的距离均大于1.5。所以不算做匹配。因此t=0。在另一组字符串DwAyNE 与 DuANE 。匹配的字符D-A-N-E 在两个字符串中有相同的字符顺序，所以不需要进行换位操作，因此t=0,m=4。

4. Jaro–Winkler distance算法公式
Jaro-Winkler算法给予了起始部分就相同的字符串更高的分数，它定义了一个前缀范围p，对于要匹配的两个字符串，如果前缀部分有长度为L的部分字符串相同，则Jaro-Winkler Distance为:
$$d_w = d_j + L * P(1 - d_J)$$
其中:   
- $d_j$是Jaro distance最后得分
- L是前缀部分匹配的长度
- P是一个范围因子常量，用来调整前缀匹配的权值，但是P的值不能超过0.25，因为这样最后得分可能超过1分.Winkler的标准默认设置值P=0.1。

### 例子
给出两个字符串 s1 MARTHA 和 s2 MARHTA、我们可以得出：
- m = 6
- |s1| = 6
- |s2| = 6
- 两组字符T/H和H/T要进行换位操作，因此t=2/2=1; 
我们可以根据公式一得出Jaro得分：
$$d_j = \frac{1}{3}\left(\frac{6}{|6|} + \frac{6}{|6|} + \frac{6-1}{6}\right)$$
如果使用Jaro–Winkler，并且取范围因子P=0.1,我们会得出:
P=0.1
L=3
$d_w = 0.944 + (3 * 0.1(1 - 0.944))=0.961$

### python 运行
1. 安装lib
```
pip install jaro_winkler
```
2. 程序运行
```
print(jaro.jaro_metric("MARTHA".decode("utf-8"),"MARHTA".decode("utf-8")))
```

### 优化

1. 多进程
2. pandas
















