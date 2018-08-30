---
title: pip使用国内源
date: 2018-07-27 11:21:51
tags:
    - pulp
    - pip
categories: 算法
---

### 安装 Pulp 等包时遇到问题
在安装一个不是很常用的python库时 遇到了长时间无法下载安装的问题，之前也有配置过国内的pip 源所以在这整理下

### 直接修改国内源为默认

创建或修改配置文件

linux的文件在~/.pip/pip.conf，

windows在%HOMEPATH%\pip\pip.ini）

```
[global]
index-url = http://pypi.douban.com/simple
[install]
trusted-host=pypi.douban.com
```

除了豆瓣还有其他的国内源：

- 阿里云 http://mirrors.aliyun.com/pypi/simple/
- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
- 豆瓣(douban) http://pypi.douban.com/simple/
- 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
- 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

### 其他

此外在此mark一下 Pulp， 处理线性规划问题

1. [文档主页](https://pythonhosted.org/PuLP/index.html)
2. [案例](https://pythonhosted.org/PuLP/CaseStudies/a_blending_problem.html)
