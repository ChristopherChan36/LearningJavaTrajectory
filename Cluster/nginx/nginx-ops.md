# Nginx 安装与配置

## 普通安装

### Windows安装（不推荐）

（1）进入[官方下载地址](https://nginx.org/en/download.html)，选择合适版本（nginx/Windows-xxx）。

![img](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-28-140222.png)

（2）解压到本地

![img](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-28-140306.png)

（3）启动

下面以 C 盘根目录为例说明下：

```bash
cd C:
cd C:\nginx-0.8.54 start nginx
```

> 注：Nginx / Win32 是运行在一个控制台程序，而非 windows 服务方式的。服务器方式目前还是开发尝试中。

### Linux 安装（推荐）

#### rpm 包方式

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

#### 源码编译方式（推荐）

##### 下载 Nginx 源码

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
（2）安装 PCRE 库，用于解析正则表达式
```bash
yum install -y pcre pcre-devel
```

执行 `pcre-config --version` 命令检测 PCRE 版本。

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
- 关闭：`./nginx -s stop`
- 重新加载：`./nginx -s reload`
- 检查 nginx.conf 配置是否正确：`./nginx -t`

打开浏览器，访问虚拟机所处内网ip即可打开nginx默认页面，显示如下便表示安装成功：
![img](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-28-140354.png)

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

## Nginx 初步配置

### nginx.conf 配置结构

nginx.conf 配置文件主要分为三部分：全局块、events 块和 https 块。

- main 全局配置
  - event 配置工作模式以及连接数
  - http http 模块相关配置
    - server 虚拟主机配置，可以配置多个
      - location 路由规则，表达式
      - upstream 集群，内网服务器

**Nginx配置语法：**

- 配置文件由指令和指令块构成
- 每条指令以分号（;）结尾，指令和参数间以空格符分隔
- 指令块以大括号{}将多条指令组织在一起
- include语句允许组合多个配置文件以提高可维护性
- 使用 # 添加注释
- 使用 $ 定义变量
- 部分指令的参数支持正则表达式

#### 全局块

全局配置部分用来配置对整个 server 都有效的参数。主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。 示例：

```nginx
# 设置worker进程的用户，指的linux中的用户，会涉及到nginx操作目录或文件的一些权限，默认为`nobody`
user root;
# worker进程工作数设置，一般来说CPU有几个，就设置几个，或者设置为N-1也行
worker_processes  1;
    
# nginx 日志级别`debug | info | notice | warn | error | crit | alert | emerg`，错误级别从左到右越来越大
# error_log logs/error.log info

# 设置nginx进程 pid
# pid        logs/nginx.pid;
```

#### events 块

events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

```nginx
# 设置工作模式
events {
  # 默认使用 epoll
  use epoll;
  # 设置每个worker进程的最大连接数，它决定了Nginx的并发能力
  worker_connections  1024;
}
```

#### http 块

这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。 需要注意的是：http 块也可以包括 http 全局块、server 块。

**http 全局块**

http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

```nginx
# http 是指令块，针对http网络传输的一些指令配置
http {
    # include 引入外部配置，提高可读性，避免单个配置文件过大
    include      mime.types;
    default_type  application/octet-stream;

    # 设定日志格式，`main`为定义的格式名称，如此 access_log 就可以直接使用这个变量了
    # 注释了日志格式的配置，使用默认
    ...

    # sendfile使用高效文件传输，提升传输性能。
    # 启用后才能使用`tcp_nopush`，是指当数据表累积一定大小后才发送，提高了效率。
    sendfile        on;
    tcp_nopush      on;

    # 重要参数，是一个请求完成之后还要保持连接多久，不是请求时间多久，
    # 目的是保持长连接，减少创建连接过程给系统带来的性能损耗
    keepalive_timeout  65;

    # gzip启用压缩，html/js/css压缩后传输会更快
    gzip    on;
}
```

**server 块**

这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。

每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。

而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。

- **全局 server 块**

  也被叫做『虚拟服务器』部分，它描述的是一组根据不同 server_name 指令逻辑分割的资源，这些虚拟服务器响应 HTTP 请求，因此都包含在http部分。最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

  ```nginx
  server {
    listen       80;
    #server_name也支持通配符，*.example.com、www.example.*、.example.com
    server_name  localhost;
    #charset koi8-r;
    #access_log  logs/host.access.log  main;
  ```

- **location 块**

  一个 server 块可以配置多个 location 块。

  这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称 （也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

  location 匹配是在 FIND_CONFIG 阶段进行的，我们需要掌握 location 的匹配规则和匹配顺序。

  **location 匹配规则**

  语法如下：`location [ = | ~ | ~* | ^~] uri{}`

  > **Tip** 注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。

  | 规则 | 匹配                                                         |
  | :--- | :----------------------------------------------------------- |
  | =    | 严格匹配。如果请求匹配这个 location，那么将停止搜索并立即处理此请求 |
  | ~    | 区分大小写匹配(可用正则表达式)                               |
  | ~*   | 不区分大小写匹配(可用正则表达式)                             |
  | !~   | 区分大小写不匹配                                             |
  | !~*  | 不区分大小写不匹配                                           |
  | ^~   | 前缀匹配                                                     |
  | @    | “@” 定义一个命名的location，使用在内部定向时                 |
  | /    | 通用匹配，任何请求都会匹配到（默认）                         |

  - 没有正则表达式的location被作为最佳的匹配，独立于含有正则表达式的location顺序；
  - **在配置文件中按照查找顺序进行正则表达式匹配**。在查找到第一个正则表达式匹配之后结束查找。由这个最佳的location提供请求处理。
  - = ：该修饰符使用精确匹配并且终止搜索。
  - ~：该修饰符使用区分大小写的正则表达式匹配。
  - ~*：该修饰符使用不区分大小写的正则表达式匹配。
  - ^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字 符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。

  **location 匹配顺序**

  - “=” 精准匹配，如果匹配成功，则停止其他匹配
  - 普通字符串指令匹配，优先级是从长到短(匹配字符越多，则选择该匹配结果)。匹配成功的location如果使用^~，则停止其他匹配（正则匹配）
  - 正则表达式指令匹配，按照配置文件里的顺序(从上到下)，成功就停止其他匹配
  - 如果正则匹配成功，使用该结果;否则使用普通字符串匹配结果

  有一个简单总结如下：

  > (location =) > (location 完整路径) > (location ^~ 路径) > (location ,* 正则顺序) > (location 部分起始路径) > (location /)

  即：

  > (精确匹配）> (最长字符串匹配，但完全匹配) >（非正则匹配）>（正则匹配）>（最长字符串匹配，不完全匹配）>（location通配）

  ![location 匹配顺序](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-18-145056.png)


### nginx.conf 详细配置

在前面搭建好 Nginx 环境后，编译的 Nginx 根路径为 /usr/local/nginx，那么对应的配置文件为 /usr/local/nginx/conf/nginx.conf ，直接用 cat 命令查看这里的配置文件内容（删除掉了原配置文件中的英文注释，并对主要配置项增加中文注释）：

```bash
# 设置worker进程的用户，指的linux中的用户，会涉及到nginx操作目录或文件的一些权限，默认为`nobody`
user root;
# worker进程工作数设置，一般来说CPU有几个，就设置几个，或者设置为N-1也行
worker_processes  1;

# nginx 日志级别`debug | info | notice | warn | error | crit | alert | emerg`，错误级别从左到右越来越大
# error_log logs/error.log info

# 设置nginx进程 pid
# pid        logs/nginx.pid;

# 指定进程可以打开的最大描述符：数目
# 工作模式与连接数上限
## 这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
# 这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
worker_rlimit_nofile 65535;

#################################  events  ###############################
# 设置工作模式
events {
		# 默认使用 epoll
		# 参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型
		use epoll;
    # 设置每个worker进程的最大连接数，它决定了Nginx的并发能力
    worker_connections  1024;
    
    #keepalive 超时时间
    keepalive_timeout 60;
    
    #客户端请求头部的缓冲区大小。
    client_header_buffer_size 4k;
    
    # 这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
    open_file_cache max=65535 inactive=60s;
    # 这个是指多长时间检查一次缓存的有效信息。
    open_file_cache_valid 80s;
    # open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
    open_file_cache_min_uses 1;
    
    # 语法:open_file_cache_errors on | off 默认值:open_file_cache_errors off 使用字段:http, server, location 这个指令指定是否在搜索一个文件是记录cache错误.
    open_file_cache_errors on;
}

##############################   http    ##################################
# http 是指令块，针对http网络传输的一些指令配置，利用它的反向代理功能提供负载均衡支持
http {
    # include 引入外部配置，提高可读性，避免单个配置文件过大
    include      mime.types;
		default_type  application/octet-stream;

    # 设定日志格式，`main`为定义的格式名称，如此 access_log 就可以直接使用这个变量了
    # 注释了日志格式的配置，使用默认
    ...

    # sendfile使用高效文件传输，提升传输性能。
    # 启用后才能使用`tcp_nopush`，是指当数据表累积一定大小后才发送，提高了效率。
    sendfile        on;
    tcp_nopush      on;

    # 重要参数，是一个请求完成之后还要保持连接多久，不是请求时间多久，
    # 目的是保持长连接，减少创建连接过程给系统带来的性能损耗
    keepalive_timeout  65;
    
    # FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    
    # gzip启用压缩，html/js/css压缩后传输会更快
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k;    #最小压缩文件大小
    gzip_buffers 4 16k;    #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2;     #压缩等级
    #压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_types text/plain application/x-javascript text/css application/xml;    
    gzip_vary on;

    #开启限制IP连接数的时候需要使用
    # limit_zone crawler $binary_remote_addr 10m;

		#负载均衡配置
    upstream lazyegg {
        # upstream的负载均衡，weight是权重，可以根据机器配置定义权重。
        # weigth参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;

        # nginx的upstream目前支持4种方式的负载均衡策略
        # 1.1、轮询（默认）
        # 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
        # 1.2、weight（加权轮询）
        # 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
        # 例如：
        # upstream bakend {
        #     server 192.168.0.14 weight=10;
        #     server 192.168.0.15 weight=10;
        # }
        # 2、ip_hash
        # 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
        # 例如：
        # upstream bakend {
        #     ip_hash;
        #     server 192.168.0.14:88;
        #     server 192.168.0.15:80;
        # }
        # 3、fair（第三方）
        # 按后端服务器的响应时间来分配请求，响应时间短的优先分配。
        # upstream backend {
        #     server server1;
        #     server server2;
        #     fair;
        # }
        # 4、url_hash（第三方）
        # 按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
        # 例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
        # upstream backend {
        #     server squid1:3128;
        #     server squid2:3128;
        #     hash $request_uri;
        #     hash_method crc32;
        # }

        # tips:
        # upstream bakend{#定义负载均衡设备的Ip及设备状态}{
        #     ip_hash;
        #     server 127.0.0.1:9090 down;
        #     server 127.0.0.1:8080 weight=2;
        #     server 127.0.0.1:6060;
        #     server 127.0.0.1:7070 backup;
        # }
        # 在需要使用负载均衡的server中增加 proxy_pass http://bakend/;

        # 每个设备的状态设置为:
        # 1.down表示单前的server暂时不参与负载
        # 2.weight为weight越大，负载的权重就越大。
        # 3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
        # 4.fail_timeout:max_fails次失败后，暂停的时间。
        # 5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

        # nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
        # client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
        # client_body_temp_path设置记录文件的目录 可以设置最多3层目录
        # location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
    }

    # server块配置，虚拟主机的配置
    server {
        # 监听80端口
        listen       80;
        # 域名可以有多个，用空格隔开
        server_name  localhost www.lazyegg.net;

        # 匹配url /，会在html目录下，访问index.html或index.htm文件
        location / {
            root   html;
            index  index.html index.htm;
        }
        
        # 对 "lazyegg.net" 启用反向代理
        location / {
            proxy_pass http://lazyegg.net; 
             
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             
            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;

            #允许客户端请求的最大单文件字节数
            client_max_body_size 10m;

            #缓冲区代理缓冲用户端请求的最大字节数，
            #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
            #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
            client_body_buffer_size 128k;

            #表示使nginx阻止HTTP应答代码为400或者更高的应答。
            proxy_intercept_errors on;

            #后端服务器连接的超时时间_发起握手等候响应超时时间
            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout 90;

            #后端服务器数据回传时间(代理发送超时)
            #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
            proxy_send_timeout 90;

            #连接成功后，后端服务器响应时间(代理接收超时)
            #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
            proxy_read_timeout 90;

            #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            #设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
            proxy_buffer_size 4k;

            #proxy_buffers缓冲区，网页平均在32k以下的设置
            #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
            proxy_buffers 4 32k;

            #高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k;

            #设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_file_write_size 64k;
        }
        
        #本地动静分离反向代理配置
        #所有jsp的页面均交由tomcat或resin处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }

        # 指定500 502 503 504出错的错误页面
        error_page   500 502 503 504  /50x.html;
            location = /50x.html {
            root   html;
        }
		}
}  
```

设定日志格式，`main`为定义的格式名称，如此 access_log 就可以直接使用这个变量了
![](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-04-153849.jpg)

| 参数名                | 参数意义                             |
| :-------------------- | :----------------------------------- |
| $remote_addr          | 客户端ip                             |
| $remote_user          | 远程客户端用户名，一般为：’-’        |
| $time_local           | 时间和时区                           |
| $request              | 请求的url以及method                  |
| $status               | 响应状态码                           |
| $body_bytes_send      | 响应客户端内容字节数                 |
| $http_referer         | 记录用户从哪个链接跳转过来的         |
| $http_user_agent      | 用户所使用的代理，一般来时都是浏览器 |
| $http_x_forwarded_for | 通过代理服务器来记录客户端的ip       |

### Nginx 日志切割

现有的日志都会存在 `access.log` 文件中，但是随着时间的推移，这个文件的内容会越来越多，体积会越来越大，不便于运维人员查看，所以我们可以通过把这个大的日志文件切割为多份不同的小文件作为日志，切割规则可以以`天`为单位，如果每天有几百G或者几个T的日志的话，则可以按需以`每半天`或者`每小时`对日志切割一下。

#### 手动日志切割

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

#### 使用定时任务

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

#### 定时任务表达式：

Cron表达式是，分为5或6个域，每个域代表一个含义，如下所示：

|          | 分   | 时   | 日   | 月   | 星期几 | 年（可选）       |
| :------- | :--- | :--- | :--- | :--- | :----- | :--------------- |
| 取值范围 | 0-59 | 0-23 | 1-31 | 1-12 | 1-7    | 2019/2020/2021/… |

#### 常用表达式：

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

### 使用 Gzip 压缩提升请求速度

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

### root 与 alias

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

### location 的匹配规则

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
- https://mp.weixin.qq.com/s/jA-6tDcrNgd-Wtncj6D6DQ
- [nginx+keepalived实现nginx双主高可用的负载均衡](https://blog.51cto.com/kling/1253474)
- 每天定时为数据库备份：[腾讯云服务器 - 定时备份MariaDB/MySQL](https://www.cnblogs.com/leechenxiang/p/7110382.html) 