---
title: airflow EmailOperator 发送邮件 附件文件名丢失或乱码问题
date: 2018-08-30 08:21:51
tags:
    - airflow
    - 任务调度
categories: 任务调度
---

### 开始使用airflow
最早开始使用airflow是因为公司的服务过度 依赖crontab了，完全通过时间进行任务调度，而且不便于追踪任务运行情况，也不好处理任务之间的依赖关系。更不要说管理多服务器的crontab问题了。

于是自己进行软件选型，查阅相关资料。因为本身是做数据工作的，不希望只是一个简简单单的crontab功能升级，于是选择了airflow。

airflow的主要特点如下：

- 我们可以将一些常做的巡检任务，定时脚本（如 crontab ），ETL处理，监控等任务放在 airflow 上集中管理，甚至都不用再写监控脚本，作业出错会自动发送日志到指定人员邮箱，低成本高效率地解决生产问题。
- Airflow 适用于调度作业较为复杂，特别是各作业之间的依赖关系复杂的情况​

备选是cronsun

- cronsun 是一个分布式任务系统，单个结点和 *nix 机器上的 crontab 近似。支持界面管理机器上的任务，支持任务失败邮件提醒，安装简单，使用方便，是替换 crontab 一个不错的选择）

具体关于任务调度可以参考之前转发的一篇文章，[浅谈工作流调度系统](https://blog.csdn.net/frone/article/details/82218370)

### 使用 EmailOperator 模块发送邮件
当然写本篇文章的目的是解决使用airflow时遇到的发送邮件问题，我在使用EmailOperator发送xlsx附件的时候遇到了附件文件名丢失问题，有的客户端可以正常接收（例如mac的邮件），但是服务器端和Foxmail则遇到了问题，因为之前使用smtp发送邮件的时候遇到过类似问题，所以决定去airflow的源代码中一探究竟。
![附件文件名丢失](https://img-blog.csdn.net/20180830114431733?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zyb25l/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 修改源代码处理文件名问题
查看源代码追踪到了问题，EmailOperator使用了 from airflow.utils.email import send_email，通过email这个类完成邮件的最终发送，所以讲问题定位到了这里。
第84行
```
part['Content-Disposition'] = 'attachment; filename="%s"' % basename
part['Content-ID'] = '<%s>' % basename
```
修改为
```
# update by frone due to email attachment reason
part.add_header('Content-Disposition', 'attachment', filename=('utf-8', '', basename))
part.add_header('Content-ID','<%s>'%basename)
```

最终成功解决问题，可以正常使用airflow的邮件功能了。
![问题得到解决](https://img-blog.csdn.net/20180830114520776?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zyb25l/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

