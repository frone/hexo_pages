---
title: cloudera大数据平台安装文档  
date: 2016-01-27 12:21:52
tags: 
    - bigdata
    - CDH
    - Zeppelin
---
<Excerpt in index | 首页摘要> 
CDH大数据平台安装文档 <!-- more -->
<The rest of contents | 余下全文>

# cloudera大数据平台安装文档
- - -
## 系统环境
- 系统环境
    - 主机：IDC 8台主机
    - 操作系统：**centos 6.5**
    - Cloudera Manager：**5.4.7**
    - CDH： **5.4.7**

- - -
## 安装说明
** 如无说明，指所有主机均进行相同操作 **

- 配置系统环境
    - 关闭Firewall和SELinux
    ```bash
        service iptables stop
        chkconfig iptables off
        sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
    ```

    - 主机时间同步 **ntpdate**
    - 修改hosts文件
        **注意：**推荐构建DNS服务, `host -t  PTR  hostname` 需解析成正确的IP
        - 将各个节点及主机名加入hosts文件
    - ssh无密码互访
        - 参考网络资料

- 创建CDH本地库
    [参考CDH官方手册](http://www.cloudera.com/content/www/en-us/documentation/enterprise/latest/topics/cm_ig_create_local_package_repo.html)
    - 下载库文件
    ```bash
        mkdir -p /var/www/html && cd /var/www/html
        wget -r http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.4.7/
        wget -r http://archive.cloudera.com/cdh5/parcels/5.4.7/
        wget -r http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/5.4.7/
    ```
    - 构建http服务
        - 安装nginx
       ```bash
            yum install nginx
        ```
        - 配置nginx server
        ```bash
            vi /etc/nginx/conf.d/default.conf
        ```
        将 root 对应的目录改成： ** /var/www/html **，如：
        ```text
        location / {
            root   /var/www/html;
            index  index.html index.htm;
            autoindex on;
        }
        ```
        - 启动nginx
       ```bash
            nginx -s reload
        ```
    - 添加yum库文件
        -  创建CDH库文件
        ```bash
            cat /etc/yum.repos.d/cloudera-cdh5.repo
        ```
        内容如：
        ```text
            [cloudera-cdh5]
            name=Cloudera's Distribution for Hadoop, Version 5
            baseurl=http://172.17.4.101/cdh/5/
            gpgkey=http://172.17.4.101/cdh/RPM-GPG-KEY-cloudera
            gpgcheck = 1
        ```
        - 更新yum库
        ```bash
            yum repolist
        ```
- 安装Java
    - CDH库中带有Oracle Java
    ```bash
        yum search oracle-j2sdk
    ```
    安装：
    ```bash
        yum install oracle-j2sdk1.7 -y
    ```
    默认安装路径：**/usr/java/$jdkversion** --> **JAVA_HOME**

- 安装cloudera-manager **（任选一台主机）**
    参考官网手册 [CDH5.4安装](http://www.cloudera.com/content/www/en-us/documentation/enterprise/latest/topics/cm_ig_install_path_a.html)
    - 下载cloudera-manager-installer.bin
    ```bash
        wget http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin
        chmod +x cloudera-manager-installer.bin
    ```
    - 运行Cloudera Manager Server installer (install from the local repository)
    ```bash
        sudo ./cloudera-manager-installer.bin --skip_repo_package=1
    ```
    按提示操作，脚本会自动安装Cloudera Manager Server，安装并初始化内嵌数据库（postgresql），然后启动7180端口。
    安装路径：** /opt/cloudera/parcels/ **

- 添加节点及服务
    - 登陆cloudera-manager管理界面
        访问入口： ** http://cloudera-manager-server:7180 **
        默认用户名/密码： **admin/admin**
    - 按提示操作加入节点，并安装软件包。
        - ![选择CM版本](http://abcxy.qiniudn.com/cloudera_step1-selectEdition.jpg)
        - ![组件展示](http://abcxy.qiniudn.com/cloudera_step2-showStack.jpg)
        - ![配置存储库](http://abcxy.qiniudn.com/cloudera_step3-selectRepo.jpg)
        - ![JDK安装选项](http://abcxy.qiniudn.com/cloudera_step4-installJDK.jpg)
        - ![启用点用户模式](http://abcxy.qiniudn.com/cloudera_step5-singleUser.jpg)
        - ![配置ssh登陆信息](http://abcxy.qiniudn.com/cloudera_step6-configSSH.jpg)
        - ![执行安装脚本](http://abcxy.qiniudn.com/cloudera_step7-clusterInstallation.jpg)
        - ![下载并分配软件包](http://abcxy.qiniudn.com/cloudera_step8-installParcel.jpg)
        - ![主机巡检](http://abcxy.qiniudn.com/cloudera_step9-hostsCheck.jpg)
        - ![主机检测结果](http://abcxy.qiniudn.com/cloudera_step10-showCheck.jpg)
        - ![角色分配](http://abcxy.qiniudn.com/cloudera_step11-clusterConfig.jpg)
        - ![数据库设置](http://abcxy.qiniudn.com/cloudera_step12-dbSetting.jpg)
        - ![审核配置](http://abcxy.qiniudn.com/cloudera_step13-Audit.jpg)
        - ![启动并初始化集群](http://abcxy.qiniudn.com/cloudera_step14-startCluster.jpg)

    - 安装并初始化zookeeper

    - 配置Isilon环境
        - **需在Isilon上新建zone，并创建相应的用户和用户组**
        - 在Isilon URI中填写创建的域名配置信息，如：*hdfs://hadoop.example.com:8020*

    - 添加yarn服务

    - 添加spark服务

    - 添加kafka服务 * 未在基础parcels包中，必要时，创建离线安装库 *


- - -
## 构建Zeppelin环境
[Apache Zeppelin简介](http://zeppelin.incubator.apache.org/)

- 准备编译环境
    - Java
        > 见** 安装说明 **中的第三节
    - Maven
        - 下载maven二进制包，解压，
        - 将bin文件夹加入PATH变量中
    - Node.js （建议安装，编译时也会自动安装，如编译失败，手动安装依赖包）
        - 下载nodejs二进制包，解压
            **注意：** nodejs版本无需过高，推荐使用 v0.10.18（zeppelin默认版本）
        - 将bin文件夹加入PATH变量中

- 下载及编译
    - 下载Zeppelin源码
    ```bash
        git clone https://github.com/apache/incubator-zeppelin.git
    ```
    - 使用maven编译Zeppelin
    ```bash
        mvn clean package -Dhadoop.version=2.6.0-cdh5.4.0 -DskipTests
    ```
    **注意：**如果失败，多尝试几次；如缺少node依赖包，使用node或者bower手动安装

- 配置Zeppelin环境
    - zeppelin-env.sh
        修改`SPARK_HOME`、`MASTER`、`HADOOP_CONF_DIR`等配置项
    - zeppelin-site.xml
        设置`zepplin.server.port`等，如改成**11180**，默认8080

- 启动与停止Zeppelin
    - 启动
    ```bash
        cd $ZEPPELIN_HOME
        ./bin/zeppelin-daemon.sh start
    ```
    - 停止
    ```bash
        cd $ZEPPELIN_HOME
        ./bin/zeppelin-daemon.sh stop
    ```

- 创建Zeppelin Notebook
    - 使用浏览器打开Zeppelin页面
        `http://zeppelin-server:11180`
    - 在Zeppelin主页上设置Interpreter、创建Notebook等
        参见[Zeppelin Tutorial](http://zeppelin.incubator.apache.org/)

- - -
