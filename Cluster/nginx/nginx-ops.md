# Nginx 运维（安装与使用）

## 普通安装

### Windows安装

（1）进入[官方下载地址](https://nginx.org/en/download.html)，选择合适版本（nginx/Windows-xxx）。

![img](http://dunwu.test.upcdn.net/snap/20180920181023092347.png)

（2）解压到本地

![img](http://dunwu.test.upcdn.net/snap/20180920181023092044.png)

（3）启动

下面以 C 盘根目录为例说明下：

```bash
cd C:
cd C:\nginx-0.8.54 start nginx
```

> 注：Nginx / Win32 是运行在一个控制台程序，而非 windows 服务方式的。服务器方式目前还是开发尝试中。

### Linux 安装

#### rpm 包方式（推荐）

（1）进入[下载页面](http://nginx.org/packages/)，选择合适版本下载。

```bash
$ wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

（2）安装 nginx rpm 包

nginx rpm 包实际上安装的是 nginx 的 yum 源。

```bash
$ rpm -ivh nginx-*.rpm
```

（3）正式安装 rpm 包

```bash
$ yum install nginx
```

（4）关闭防火墙

```bash
$ firewall-cmd --zone=public --add-port=80/tcp --permanent
$ firewall-cmd --reload
```

#### 源码编译方式

##### 下载源码

进入官网下载地址：http://nginx.org/en/download.html ，选择合适的版本下载（PS: 建议选择最新的稳定版本）。

这里我选择的是 1.16.1 版本：http://nginx.org/download/nginx-1.16.1.tar.gz

```bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

##### 安装编译工具及库

Nginx 源码的编译依赖于 gcc 以及一些库文件，所以必须提前安装。

（1）安装gcc环境
```bash
yum install gcc-c++
```
（2）安装PCRE库，用于解析正则表达式
```bash
yum install -y pcre pcre-devel
```

执行 `pcre-config --version` 命令。

（3）zlib压缩和解压缩依赖，
```bash
yum install -y zlib zlib-devel
```
（4）SSL 安全的加密的套接字协议层，用于HTTP安全传输，也就是https

```bash
yum install -y openssl openssl-devel
```

##### 编译源码并安装

（1）解压，需要注意，解压后得到的是源码，源码需要编译后才能安装
```bash
tar -zxvf nginx-1.16.1.tar.gz
```

（2）编译之前，先创建nginx临时目录，如果不创建，在启动nginx的过程中会报错

```bash
mkdir /var/temp/nginx -p
```

（3）在nginx目录，输入如下命令进行配置，目的是为了创建makefile文件

```bash
./configure --prefix=/usr/local/nginx --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_gzip_static_module --http-client-body-temp-path=/var/temp/nginx/client --http-proxy-temp-path=/var/temp/nginx/proxy --http-fastcgi-temp-path=/var/temp/nginx/fastcgi --http-uwsgi-temp-path=/var/temp/nginx/uwsgi --http-scgi-temp-path=/var/temp/nginx/scgi
```

- *注： 代表在命令行中换行，用于提高可读性*

- 配置命令：

  | 命令                          | 解释                                 |
  | :---------------------------- | :----------------------------------- |
  | –prefix                       | 指定nginx安装目录                    |
  | –pid-path                     | 指向nginx的pid                       |
  | –lock-path                    | 锁定安装文件，防止被恶意篡改或误操作 |
  | –error-log                    | 错误日志                             |
  | –http-log-path                | http日志                             |
  | –with-http_gzip_static_module | 启用gzip模块，在线实时压缩输出数据流 |
  | –http-client-body-temp-path   | 设定客户端请求的临时目录             |
  | –http-proxy-temp-path         | 设定http代理临时目录                 |
  | –http-fastcgi-temp-path       | 设定fastcgi临时目录                  |
  | –http-uwsgi-temp-path         | 设定uwsgi临时目录                    |
  | –http-scgi-temp-path          | 设定scgi临时目录                     |


（4）make编译并安装
```bash
make
make install
```

（5）关闭防火墙

```bash
$ firewall-cmd --zone=public --add-port=80/tcp --permanent
$ firewall-cmd --reload
```

（6） 启动 Nginx

安装成功后，进入sbin目录启动 nginx
```bash
./nginx
```
- 停止：`./nginx -s stop`
- 重新加载：`./nginx -s reload`

打开浏览器，访问虚拟机所处内网ip即可打开nginx默认页面，显示如下便表示安装成功：
![img](http://dunwu.test.upcdn.net/snap/20180920181016133223.png)

##### 注意事项:
- 如果在云服务器安装，需要开启默认的nginx端口：80
- 如果在虚拟机安装，需要关闭防火墙
- 本地win或mac需要关闭防火墙

#### Linux 开机自启动

Centos7 以上是用 Systemd 进行系统初始化的，Systemd 是 Linux 系统中最新的初始化系统（init），它主要的设计目标是克服 sysvinit 固有的缺点，提高系统的启动速度。Systemd 服务文件以 .service 结尾。

##### rpm 包方式

如果是通过 rpm 包安装的，会自动创建 nginx.service 文件。

直接用命令：

```bash
$ systemctl enable nginx.service
```

设置开机启动即可。

##### 源码编译方式

如果采用源码编译方式，需要手动创建 nginx.service 文件。

详情参考：[centos 7.x编写开机启动服务](../linux/linux-centos-start.md)

## Docker 安装

- 官网镜像：https://hub.docker.com/_/nginx/
- 下载镜像：`docker pull nginx`
- 启动容器：`docker run --name my-nginx -p 80:80 -v /data/docker/nginx/logs:/var/log/nginx -v /data/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx`
- 重新加载配置（目前测试无效，只能重启服务）：`docker exec -it my-nginx nginx -s reload`
- 停止服务：`docker exec -it my-nginx nginx -s stop` 或者：`docker stop my-nginx`
- 重新启动服务：`docker restart my-nginx`

## 脚本安装

> CentOS7 环境安装脚本：[软件运维配置脚本集合](https://github.com/dunwu/linux-tutorial/tree/master/codes/linux/soft)

**安装说明**

- 采用编译方式安装 Nginx, 并将其注册为 systemd 服务
- 安装路径为：`/usr/local/nginx`
- 默认下载安装 `1.16.0` 版本

**使用方法**

- 默认安装 - 执行以下任意命令即可：

```shell
curl -o- https://gitee.com/turnon/linux-tutorial/raw/master/codes/linux/soft/nginx-install.sh | bash
wget -qO- https://gitee.com/turnon/linux-tutorial/raw/master/codes/linux/soft/nginx-install.sh | bash
```

- 自定义安装 - 下载脚本到本地，并按照以下格式执行：

```bash
sh nginx-install.sh [version]
```

## Nginx 日志切割

现有的日志都会存在 `access.log` 文件中，但是随着时间的推移，这个文件的内容会越来越多，体积会越来越大，不便于运维人员查看，所以我们可以通过把这个大的日志文件切割为多份不同的小文件作为日志，切割规则可以以`天`为单位，如果每天有几百G或者几个T的日志的话，则可以按需以`每半天`或者`每小时`对日志切割一下。

### 手动日志切割

**具体步骤如下：**

1. 创建一个shell可执行文件：`cut_log.sh`，内容为：

```bash
#!/bin/bash
LOG_PATH="/var/log/nginx/"
RECORD_TIME=$(date -d "yesterday" +%Y-%m-%d+%H:%M)
PID=/var/run/nginx/nginx.pid
mv ${LOG_PATH}/access.log ${LOG_PATH}/access.${RECORD_TIME}.log
mv ${LOG_PATH}/error.log ${LOG_PATH}/error.${RECORD_TIME}.log

#向Nginx主进程发送信号，用于重新打开日志文件
kill -USR1 `cat $PID`
```

2. 为`cut_log.sh`添加可执行的权限：

```bash
chmod +x cut_log.sh
```

3. 测试日志切割后的结果:

```bash
./cut_log.sh
```

### 使用定时任务

1. 安装定时任务：

   ```bash
   yum install crontabs
   ```

2. `crontab -e` 编辑并且添加一行新的任务：

   ```bash
   */1 * * * * /usr/local/nginx/sbin/cut_log.sh
   ```

3. 重启定时任务：

   ```bash
   service crond restart
   systemctl restart crond.service
   ```

- 附：常用定时任务命令：

  ```bash
  service crond start         // 启动服务
  service crond stop          // 关闭服务
  service crond restart       // 重启服务
  service crond reload        // 重新载入配置
  crontab -e                  // 编辑任务
  crontab -l                  // 查看任务列表
  ```

### 定时任务表达式：

Cron表达式是，分为5或6个域，每个域代表一个含义，如下所示：

|          | 分   | 时   | 日   | 月   | 星期几 | 年（可选）       |
| :------- | :--- | :--- | :--- | :--- | :----- | :--------------- |
| 取值范围 | 0-59 | 0-23 | 1-31 | 1-12 | 1-7    | 2019/2020/2021/… |

### 常用表达式：

- 每分钟执行：

  ```
  */1 * * * *
  ```

- 每日凌晨（每天晚上23:59）执行：

  ```
  59 23 * * *
  ```

- 每日凌晨1点执行：

  ```
  0 1 * * *
  ```

## 使用 Gzip 压缩提升请求速度

```bash
# 开启 gzip 压缩功能，目的：提高传输效率，节约服务器带宽
gzip  on;
# 限制最小压缩，小于 1 字节文件不会压缩
gzip_min_length 1;
# 设置图片压缩级别（压缩比，文件越大，压缩越多，但是 CPU 使用会越多）
gzip_comp_level 3;
# 定义压缩文件的类型
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/json;
```

## root 与 alias

假如服务器路径为：/home/imooc/files/img/face.png

- root 路径完全匹配访问

配置的时候为：

```bash
location /imooc {
	root /home
}
```

用户访问的时候请求为：`url:port/imooc/files/img/face.png`

- alias 可以为你的路径做一个别名，对用户透明

配置的时候为：

```bash
location /hello {
	alias /home/imooc
}
```

用户访问的时候请求为：`url:port/hello/files/img/face.png`，如此相当于为目录`imooc`做一个自定义的别名。

## location 的匹配规则

- `空格`：默认匹配，普通匹配

  ```bash
  location / {
       root /home;
  }
  ```

- `=`：精确匹配

  ```bash
  location = /imooc/img/face1.png {
      root /home;
  }
  ```

- `~*`：匹配正则表达式，不区分大小写

  ```bash
  #符合图片的显示
  location ~* .(GIF|jpg|png|jpeg) {
      root /home;
  }
  ```

- `~`：匹配正则表达式，区分大小写

  ```bash
  #GIF必须大写才能匹配到
  location ~ .(GIF|jpg|png|jpeg) {
      root /home;
  }
  ```

- `^~`：以某个字符路径开头

  ```bash
  location ^~ /imooc/img {
      root /home;
  }
  ```

## 参考资料

- http://www.dohooe.com/2016/03/03/352.html?utm_source=tuicool&utm_medium=referral
- [nginx+keepalived实现nginx双主高可用的负载均衡](https://blog.51cto.com/kling/1253474)
- 每天定时为数据库备份：[腾讯云服务器 - 定时备份MariaDB/MySQL](https://www.cnblogs.com/leechenxiang/p/7110382.html) 