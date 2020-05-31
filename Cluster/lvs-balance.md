#  负载均衡 LVS 详解

> 在业务初期，我们一般会先使用单台服务器对外提供服务。随着业务流量越来越大，单台服务器无论如何优化，无论采用多好的硬件，总会有性能天花板，当单服务器的性能无法满足业务需求时，就需要把多台服务器组成`集群系统`提高整体的处理性能。不过我们要使用统一的入口方式对外提供服务，所以需要一个`流量调度器`通过均衡的算法，将用户大量的请求均衡地分发到后端集群不同的服务器上。这就是**负载均衡**的由来。

## 负载均衡介绍

https://blog.csdn.net/liwei0526vip/article/details/103104393

## LVS 负载均衡

- Linux Virtual Server
- LVS（IPvs）已被集成到 Linux 内核中
- 负载均衡调度器

LVS 和 Nginx 的功能类似，都可以起到一个负载均衡调度器的作用

### 为什么使用 LVS + Nginx

- LVS 基于四层，工作效率更高
- 单个 Nginx 承受的压力有限，需要集群
- LVS 充当 Nginx 集群的调度者
- Nginx 接受请求并处理返回，LVS 可以只接受请求不处理响应

### LVS 基本原理：IPVS & LVS 模式

https://blog.csdn.net/liwei0526vip/article/details/103104483

### LVS 模式扩展：FullNAT 模式

https://blog.csdn.net/liwei0526vip/article/details/103104483

## LVS 操作实践

### LVS-DR 模式搭建

#### 配置 LVS 节点与 ipvsadm

##### 前期准备

1. 服务器与ip规划：

   - LVS - 1台
     - VIP（虚拟IP）：192.168.1.150
     - DIP（转发者IP/内网IP）：192.168.1.151
   - Nginx - 2台（RealServer）
     - RIP（真实IP/内网IP）：192.168.1.171
     - RIP（真实IP/内网IP）：192.168.1.172

2. 所有计算机节点关闭网络配置管理器，因为有可能会和网络接口冲突：

   ```shell
   systemctl stop NetworkManager 
   systemctl disable NetworkManager
   ```

##### 创建子接口

1. 进入到网卡配置目录，找到咱们的ens33：
   ![图片描述](https://climg.mukewang.com/5dc2875b08ce651416000419.jpg)
2. 拷贝并且创建子接口：

```
cp ifcfg-ens33 ifcfg-ens33:1
* 注：`数字1`为别名，可以任取其他数字都行
```

1. 修改子接口配置： `vim ifcfg-ens33:1`
2. 配置参考如下：
   ![图片描述](https://climg.mukewang.com/5dc287db0882451906340344.jpg)
   - 注：配置中的 192.168.1.150 就是咱们的vip，是提供给外网用户访问的ip地址，道理和nginx+keepalived那时讲的vip是一样的。
3. 重启网络服务，或者重启linux：
   ![图片描述](https://climg.mukewang.com/5dc287fc08c55cbc15100108.jpg)
4. 重启成功后，ip addr 查看一下，你会发现多了一个ip，也就是虚拟ip（vip）
   ![图片描述](https://climg.mukewang.com/5dc2887b086239f113560442.jpg)

##### 创建子接口 - 方式2（不推荐）

1. 创建网络接口并且绑定虚拟ip：

   ```
   ifconfig ens33:1 192.168.1.150/24
   ```

   - 配置规则如下：
     ![图片描述](https://climg.mukewang.com/5dc2888d08852c0412950520.jpg)
     配置成功后，查看ip会发现新增一个192.168.1.150：
     ![图片描述](https://climg.mukewang.com/5dc288a0088cf91d11070450.jpg)

> - 通过此方式创建的虚拟ip在重启后会自动消失

##### 安装 ipvsadm

现如今的centos都是集成了LVS，所以`ipvs`是自带的，相当于苹果手机自带ios，我们只需要安装`ipvsadm`即可（ipvsadm是管理集群的工具，通过ipvs可以管理集群，查看集群等操作），命令如下：

```bash
yum install ipvsadm
```

安装成功后，可以检测一下： ![图片描述](https://climg.mukewang.com/5dc288c1086c8e5015100198.jpg) 图中显示目前版本为1.2.1，此外是一个空列表，啥都没。

- 注：关于虚拟ip在云上的事儿

1. 阿里云不支持虚拟IP，需要购买他的负载均衡服务
2. 腾讯云支持虚拟IP，但是需要额外购买，一台节点最大支持10个虚拟ip

#### 两台 RS 配置虚拟 IP

配置虚拟网络子接口（回环接口）

1. 进入到网卡配置目录，找到lo（本地环回接口，用户构建虚拟网络子接口），拷贝一份新的随后进行修改：![图片描述](https://climg.mukewang.com/5dc28b540801d7ec16000905.jpg)
2. 修改内容如下：
   ![图片描述](https://climg.mukewang.com/5dc28b660814300912740488.jpg)
3. 重启后通过ip addr 查看如下，表示ok：
   ![图片描述](https://climg.mukewang.com/5dc28b8708f7cadb09200424.jpg)

#### 两台 RS 配置 ARP

##### ARP响应级别与通告行为 的概念

1. arp-ignore：ARP响应级别（处理请求）
   - 0：只要本机配置了ip，就能响应请求
   - 1：请求的目标地址到达对应的网络接口，才会响应请求
2. arp-announce：ARP通告行为（返回响应）
   - 0：本机上任何网络接口都向外通告，所有的网卡都能接受到通告
   - 1：尽可能避免本网卡与不匹配的目标进行通告
   - 2：只在本网卡通告

##### 配置ARP

1. 打开sysctl.conf:

   ```
   vim /etc/sysctl.conf
   ```

2. 配置`所有网卡`、`默认网卡`以及`虚拟网卡`的arp响应级别和通告行为，分别对应：`all`，`default`，`lo`：

   ```
   # configration for lvs
   net.ipv4.conf.all.arp_ignore = 1
   net.ipv4.conf.default.arp_ignore = 1
   net.ipv4.conf.lo.arp_ignore = 1
   
   net.ipv4.conf.all.arp_announce = 2
   net.ipv4.conf.default.arp_announce = 2
   net.ipv4.conf.lo.arp_announce = 2
   ```

3. 刷新配置文件：
   ![图片描述](https://climg.mukewang.com/5dc28bf408d55df509500336.jpg)

4. 增加一个网关，用于接收数据报文，当有请求到本机后，会交给lo去处理：![图片描述](https://climg.mukewang.com/5dc28c0a08af26b516000379.jpg)

5. 防止重启失效，做如下处理，用于开机自启动：

```
echo "route add -host 192.168.1.150 dev lo:1" >> /etc/rc.local
```

![图片描述](https://climg.mukewang.com/5dc28c2208e548eb13020736.jpg)

#### 使用 ipvsadm 配置集群规则

1. 创建 LVS 节点，用户访问的集群调度者

   ```bash
   ipvsadm -A -t 192.168.1.150:80 -s rr -p 5
   ```

   - -A：添加集群
   - -t：tcp协议
   - ip地址：设定集群的访问ip，也就是LVS的虚拟ip
   - -s：设置负载均衡的算法，rr表示轮询
   - -p：设置连接持久化的时间

2. 创建2台RS真实服务器

   ```bash
   ipvsadm -a -t 192.168.1.150:80 -r 192.168.1.171:80 -g
   ipvsadm -a -t 192.168.1.150:80 -r 192.168.1.172:80 -g
   ```

   - -a：添加真实服务器
   - -t：tcp协议
   - -r：真实服务器的ip地址
   - -g：设定 DR 模式

3. 保存到规则库，否则重启失效

   ```bash
   ipvsadm -S
   ```

4. 检查集群

   - 查看集群列表

     ```shell
     ipvsadm -Ln
     ```

   - 查看集群状态

     ```shell
     ipvsadm -Ln --stats
     ```

5. 其他命令：

   ```shell
   # 重启ipvsadm，重启后需要重新配置
   service ipvsadm restart
   # 查看持久化连接
   ipvsadm -Ln --persistent-conn
   # 查看连接请求过期时间以及请求源ip和目标ip
   ipvsadm -Lnc
   
   # 设置tcp tcpfin udp 的过期时间（一般保持默认）
   ipvsadm --set 1 1 1
   # 查看过期时间
   ipvsadm -Ln --timeout
   
   # 清除目前已配置的所有集群规则
   ipvsadm -C
   ```

6. 更详细的帮助文档：

   ```shell
   ipvsadm -h
   man ipvsadm
   ```

### 使用 Keepalived 配置 LVS

上述的所有操作都是通过 `ipvsadm` 命令来实现的，虽然也能完成相应的功能，但也存在如下的问题：

- 命令行操作和变更，很不方便
- 配置信息不方便持久化。
- 假设一个 RealServer 宕机或无响应，LVS 仍会将请求转发至后端，导致业务不可用。
- 存在单点问题，如果 LVS 服务器宕机，则全部服务不可用。

keepalived 本身就是为 LVS 而开发的，能够优雅的解决上述问题。

（一）**实验环境**

客户端：192.168.1.208
LVS-1：VIP=192.168.1.150 DIP=192.168.1.151
LVS-2：VIP=192.168.1.150 DIP=192.168.1.152
NGX-1：192.168.1.104
NGX-2：192.168.1.105

（二）**安装配置keepalvied**

Keepalived 安装详细参考：[Keepalived 原理与实战](https://www.cnblogs.com/christopherchan/p/12953230.html)

Keepalived MASTER 配置文件:

```nginx
global_defs {
		router_id LVS_151  
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 41
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.150
    }
}

# 配置集群地址访问的 IP + 端口，端口和 Nginx 保持一致，都是 80
virtual_server 192.168.1.150 80 {
  	# 健康检查的时间，单位：秒
    delay_loop 6
    # 配置负载均衡的算法，默认是轮询
    lb_algo wrr
    # 设置 LVS 的模式 NAT|TUN|DR
    lb_kind DR
    # 设置会话持久化的时间
    persistence_timeout 50
    # 协议 -t
    protocol TCP

    # 负载均衡的真实服务器 RS，也就是 Nginx 节点的具体的 IP 地址
    real_server 192.168.1.104 80 {
    		# 轮询的默认权重配比设置
        weight 1
        # 设置健康检查
        TCP_CHECK {
      			# 检查的超时时间
            connect_timeout 10
        		# 检查的端口
            connect_port 80
        		# 重试次数 5次
        		nb_get_retry 5
        		# 间隔时间 3s
        		delay_before_retry 3
        }
    }

    real_server 192.168.1.105 80 {
        weight 1
        TCP_CHECK {
            # 检查的超时时间
            connect_timeout 10
        		# 检查的端口
            connect_port 80
        		# 重试次数 5次
        		nb_get_retry 5
        		# 间隔时间 30s
        		delay_before_retry 30
        }
    }
}
```

Keepalived BACKUP 配置文件:

```nginx
global_defs {
		router_id LVS_152 
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 41
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.150
    }
}

# 配置集群地址访问的 IP + 端口，端口和 Nginx 保持一致，都是 80
virtual_server 192.168.1.150 80 {
  	# 健康检查的时间，单位：秒
    delay_loop 6
    # 配置负载均衡的算法，默认是轮询
    lb_algo wrr
    # 设置 LVS 的模式 NAT|TUN|DR
    lb_kind DR
    # 设置会话持久化的时间
    persistence_timeout 50
    # 协议 -t
    protocol TCP

    # 负载均衡的真实服务器 RS，也就是 Nginx 节点的具体的 IP 地址
    real_server 192.168.1.104 80 {
    		# 轮询的默认权重配比设置
        weight 1
        # 设置健康检查
        TCP_CHECK {
      			# 检查的超时时间
            connect_timeout 10
        		# 检查的端口
            connect_port 80
        		# 重试次数 5次
        		nb_get_retry 5
        		# 间隔时间 3s
        		delay_before_retry 3
        }
    }

    real_server 192.168.1.105 80 {
        weight 1
        TCP_CHECK {
            # 检查的超时时间
            connect_timeout 10
        		# 检查的端口
            connect_port 80
        		# 重试次数 5次
        		nb_get_retry 5
        		# 间隔时间 30s
        		delay_before_retry 30
        }
    }
}
```

上述 Keepalived + LVS 配置为双机主备配置，也可进一步优化为双机热配配置，详细请见[Keepalived 原理与实战](https://www.cnblogs.com/christopherchan/p/12953230.html)，与配置 Nginx 的双机主备类似。

> 更多 LVS 模式操作实践参考：https://blog.csdn.net/liwei0526vip/article/details/103104496

## LVS的负载均衡算法

### 静态算法

静态：根据LVS本身自由的固定的算法分发用户请求。

1. 轮询（Round Robin 简写`rr`）：轮询算法假设所有的服务器处理请求的能力都一样的，调度器会把所有的请求平均分配给每个真实服务器。（同Nginx的轮询）
2. 加权轮询（Weight Round Robin 简写`wrr`）：安装权重比例分配用户请求。权重越高，被分配到处理的请求越多。（同Nginx的权重）
3. 源地址散列（Source Hash 简写`sh`）：同一个用户ip的请求，会由同一个RS来处理。（同Nginx的ip_hash）
4. 目标地址散列（Destination Hash 简写`dh`）：根据url的不同，请求到不同的RS。（同Nginx的url_hash）

### 动态算法

动态：会根据流量的不同，或者服务器的压力不同来分配用户请求，这是动态计算的。

1. 最小连接数（Least Connections 简写`lc`）：把新的连接请求分配到当前连接数最小的服务器。
2. 加权最少连接数（Weight Least Connections 简写`wlc`）：服务器的处理性能用数值来代表，权重越大处理的请求越多。Real Server 有可能会存在性能上的差异，wlc动态获取不同服务器的负载状况，把请求分发到性能好并且比较空闲的服务器。
3. 最短期望延迟（Shortest Expected Delay 简写`sed`）：特殊的wlc算法。举例阐述，假设有ABC三台服务器，权重分别为1、2、3 。如果使用wlc算法的话，当一个新请求进来，它可能会分给ABC中的任意一个。使用sed算法后会进行如下运算：
   - A：（1+1）/1=2
   - B：（1+2）/2=3/2
   - C：（1+3）/3=4/3
     最终结果，会把这个请求交给得出运算结果最小的服务器。
4. 最少队列调度（Never Queue 简写`nq`）：永不使用队列。如果有Real Server的连接数等于0，则直接把这个请求分配过去，不需要在排队等待运算了（sed运算）。

### 总结：

LVS在实际使用过程中，负载均衡算法用的较多的分别为`wlc`或`wrr`，简单易用。

参考文献：[官方文档](http://www.linuxvirtualserver.org/zh/lvs4.html)

> 作为架构师，对lvs集群的负载算法有一定的了解即可，因为你要和运维人员进行有效沟通；但是如果作为运维的话那么是要深入钻研LVS了，一个企业如果发展并且使用到LVS了，那么业务量是十分巨大的，并且也会有专门的运维团队来负责网络架构的。

## 参考

- LVS 中文文档 http://www.linuxvirtualserver.org/zh/index.html
- https://blog.csdn.net/liwei0526vip/article/details/103104393
- https://blog.csdn.net/liwei0526vip/article/details/103104483
- https://blog.csdn.net/liwei0526vip/article/details/103104496