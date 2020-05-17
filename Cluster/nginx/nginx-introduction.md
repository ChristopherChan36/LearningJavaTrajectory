# 深入浅出搞懂 Nginx

> **Nginx（engine x）**是一款开源、高性能、轻量级的 HTTP 和反向代理 Web 服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。

## 一、Nginx 简介

### 什么是 Nginx

**Nginx (engine x)** 是一款轻量级的 Web 服务器 、反向代理服务器及电子邮件（IMAP/POP3）代理服务器。

![NGINX 架构图](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/04/15859260628695.jpg)

上图基本上说明了当下流行的技术架构，其中 Nginx 类似于入口**网关**的作用。

### Nginx 使用场景

Nginx 的使用场景如下：

**HTTP 服务器**

Nginx 作为 Web 服务器能独立提供 Http 服务。另外，我们常常通过 Nginx 作为**静态资源**服务器来访问服务器上的静态资源，比如对于最新热门的**前后端分离架构**，前端打好包后直接放到某个地址，在 Nginx 配置后可以通过 Nginx 来访问主机上的前端页面。

**反向代理**

反向代理（Reverse Proxy）方式是指以代理服务器来接受 Internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。这样的好处是，将不暴露内部的服务地址，只统一使用一个公共出口，通过 URI 匹配转发到不同的内部服务处理请求。

**负载均衡**

负载均衡也是 Nginx 的一个高频使用场景，对于下游存在的多个相同服务，可以将请求采用某种策略（随机、轮询、权重）发到相应的服务处理。这样由于多个相同服务的存在，可以实现高可用功能，在一个服务不可用时，Nginx 会自动发现并将其剔出服务集群，将请求转发给正常的服务进行处理。

**第三方插件**

基于第三方插件，Nginx 可以完成各种各样复杂的功能，全方位满足程序员的想法。比如在 Nginx 中引入 lua 模块，可以实现对 Http 请求更细粒度的限制，包括限速、限流、校验认证等等。后续，在 Nginx 上发展出来的 OpenResty 已经应用到了微服务网关方向。

### Web 服务器的市场情况

[Netcraft公司官网](https://news.netcraft.com/) 每月公布的全球 Web 服务器调查报告“Web Server Survey”是当前人们了解全球网站数量以及服务器市场分额情况的主要参考依据，2020 年 4 月份的报告目前已经发布，我们来一睹为快。

![](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-04-124447.png)

![image-20200504204528143](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-04-124528.png)

可以明显看到，**Nginx 已经确确实实处于 Web 服务器市场的领先地位，成功超过了老大哥 Apache，千年老二至此翻身当上了大哥。**

### 什么是反向代理

经常听人说到一些术语，比如反向代理，那么什么是反向代理呢？什么又是正向代理？

**正向代理**

![](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/04/15859263037103.jpg)

例如：由于防火墙的原因，我们并不能直接访问谷歌，那么我们可以借助 VPN 来实现，这就是一个简单的正向代理的例子。这里你能够发现，正向代理『代理』的是客户端，而且客户端是知道目标的，而目标是不知道客户端是通过 VPN 访问的。

**反向代理**

反向代理（Reverse Proxy）方式是指以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

![反向代理](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/15856562838826.jpg)

当我们在外网访问百度的时候，其实会进行一个转发，代理到内网去，这就是所谓的反向代理，即反向代理『代理』的是服务器端，而且这一个过程对于客户端而言是透明的。

### 什么是负载均衡 ？

**Load balancing，即负载均衡，是一种计算机技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器或其他资源中分配负载，以达到最优化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。**

更多关于反向代理以及负载均衡点击[这里](nginx-balance.md)查看。

## 二、Nginx 入门

### Nginx 常用命令

nginx 的使用比较简单，就是几条命令。

常用到的命令如下：

```bash
nginx -s stop       快速关闭Nginx，可能不保存相关信息，并迅速终止web服务。
nginx -s quit       平稳关闭Nginx，保存相关信息，有安排的结束web服务。
nginx -s reload     因改变了Nginx相关配置，需要重新加载配置而重载。
nginx -s reopen     重新打开日志文件。
nginx -c filename   为 Nginx 指定一个配置文件，来代替缺省的。
nginx -t            不运行，仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
nginx -v            显示 nginx 的版本。
nginx -V            显示 nginx 的版本，编译器版本和配置参数。
```

### Nginx 的 Master-Worker 模式

Nginx 的进程模型采用了 Master/Workers 进程池的机制，即通常情况下，Nginx 会启动一个 Master 进程（当然，也可以无 master 进程）和多个 Worker 进程对外提供服务。Master 进程是监控进程，本身并不处理具体的 TCP 和 HTTP 请求，只负责接受 UNIX 信号，管理 Worker 进程，类似于工地的包工头。

Worker 进程是比较累的，负责处理客户端的连接请求，它充分利用了 Linux 系统中的 epoll、kqueue 等机制，高效处理 TCP 和 HTTP 请求，利用这些特点，**Nginx 充分挖掘了服务器的潜能，让服务器更快响应用户请求。**

一般情况下，10000 个非活跃的 HTTP Keep-Alive 连接在 Nginx 中仅仅消耗 2.5M 内存，这是 Nginx 支持高并发连接的基础，体现了 Nginx 高性能的特点。另外，由于官方提供的模块都非常稳定，每个 Worker 进程都相对独立，Woker 进程出错时，Master 进程会立马感知到并快速拉起新的 Worker 子进程不间断提供服务，保证服务的稳定性。

**nginx的进程模型**

![Master-Worker 模式](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-04-03-160633.png)

启动Nginx后，其实就是在80端口启动了Socket服务（master）进行监听，如图所示，Nginx涉及Master进程和Worker进程。

![Nginx 的 Master-Worker 模式](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-04-140201.png)

#### Master进程的作用

**读取并验证配置文件nginx.conf；管理worker进程；**

#### Worker进程的作用

**每一个Worker进程都维护一个线程（避免线程切换），处理连接和请求；注意Worker进程的个数由配置文件决定，一般和CPU个数相关（有利于进程切换），配置几个就有几个Worker进程。**

> 详细安装方法及相应配置详解请参考：[Nginx 运维](nginx-ops.md)

## 三、Nginx 进阶与实战

### 解决跨域

web 领域开发中，经常采用前后端分离模式。这种模式下，前端和后端分别是独立的 web 应用程序，例如：后端是 Java 程序，前端是 React 或 Vue 应用。

各自独立的 web app 在互相访问时，势必存在跨域问题。解决跨域问题一般有两种思路：

1. **CORS**

   在后端服务器设置 HTTP 响应头，把你需要允许访问的域名加入 `Access-Control-Allow-Origin` 中。

2. **jsonp**

   把后端根据请求，构造 json 数据，并返回，前端用 jsonp 跨域。

这两种思路，本文不展开讨论。

需要说明的是，nginx 根据第一种思路，也提供了一种解决跨域的解决方案。

3. **Nginx** 

举例：www.helloworld.com 网站是由一个前端 app ，一个后端 app 组成的。前端端口号为 9000， 后端端口号为 8080。

前端和后端如果使用 http 进行交互时，请求会被拒绝，因为存在跨域问题。来看看，nginx 是怎么解决的吧：

首先，在 enable-cors.conf 文件中设置 cors ：

```nginx
# allow origin list
set $ACAO '*';

# set single origin
if ($http_origin ~* (www.helloworld.com)$) {
  set $ACAO $http_origin;
}

if ($cors = "trueget") {
	add_header 'Access-Control-Allow-Origin' "$http_origin";
	add_header 'Access-Control-Allow-Credentials' 'true';
	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
	add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
}

if ($request_method = 'OPTIONS') {
  set $cors "${cors}options";
}

if ($request_method = 'GET') {
  set $cors "${cors}get";
}

if ($request_method = 'POST') {
  set $cors "${cors}post";
}
```

接下来，在你的服务器中 `include enable-cors.conf` 来引入跨域配置：

```nginx
# ----------------------------------------------------
# 此文件为项目 nginx 配置片段
# 可以直接在 nginx config 中 include（推荐）
# 或者 copy 到现有 nginx 中，自行配置
# www.helloworld.com 域名需配合 dns hosts 进行配置
# 其中，api 开启了 cors，需配合本目录下另一份配置文件
# ----------------------------------------------------
upstream front_server{
  server www.helloworld.com:9000;
}
upstream api_server{
  server www.helloworld.com:8080;
}

server {
  listen       80;
  server_name  www.helloworld.com;

  location ~ ^/api/ {
    include enable-cors.conf;
    proxy_pass http://api_server;
    rewrite "^/api/(.*)$" /$1 break;
  }

  location ~ ^/ {
    proxy_pass http://front_server;
  }
}
```

到此，就完成了。

###  Nginx 防盗链配置

#### 什么是盗链?

百度百科的解释如下:

> 盗链是指服务提供商自己不提供服务的内容，通过技术手段绕过其它有利益的最终用户界面（如广告），直接在自己的网站上向最终用户提供其它服务提供商的服务内容，骗取最终用户的浏览和点击率。受益者不提供资源或提供很少的资源，而真正的服务提供商却得不到任何的收益。

盗链在如今的互联网世界无处不在，盗图，盗视频、盗文章等等，都是通过获取正规网站的图片、视频、文章等的 url 地址，直接放到自己网站上使用而未经授权。 Nginx 在代理这类静态资源(图片、视频、文章等)时，可以通过配置实现防盗连的功能。

#### 如何防盗链？

前面介绍到，盗链是直接使用正规网站保存图片、视频等的 URL 以获取相应的资源。最简单的防盗想法就是根据客户端请求资源时所携带的一些关键信息来验证请求的合法性，比如客户端 IP、请求 URL 中携带的 referer，如果不合法则直接拒绝请求。此外，由于这些基础信息都可以伪造，因此这样的基础手段也不一定安全。此外，还有登录认证、使用 cookie 等其他防盗连手段。另外，针对特定场景，比如流媒体直播中还有更为高级的防盗手段包括时间戳防盗链、swf 防盗链、回源鉴权防盗链等。

#### Nginx中防盗链配置

##### refer模块防盗

Nginx 用于实现防盗链功能的模块为 refer 模块,其依据的原理是: 如果网站盗用了你的图片，那么用户在点击或者查看这个盗链内容时，发送 http 请求的头部中的 referer 字段将为该盗版网站的 url。这样我们通过获取这个头部信息，知道 http 发起请求的页面，然后判断这个地址是否是我们的合法页面，不是则判断为盗链。Nginx 的 referer 模块中有3个指令，用法分别如下：

```nginx
Syntax:	referer_hash_bucket_size size;
Default: referer_hash_bucket_size 64;
Context: server, location

Syntax:	referer_hash_max_size size;
Default: referer_hash_max_size 2048;
Context: server, location

Syntax:	valid_referers none | blocked | server_names | string ...;
Default: —
Context: server, location
```

最重要的是 valid_referers 指令，它后面可以带上多个参数，表示多个 referer 头都是有效的。它的参数形式有:

- none: 允许缺失 referer 头部的请求访问
- blocked: 有 referer 这个字段，但是其值被防火墙或者是代理给删除了
- server_names: 若 referer 中的站点域名和 server_names 中的某个域名匹配，则允许访问
- 任意字符或者正则表达式

**Nginx 会通过查看 referer 字段和 valid_referers 后面的 referer 列表进行匹配，如果匹配到了就将内置的变量$invalid_referer值设置为0，否则设置该值为1**

这样一个简单的 Nginx 防盗链配置如下:

```bash
...
    location / {
    #对源站点验证
       valid_referers none blocked *.domain.pub www.domain.com/nginx server_names ~\.baidu\.;
       #非法引入会进入下方判断
       if ($invalid_referer) {
          return 403;
       }
       return 200 "valid\n";
   }
...
```

##### secure_link模块防盗

前面这种简单检查 referer 头部值的防盗链方法过于脆弱，盗用者很容易通过伪造 referer 的值轻而易举跳过防盗措施。在 Nginx 中有一种更为高级的防盗方式，即基于 secure_link 模块，该模块能够检查请求链接的权限以及是否过期，多用于下载服务器防盗链。这个模块默认未编译进 Nginx，需要在源码编译时候使用 `--with-secure_link_module` 添加。

该模块的通过验证 URL 中的哈希值的方式防盗链。它的防盗过程如下：

- 由服务器或者 Nginx 生成安全的加密后的 URL, 返回给客户端;
- 客户端使用安全的 URL 访问 Nginx，获取图片等资源，由 Nginx 的 secure_link 变量判断是否验证通过;

secure_link 模块中总共有3个指令，其格式和说明分别如下：

```nginx
Syntax:	secure_link expression;
Default: —
Context: http, server, location

Syntax:	secure_link_md5 expression;
Default: —
Context: http, server, location

Syntax:	secure_link_secret word;
Default: —
Context: location
```

通过配置 secure_link, secure_link_md5 指令，可实现对链接进行权限以及过期检查判断的功能。

和 referer 模块中的 $invalid_referer 变量一样，secure_link 模块也是通过内置变量 KaTeX parse error: Expected 'EOF', got '判' at position 14: secure\_link 判̲断验证是否通过。secure_link 的值有如下三种情况：

- 空字符串: 验证不通过
- 0: URL 过期
- 1: 验证通过

通常使用这个模块进行 URL 校验，我们需要考虑的是如何生成合法的 URL ？另外，需要在 Nginx 中做怎样的配置才可以校验这个 URL？

对于第一个问题，生成合法的 URL 和 指令 secure_link_md5 有关。例如:

```shell
secure_link_md5 "$secure_link_expires$uri$remote_addr secret";
```

如果 Nginx 中secure_link_md5 是上述配置，那么生成合法 url 的命令如下:

```shell
# 2020-02-05 21:00:00 转换成时间戳为1580907600
echo -n '1580907600/test.png127.0.0.1 secret' | \
    openssl md5 -binary | openssl base64 | tr +/ -_ | tr -d =
```

通过上述命令，我们得到了一个 md5 值:cPnjBG9bAZvY_jbPOj13mA，这个非常重要。接下来，构造合的 URL 和指令 secure_link 相关。如果 secure_link 指令的配置如下:

```shell
secure_link $arg_md5,$arg_expires;
```

那么我们的请求的 url 中必须带上 md5 和 expires 参数，例如:

```shell
http://180.76.152.113:9008/test.png?md5=cPnjBG9bAZvY_jbPOj13mA&expires=1580907600
```

对于 Nginx 中的校验配置示例如下:

```shell
location ~* .(gif|jpg|png|swf|flv|mp4)$  {
    secure_link $arg_md5,$arg_expires;
    secure_link_md5 "$secure_link_expires$uri$remote_addr secret";

    # 空字符串，校验不通过
    if ($secure_link = "") {
        return 403;
    }

    # 时间过期
    if ($secure_link = "0") {
        return 410 "URL过期，请重新生成";
    }

    root /root/test;
}
```

在 Nginx 的配置中，除了前面提到的 secure_link 和 secure_link_md5 指令外，我们对通过校验和校验失败的情况进行了处理。接下来请看实验部分。

#### 案例实战

##### refer 模块防盗链测试

在 nginx.conf 中加入如下防盗配置:

```shell
...
http {
    ...
    server {
       listen 9008;

       location / {
           valid_referers none blocked *.domain.pub www.domain.com/nginx server_names ~\.baidu\.;
           if ($invalid_referer) {
              return 403;
           }
           return 200 "valid\n";
       }
    }
    ...
}
...
```

重新加载或者启动 Nginx 后，我们进行如下操作:

```bash
[shen@shen Desktop]$ curl -H 'referer: http://www.domain.com/test' http://180.76.152.113:9008 
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.17.6</center>
</body>
</html>
[shen@shen Desktop]$ curl -H 'referer: http://www.domain.com/nginx' http://180.76.152.113:9008 
valid
[shen@shen Desktop]$ curl -H 'referer: ' http://180.76.152.113:9008 
valid
[shen@shen Desktop]$ curl http://180.76.152.113:9008 
valid
[shen@shen Desktop]$ curl -H 'referer: http://www.domain.pub/test' http://180.76.152.113:9008 
valid
```

第一个 http 请求 referer 的值存在，但是没有匹配后面的域名，所以返回403。其余的请求中 referer 值要么不存在，要么没有这个头部，要么匹配了后面的域名正则表达，都通过了 referer 校验，所以都返回 “valid” 字符串。我们通过构造不同的 referer 头部字段成功的绕过了 Nginx 的referer 模块校验，也说明了这种防盗的方式极不靠谱。

##### secure_link 防盗链测试

我们准备一个静态图片, 名为 test.png，放到搭建了 Nginx 的服务器上，全路径为 /root/test/test.png。
我们准备 Nginx 配置如下:

```shell
...
http {
    ...
    server {
       listen  8000;

       location / {
           # return 200 "$remote_addr";
           root /root/test;
       }
    }

    server {
       listen 8001;

       location ~* .(jpg|png|flv|mp4)$  {
          secure_link $arg_md5,$arg_expires;
          secure_link_md5 "$secure_link_expires$uri$remote_addr secret";

          # 空字符串，校验不通过
          if ($secure_link = "") {
             return 403;
          }

          # 时间过期
          if ($secure_link = "0") {
             return 410;
          }

          # 校验通过，访问对的静态资源
          root /root/test;
       }
    }
}
...
```

首先，在浏览器上访问8000端口我们可以获取对应的 $remote_addr 变量值(打开 return 的注释配置)，结果为103.46.244.69， 这是客户端请求时的对外 IP。访问浏览器上访问8000端口，URI=/test.png， 可以看到这个静态图片。

![图片描述](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-12-153349.png)

接下来，我们在访问8001端口，URI=/test.png时，可以发现返回403页面，说明安全模块生效。

![图片描述](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-12-153400.png)

当前时间为2020年02月05日晚上9点半，我们找一个过期时间晚上10点，得到相应的时间戳为1580911200。按照 secure_link_md5 指令格式，使用如下 shell 命令生成 md5 值:

```shell
[shen@shen Desktop]$ echo -n '1580911200/test.png103.46.244.69 secret' | openssl md5 -binary | openssl base64 | tr +/ -_ | tr -d =
KnJx3J6fN_0Qc1W5TqEVXw
```

这样可以得到我们的安全访问 URL 为:

```shell
# 访问静态资源test.png的安全URL为:
http://180.76.152.113:8001/test.png?md5=KnJx3J6fN_0Qc1W5TqEVXw&expires=1580911200
```

再次到浏览器上访问时候，我就可以看到静态图片了。

![图片描述](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-12-153424.png)

此外，我们还可以等到10点之后，测试过期后的结果。在过期之后再用这个 URL 访问时无法查看图片，而且返回的是 410 的状态码，这说明 Nginx 成功检测到这个密钥值已经过期。

#### 小结

一般的 Nginx 防盗链手段都是通过 referer 字段来判断请求的来源地，由此去判定请求是否合法。但是该字段容易伪造，所以很少用该方法实现防盗功能。而Nginx 的 secure_link 模块主要是使用 hash 算法加密方式，一般用于图片、视频下载，生成下载 URL，安全性高。此外，我们也可以使用一些第三方的模块增强 Nginx 的防盗链功能，比如常用的第三放模块ngx_http_accesskey_module 可用于实现文件下载的防盗功能。

### 负载均衡

前面的例子中，代理仅仅指向一个服务器。

但是，网站在实际运营过程中，大部分都是以集群的方式运行，这时需要使用负载均衡来分流。

nginx 也可以实现简单的负载均衡功能。

![负载均衡](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/15856563358077.jpg)

假设这样一个应用场景：将应用部署在 192.168.1.11:80、192.168.1.12:80、192.168.1.13:80 三台 linux 环境的服务器上。网站域名叫 www.helloworld.com，公网 IP 为 192.168.1.11。在公网 IP 所在的服务器上部署 nginx，对所有请求做负载均衡处理（下面例子中使用的是加权轮询策略）。

nginx.conf 配置如下：

```nginx
http {
    #设定负载均衡的服务器列表
    upstream load_balance_server {
        #weigth参数表示权值，权值越高被分配到的几率越大
        server 192.168.1.11:80   weight=5;
        server 192.168.1.12:80   weight=1;
        server 192.168.1.13:80   weight=6;
    }
   #HTTP服务器
   server {
        #侦听80端口
        listen       80;
        #定义使用www.xx.com访问
        server_name  www.helloworld.com;

        #对所有请求进行负载均衡请求
        location / {
            root        /root;                 #定义服务器的默认网站根目录位置
            index       index.html index.htm;  #定义首页索引文件的名称
            #请求转向load_balance_server 定义的服务器列表
            proxy_pass  http://load_balance_server ;

            #以下是一些反向代理的配置(可选择性配置)
            #proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_connect_timeout 90;          #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_send_timeout 90;             #后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90;             #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k;              #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k;               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
            proxy_busy_buffers_size 64k;       #高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k;    #设定缓存文件夹大小，大于这个值，将从upstream服务器传

            client_max_body_size 10m;          #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k;      #缓冲区代理缓冲用户端请求的最大字节数
        }
    }
}
```

#### 负载均衡策略

Nginx 提供了多种负载均衡策略，让我们来一一了解一下：

##### 轮询（默认）

```nginx
upstream bck_testing_01 {
  # 默认所有服务器权重为 1
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

##### 加权轮询

```nginx
upstream bck_testing_01 {
  server 192.168.250.220:8080   weight=3
  server 192.168.250.221:8080              # default weight=1
  server 192.168.250.222:8080              # default weight=1
}
```

##### IP Hash

`ip_hash` 可以保证用户访问可以请求到上游服务中的固定的服务器，前提是用户ip没有发生更改。
使用ip_hash的注意点：
不能把后台服务器直接移除，只能标记`down`.

> If one of the servers needs to be temporarily removed, it should be marked with the down parameter in order to preserve the current hashing of client IP addresses.

```nginx
upstream bck_testing_01 {

  ip_hash;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080

}
```

##### 最少连接

根据最少的连接数去分配请求，以此保证服务器分配的均衡。

```nginx
upstream bck_testing_01 {
  least_conn;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

##### 加权最少连接

```nginx
upstream bck_testing_01 {
  least_conn;

  server 192.168.250.220:8080   weight=3
  server 192.168.250.221:8080              # default weight=1
  server 192.168.250.222:8080              # default weight=1
}
```

##### 普通 Hash（URL）

根据每次请求的url地址，hash后访问到固定的服务器节点

```nginx
upstream bck_testing_01 {

  hash $request_uri;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080

}
```

##### 一致性 Hash 算法

详细参考: [分布式系统中一致性哈希算法](https://www.cnblogs.com/christopherchan/p/11410108.html)

#### upstream 指令参数

- max_conns

  限制每台server的连接数，用于保护避免过载，可起到限流作用。
  测试参考配置如下：

  ```nginx
  # worker进程设置1个，便于测试观察成功的连接数
  worker_processes  1;
  
  upstream tomcats {
          server 192.168.1.173:8080 max_conns=2;
          server 192.168.1.174:8080 max_conns=2;
          server 192.168.1.175:8080 max_conns=2;
  }
  ```

- slow_start

  ***商业版，需要付费*** ，服务器慢慢的加入到集群中，便于运维监控
  配置参考如下：

  ```nginx
  upstream tomcats {
          server 192.168.1.173:8080 weight=6 slow_start=60s;
  #       server 192.168.1.190:8080;
          server 192.168.1.174:8080 weight=2;
          server 192.168.1.175:8080 weight=2;
  }
  ```

  注意

  - 该参数不能使用在`hash`和`random load balancing`中。
  - 如果在 upstream 中只有一台 server，则该参数失效。

- down / backup

  `down` 用于标记服务节点不可用：

  ```nginx
  upstream tomcats {
          server 192.168.1.173:8080 down;
  #       server 192.168.1.190:8080;
          server 192.168.1.174:8080 weight=1;
          server 192.168.1.175:8080 weight=1;
  }
  ```

  `backup`表示当前服务器节点是备用机，只有在其他的服务器都宕机以后，自己才会加入到集群中，被用户访问到：

  ```nginx
  upstream tomcats {
          server 192.168.1.173:8080 backup;
  #       server 192.168.1.190:8080;
          server 192.168.1.174:8080 weight=1;
          server 192.168.1.175:8080 weight=1;
  }
  ```

  注意

  - `backup`参数不能使用在`hash`和`random load balancing`中。

- max_fails / fail_timeout

  `max_fails`：表示失败几次，则标记server已`宕机`，剔出上游服务。
  `fail_timeout`：表示失败的重试时间。
  假设目前设置如下：

  ```nginx
  upstream tomcats {
          server 192.168.1.173:8080 max_fails=2 fail_timeout=15s;
  #       server 192.168.1.190:8080;
          server 192.168.1.174:8080 weight=1;
          server 192.168.1.175:8080 weight=1;
  }
  ```

  则代表在15秒内请求某一server失败达到2次后，则认为该server已经挂了或者宕机了，随后再过15秒，这15秒内不会有新的请求到达刚刚挂掉的节点上，而是会请求到正常运作的server，15秒后会再有新请求尝试连接挂掉的server，如果还是失败，重复上一过程，直到恢复。

- Keepalived 提高吞吐量

  `keepalived`： 设置长连接处理的数量
  `proxy_http_version`：设置长连接http版本为1.1
  `proxy_set_header`：清除connection header 信息

  ```nginx
  upstream tomcats {
  #       server 192.168.1.173:8080 max_fails=2 fail_timeout=1s;
          server 192.168.1.190:8080;
  #       server 192.168.1.174:8080 weight=1;
  #       server 192.168.1.175:8080 weight=1;
          keepalive 32;
  }
  
  server {
          listen       80;
          server_name  www.tomcats.com;
  
          location / {
              proxy_pass  http://tomcats;
              proxy_http_version 1.1;
              proxy_set_header Connection "";
          }
  }
  ```

详细指令参数参考官方文档：http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html

### Nginx 缓存

#### Nginx 中的缓存介绍

由于 Nginx 是在网站的所有其他后台服务的最前线，它接收的请求和流量是后台服务的数倍甚至数十倍之多。因此，用好 Nginx 的缓存功能对于大型网站而言至关重要。Nginx 中的缓存功能优势如下：

- 缓存在nginx端，提升所有访问到nginx这一端的用户
- 有效降低上游服务器的负载
- 减少上游服务器之间的流量消耗

Nginx 的 Web 缓存服务主要由 `proxy_cache` 相关指令集和 `fastcgi_cache` 相关指令集构成，前者用于反向代理时，对后端内容源服务器进行缓存，后者主要用于对 FastCGI 的动态程序进行缓存。两者的功能基本上一样。强大的缓存功能也成为了 Nginx 吸引众多用户的重要因素之一。

#### Nginx中缓存指令

1. **expires指令**

Nginx 中的 `expires` 指令通过控制 HTTP 相应中的『Expires』 和 『Cache-Control』的头部值，达到控制浏览器缓存时间的效果。指令格式如下：

```nginx
Syntax:	expires [modified] time;
expires epoch | max | off;
Default:	
expires off;
Context: http, server, location, if in location
```

Nginx 中的时间单位有s(秒), m(分), h(小), d(天)。指令参数说明:

- epoch: 指定"Expires"的值为1, 即 January,1970,00:00:01 GMT;
- max: 指定"Expires"的值为31 December2037 23:59:59GMT, "Cache-Control"的值为10年;
- -1：指定"Expires"的值为当前服务器时间-1s，即永远过期;
  off：不修改"Expires"和"Cache-Control"的值
- time 中出现@表示具体的时间，比如@18h30m表示的是下午6点半;

官方的示例如下:

```nginx
expires    24h;       		# 24小时过期
expires    modified +24h; 
expires    @24h;      		# 24 点过期
expires    0;         		# 不缓存，立即过期
expires    -1;        		# 永不过期
expires    epoch;     		# 不设置缓存
expires    $expires;
```

2. **proxy 模块中的 cache 相关指令**

Nginx 的 proxy 模块中定义了许多和 cache 相关的模块，这是配置 http 请求代理的缓存功能。

通常情况下，我们使用 proxy_cache 指令开启 Nginx 缓存功能，用 proxy_cache_path 指令来设置缓存的路径和其他配置。两个指令的用法如下：

```nginx
Syntax:	proxy_cache zone | off;
Default: proxy_cache off;
Context: http, server, location

Syntax:	proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default: —
Context: http\
```

proxy_cache_path 指令中有较多的参数，部分重要参数说明如下:

- **proxy_cache_path: 定义缓存目录;**
- **keys_zone: 设置共享内存以及占用空间大小**
  - name: 共享内存名
  - size: 共享内存大小
- **max_size: 设置最大的缓存文件大小**
- **inactive 超过此时间则被清理**
- levels: 定义缓存路径的目录等级，最多3级
- **use_temp_path:临时目录，使用后会影响nginx性能**
  - on: 使用proxy_temp_path定义的目录
  - off:

```nginx
# proxy_cache_path 设置缓存目录
#       keys_zone 设置共享内存以及占用空间大小
#       max_size 设置缓存大小
#       inactive 超过此时间则被清理
#       use_temp_path 临时目录，使用后会影响nginx性能
proxy_cache_path /usr/local/nginx/upstream_cache keys_zone=mycache:5m max_size=1g inactive=1m use_temp_path=off;
```

其余的重要的缓存指令有:

- proxy_cache_key: 配置缓存的关键字，格式如下:

```nginx
Syntax:	proxy_cache_key string;
Default: proxy_cache_key $scheme$proxy_host$request_uri;
Context: http, server, location
```

示例：

```nginx
proxy_cache_key "$host$request_uri $cookie_user";
```

- proxy_cache_valid: 配置缓存什么样的响应，缓存多长时间。注意，如果只设置了缓存时间，只缓存只针对相应码200, 301和302的请求 。格式如下:

```nginx
Syntax:	proxy_cache_valid [code ...] time;
Default: —
Context: http, server, location
```

示例：

```nginx
proxy_cache_valid 200 302 10m;
proxy_cache_valid 404      1m;

# 只设置了缓存时间，只对200，301和302有效
proxy_cache_valid 5m;

proxy_cache_valid 200 302 10m;
proxy_cache_valid 301      1h;

# any表示所有相应码
proxy_cache_valid any      1m;
```

- proxy_cache_methods: 对哪种 method 的请求使用缓存返回响应。

```nginx
Syntax:	proxy_cache_methods GET | HEAD | POST ...;
Default: proxy_cache_methods GET HEAD;
Context: http, server, location
```

#### Nginx 缓存实战案例

准备好 proxy_cache 缓存相关的配置，如下:

```nginx
    # 定义上游服务器
    upstream backends {
        server 127.0.0.1:8000;
        server 127.0.0.1:8001;
        server 127.0.0.1:8002;
    }
    
    # proxy_cache_path 指令
    proxy_cache_path /root/test/cache levels=1:2 keys_zone=nginx_cache:10m max_size=10g inactive=60m use_temp_path=off;
    
    server {
       listen  80;

       location / {
          proxy_pass http://backends;
          proxy_cache nginx_cache;
          # 状态码为200和301的缓存1分钟
          proxy_cache_valid 200 301 1m;
          # 其余的缓存10分钟
          proxy_cache_valid any 10m;
          # response响应的头信息中定义缓存的状态（有没有命中）
          proxy_cache_key "$host$uri$is_args$args";
          expires 1d;
          proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
          proxy_no_cache $http_pragma    $http_authorization;
          # add_header 响应添加缓冲命中结果
          add_header Nginx-Cache "$upstream_cache_status";
       }
}
```

### Nginx配置HTTPS域名证书

#### 安装SSL模块

要在nginx中配置https，就必须安装ssl模块，也就是: `http_ssl_module`。

- 进入到nginx的解压目录： /home/software/nginx-1.16.1

- 新增ssl模块(原来的那些模块需要保留)

  ```nginx
  ./configure \
  --prefix=/usr/local/nginx \
  --pid-path=/var/run/nginx/nginx.pid \
  --lock-path=/var/lock/nginx.lock \
  --error-log-path=/var/log/nginx/error.log \
  --http-log-path=/var/log/nginx/access.log \
  --with-http_gzip_static_module \
  --http-client-body-temp-path=/var/temp/nginx/client \
  --http-proxy-temp-path=/var/temp/nginx/proxy \
  --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
  --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
  --http-scgi-temp-path=/var/temp/nginx/scgi  \
  --with-http_ssl_module
  ```

- 编译和安装

  ```bash
  make
  make install
  ```

#### 配置HTTPS

- 把ssl证书 `*.crt` 和 私钥 `*.key` 拷贝到`/usr/local/nginx/conf`目录中。

- 新增 server 监听 443 端口：

  ```nginx
  server {
      listen       443;
      server_name  www.imoocdsp.com;
  
      # 开启ssl
      ssl     on;
      # 配置ssl证书
      ssl_certificate      1_www.imoocdsp.com_bundle.crt;
      # 配置证书秘钥
      ssl_certificate_key  2_www.imoocdsp.com.key;
  
      # ssl会话cache
      ssl_session_cache    shared:SSL:1m;
      # ssl会话超时时间
      ssl_session_timeout  5m;
  
      # 配置加密套件，写法遵循 openssl 标准
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
      ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
      ssl_prefer_server_ciphers on;
      
      location / {
          proxy_pass http://tomcats/;
          index  index.html index.htm;
      }
   }
  ```

#### reload nginx

```bash
./nginx -s reload
```

> 腾讯云Nginx配置https文档地址：https://cloud.tencent.com/document/product/400/35244

## 资源

- [Nginx 的中文维基](http://tool.oschina.net/apidocs/apidoc?api=nginx-zh)
- [Nginx 开发从入门到精通](http://tengine.taobao.org/book/index.html)
- [nginx-admins-handbook](https://github.com/trimstray/nginx-admins-handbook)
- [nginxconfig.io](https://nginxconfig.io/) - 一款 Nginx 配置生成器
