---
title: 企业名称的行业分类
date: 2016-09-26 15:58:00
tags:
    - classification
    - 行业分类
    - 公司名称
---
<Excerpt in index | 首页摘要>
根据公司名称进行行业分类标注 <!-- more -->
<The rest of contents | 余下全文>

原项目代码库地址为：git@192.168.30.251:tagIndustryForCompany.git

更新：行业分类标准变动，重新抓取训练数据，并更新分类模型


## 数据抓取
  - 来源：天眼查（http://www.tianyancha.com/）
  - 爬虫代码：192.168.30.251:/opt/workspace/tianyancha/crawler.py
  - 数据：192.168.30.251:/opt/workspace/tianyancha/save (备份的天眼查网页文件，未抽取)


## 清洗数据

  - 192.168.30.251:/opt/workspace/tianyancha/companys.json

    抽取天眼查页面之后的数据

  - 192.168.30.251:/opt/workspace/tianyancha/misc_TianyanZhiLianCompany.json

    每个公司对应的标签列表

  - 192.168.30.251:/opt/workspace/tianyancha/miscFlatten_TianyanZhiLianCompany.json

    与智联数据合并的数据（智联的分类与天眼查的分类对应的，即将智联的数据合并到天眼查的数据中）
    每一行仅有一个公司名称对应一个分类，存在多行是一个公司的情况（一个公司有多个标签）

  - 192.168.30.251:/opt/workspace/tianyancha/miscFlatten_TianyanZhiLianCompany.json_miss2101

    不含有`2101`类别【软件和信息技术服务业】的数据


## 训练模型
  - 详见代码：192.168.30.251:/opt/workspace/tianyancha/industryTagger.py

