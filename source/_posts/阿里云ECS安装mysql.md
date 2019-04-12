#准备工作

```shell
lsb_release -a # 查看系统版本

yum -y update # 更新系统软件

wget https://dev.mysql.com/get/mysql80-community-release-el7-2.noarch.rpm # 下载rpm包
```



# 开始安装

```shell
rpm -Uvh mysql80-community-release-el7-2.noarch.rpm

yum repolist all | grep mysql # 查看安装mysql配置文件

sudo yum install mysql-community-server
```

# 启动服务

```mysql
service mysqld start

service mysqld status

ps -aux|grep mysql # 查看进程
netstat -anp|grep `pid` # 查看进程对应端口

# 通过端口查看进程
netstat -nap | grep 端口号 
```

Mysql服务启动后发生如下事件

1. mysql server初始化

2. An SSL certificate and key files are generated in the data directory.

3. The [validate_password plugin](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html) is installed and enabled.

4. A superuser account `'root'@'localhost'` is created. A password for the superuser is set and stored in the error log file. To reveal it, use the following command:

   ```shell
   grep 'temporary password' /var/log/mysqld.log
   
   # 可以使用如下方法更新密码
   shell> mysql -uroot -p'*****'
   mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
   mysql> update user set Host = '%' where user ='root';  # 开启远程登录
   mysql> flush privileges;
   ```

   

# 安全相关设置

1. 在ECS管理页面添加安全组规则

![](http://img.alpstudy.com//imgs/20190402183134_HmTblW_Screenshot.jpeg)

2. 添加3306 入口规则，无需启用防火墙

![image-20190402183207794](/Users/xingshulin/Library/Application Support/typora-user-images/image-20190402183207794.png)

# 

# 参考资料

1. [阿里云ECS服务器自建数据库的一些坑](<https://blog.csdn.net/panyox/article/details/77780098>)
2. [阿里云ECS服务器CentOS7上安装MySql服务](<https://yq.aliyun.com/articles/285398>)

3. <https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/>
4. [用户管理](<https://www.cnblogs.com/chanshuyi/p/mysql_user_mng.html>)