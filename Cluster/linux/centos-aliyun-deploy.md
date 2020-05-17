## 阿里云部署Java网站

- 作者：xiangzepro
- 链接：[阿里云部署Java网站和微信开发调试心得技巧(上)](http://www.imooc.com/article/20583)
- 来源：慕课网

> 本文主要是记录在阿里云服务器从零开始搭建Java执行环境并且部署web project的过程

#### 一、申请阿里云服务器
> 购买阿里云服务器

#### 二、SSH远程连接云服务器

##### windows系统
> 使用第三方SSH工具：如XShell进行ssh远程连接

##### mac os 或 linux
> 1. 切换到root权限下： mac下打开终端，使用 `sudo -i`命令
> 2. 通过ssh命令连接阿里云linux服务器：`ssh root@118.31.7.201`,root是账户名，@后面是连接的linux服务器的ip地址

#### 三、搭建项目的执行环境（Java）
搭建程序的执行环境，下面是一些常用的执行环境的清单：
- JDK（这里选择的是JDK1.8） [下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- Tomcat 8  [下载地址](http://tomcat.apache.org/download-80.cgi#8.0.46)
- Mysql（这里选择的是Mysql5.7）repo源，然后通过centos自带的yum安装  [下载地址](https://dev.mysql.com/downloads/repo/yum/)
- Redis  [下载地址](https://redis.io/download)

> 将上面的软件都下载到本地，并且上传到服务器：<br/>
> （1）Mac系统或者Linux可直接使用scp命令行进行上传;<br/> 
> （2）Win系统需要通过filezilla可视化上传工具上传;<br/>
> （3）直接登录服务器，通过wget+ftp地址直接下载这些软件;<br/>

##### JDK安装（这里选择的是jdk1.8）

下载地址： [Oracle JDK download](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

- **清理系统默认自带的jdk**
```
rpm -qa | grep jdk -- 查看系统中自带jdk版本
yum remove xxx (xxx为上个命令查到的jdk版本)
```

- **使用wget命令下载jdk**
> 需要在wget的时候加上一个特殊的Cookie： `--no-cookie --header "Cookie: oraclelicense=accept-securebackup-cookie"`

完整命令：
```
 wget --no-cookie --header "Cookie: oraclelicense=accept-securebackup-cookie"  https://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.rpm
```

- **安装JDK**
    - 赋予文件可执行权限： 
    ```
    chmod +x jdk-8u191-linux-x64.rpm -- 赋予该文件可执行的权限(所有用户)
    chmod 777 jdk-8u191-linux-x64.rpm -- 777表示把用户、用户组和其他人这三个组都赋予读写执行的权限
    ```
    - 安装rpm软件包： `rpm -ivh jdk-8u191-linux-x64.rpm`
    - 默认安装路径  `/usr/java` 查看java的版本信息若出现下图信息则表示成功

![安装成功](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-11-132501.png)

- **JDK环境变量配置**
```bash
1. 编辑环境变量配置文件
vim /etc/profile

2. 在最下方增加
export JAVA_HOME=/usr/java/jdk1.8.0_191
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
注： JAVA_HOME 为安装jdk的路径

3. 在export PATH 中添加$JAVA_HOME/bin
export PATH=$PATH:$JAVA_HOME/bin


4. 保存退出，通过vim的 ":wq" 命令进行保存退出

5. 使配置生效
source /etc/profile
```

##### Mysql

> Mysql(这里选择的是Mysql5.7)repo源，后通过centos自带的yum安装

下载的地址为: [MySQL Yum存储库](https://dev.mysql.com/downloads/repo/yum/)

[使用MySQL Yum存储库在Linux上安装MySQL](https://www.cnblogs.com/fancunwei/articles/9467316.html)

- 下载并且安装MySQL
```
-- 下载配置mysql yum源的rpm包
wget http://repo.mysql.com/mysql57-community-release-el7-11.noarch.rpm
-- 安装用来配置mysql的yum源的rpm包
rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
-- 安装MySQL
yum install mysql-community-server
```

- 开启MySQL服务
```
-- 设置mysql开机自启
systemctl enable mysqld.service

-- 开启mysql服务
service mysqld start
systemctl start mysqld.service

-- 查看mysql服务开启状态
service mysqld status
systemctl status mysqld.service

-- 监听端口状态
ps -ef | grep mysqld
netstat -ano | grep 3306
```

- 查看并且修改登录密码
```
grep 'temporary password' /var/log/mysqld.log
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```
> MySQL的 validate_password 插件默认安装。这将要求密码包含至少一个大写字母，一个小写字母，一个数字和一个特殊字符，并且密码总长度至少为8个字符。

- 开启远程连接
    - 通过阿里云控制台开放3306端口
    - 配置一个支持远程登录的帐号
    ```
    -- 创建christopher帐号并授权，同时设置密码
    CREATE USER 'username'@'host' identified by 'password';
    grant all privileges on *.* to 'christopher'@'%' identified by 'ChristopherChan';
    -- 生效配置
    flush privileges;
    
    -- 远程登录
    mysql -uChristopher -P3306 -h47.107.64.174 –p 
    ```
关于MySQL5.7 的添加、删除、授权请参考

- [MySQL5.7 添加用户、删除用户与授权](https://www.cnblogs.com/xujishou/p/6306765.html)
- [MySQL5.7.12新密码登录方式及密码策略](https://www.cnblogs.com/jonsea/p/5510219.html)

##### Redis

下载地址： [Redis官网下载](https://redis.io/download)

- 下载Redis压缩包
```
-- 使用wget命令进行下载
wget http://download.redis.io/releases/redis-4.0.11.tar.gz

-- 解压redis安装包
tar -zxvf redis-4.0.11.tar.gz

-- 设置redis1️以支持远程登录
vim redis-4.0.11/redis.conf
将bind 127.0.0.1 注释掉

-- 为redis的运行设置守护进程
daemonize yes
```

- 安装Redis
```
-- 去到解压缩后的目录
cd redis-4.0.11
-- 安装redis
make
-- 启动redis服务
src/redis-server redis.conf
```

- Redis连接测试
```
-- 通过redis-cli客户端连接到redis服务器
src/ridis-cli
-- 当输入ping 得到pong的回应之后，证明redis配置完成
```

##### Tomcat

下载地址： [Tomcat download](http://tomcat.apache.org/)

- 解压tomcat压缩包
- 启动tomcat
```
-- 启动tomcat
./apache-tomcat-8.0.53/bin/startup.sh
-- 关闭tomcat
./apache-tomcat-8.0.53/bin/shutdown.sh
```

#### 四、在服务器上发布并运行web project

- 修改tomcat默认启动端口为80，便于微信登录
- 重启tomcat，使配置生效
- 修改自己本地的网站的相关配置
- 将项目打成war包
- 上传至服务器tomcat的webapps目录下

> 如何将本地的图片文件上传至服务器？

``` shell
-- 首先在服务器上创建存放图片的文件夹
mkdir -p /home/christopher/image/project/electronic-shop

-- 其次将本地的图片文件夹打成zip压缩包
-- 将压缩包上传至服务器指定图片存放目录下

-- 安装zip包以执行相关zip命令
yum install -y unzip zip

zip相关指令：
-- 将当前目录下的所有文件和文件夹全部压缩成myfile.zip文件,－r表示递归压缩子目录下所有文件
zip -r myfile.zip ./*

-- 解压缩
unzip -o -d /home/sunny myfile.zip
-o:不提示的情况下覆盖文件；
-d:-d /home/sunny 指明将文件解压缩到/home/sunny目录下；

```

#### 四、域名解析