---
title: HEXO 安装使用说明
date: 2016-10-24 12:21:51
tags:
    - hexo
    - blog
    - NexT
---

### 安装
1. 安装node.js: https://nodejs.org/en/
2. npm install hexo-cli -g
3. hexo init blog
4. cd blog
5. npm install
6. hexo server

## 251 部署信息
```
#在服务器端使用supervisor维护后台进程
supervisorctl start|stop|restart dmsite
配置文件：/etc/supervisor/conf.d/hexo.conf

192.168.30.251:4000
站点位置：/opt/workspace/hexo/dmteam/
文档位置：/opt/workspace/hexo/dmteam/source/_posts （markdown或HTML文件）
启动服务：hexo server
```


### 使用leancloud
[为NexT主题添加文章阅读量统计功能](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)

```
APP ID: ohqkXUU5txOlX3MjJPCrw7jG-gzGzoHsz
APP KEY: bPQhLzRSEsTqdIKAGRcHLhyN
MASTER KEY: lRx8SU65bjP4SA8cRjMSDwy8
```

### 文档
- [官网](https://hexo.io/)
- [NexT使用文档](http://theme-next.iissnan.com/)
- [NexT@github](https://github.com/iissnan/hexo-theme-next/blob/master/README.md)
- [leancloud](https://leancloud.cn/)
- [七牛云](http://www.qiniu.com/)