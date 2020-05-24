# Keepalived 原理与实战

> 随着系统架构的逐渐演化，服务器的数量和结构会越来越复杂，例如 Web 服务器集群的搭建，提高了系统的性能，同时也提高了系统维护的复杂度，我们需要对集群中各台服务器进行监控，来保证为用户提供服务的是正常运行的服务器，整体系统的`可用性`就至关重要。

## Keepalived 简介

### 什么是Keepalived ？

Keepalived一个基于[VRRP](https://link.zhihu.com/?target=http%3A//en.wikipedia.org/wiki/VRRP) 协议来实现的 LVS 服务高可用方案，可以利用其来解决单点故障。一个LVS服务会有2台服务器运行Keepalived，一台为主服务器（MASTER），一台为备份服务器（BACKUP），但是对外表现为一个`虚拟IP`，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候， 备份服务器就会接管虚拟IP，继续提供服务，从而保证了高可用性。

### Keepalived 的作用

如上述所说，**Keepalived** 提供了很好的`高可用性保障服务`，它可以检查服务器的状态，如果有服务器出现问题，Keepalived 会将其从系统中移除，并且同时使用备份服务器代替该服务器的工作，当这台服务器可以正常工作后，Keepalived 再将其放入服务器群中，这个过程是 Keepalived 自动完成的，不需要人工干涉，我们只需要修复出现问题的服务器即可。

## Keepalived 原理



### 基于VRRP协议的理解

Keepalived 是以 `VRRP` 协议为实现基础的，VRRP全称`Virtual Router Redundancy Protocol`，即`虚拟路由冗余协议`。

虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master 和多个 backup，master 上面有一个对外提供服务的 `VIP(Virtual IP Address)`（该路由器所在局域网内其他机器的默认路由为该 vip），master 会发组播，当 backup 收不到 vrrp 包时就认为 master 宕掉了，这时就需要根据 VRRP 的优先级来`选举`一个 backup 当 master。这样的话就可以保证路由器的高可用了。

keepalived 主要有三个模块，分别是core、check 和 vrrp。core 模块为keepalived的**核心**，负责主进程的启动、维护以及全局配置文件的加载和解析。check 负责健康检查，包括常见的各种检查方式。vrrp 模块是来实现 VRRP 协议的。

### 基于TCP/IP协议的理解

以检测 web 服务器为例，Keepalived 从3个层次来检测服务器的状态

Layer3 、Layer4 以及 Layer7 工作在IP/TCP协议栈的IP层，TCP层，及应用层,原理分别如下：

**Layer3：**

Keepalived使用Layer3的方式工作时，Keepalived会定期向服务器群中的服务器发送一个ICMP的数据包（既我们平时用的Ping程序）,如果发现某台服务的IP地址没有激活，Keepalived 便报告这台服务器失效，并将它从服务器群中剔除，这种情况的典型例子是某台服务器被非法关机。Layer3 的方式是以服务器的`IP地址`是否有效作为服务器工作正常与否的标准。

**Layer4:**

如果您理解了Layer3的方式，Layer4就容易了。Layer4主要以`TCP 端口的状态`来决定服务器工作正常与否。如 web server 的服务端口一般是80，如果 Keepalived 检测到80端口没有启动，则 Keepalived 将把这台服务器从服务器群中剔除。

**Layer7：**

Layer7 就是工作在具体的应用层了，比Layer3,Layer4要复杂一点，在网络上占用的带宽也要大一些。Keepalived 将根据用户的设定检查服务器程序的运行是否正常，如果与用户的设定不相符，则 Keepalived 将把服务器从服务器群中剔除。

![Keepalived 双机主备原理](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-20-151648.png)

## Keepalived 选举策略

### 选举策略

首先，每个节点有一个初始优先级，由配置文件中的`priority`配置项指定，MASTER 节点的 priority 应比 BAKCUP 高。运行过程中 keepalived 根据 vrrp_script 的 `weight` 设定，增加或减小节点优先级。规则如下：

1. `weight`值为正时,脚本检测成功时”weight”值会加到”priority”上,检测失败时不加
   - 主失败: 主priority < 备priority+weight之和时会切换
   - 主成功: 主priority+weight之和 > 备priority+weight之和时,主依然为主,即不发生切换
2. `weight`为负数时,脚本检测成功时”weight”不影响”priority”,检测失败时,Master节点的权值将是“priority“值与“weight”值之差
   - 主失败: 主priotity-abs(weight) < 备priority时会发生切换
   - 主成功: 主priority > 备priority 不切换
3. 当两个节点的优先级相同时，以节点发送`VRRP通告`的 IP 作为比较对象，IP较大者为MASTER。

### priority 和 weight 的设定

1. 主从的优先级初始值priority和变化量weight设置非常关键，配错的话会导致无法进行主从切换。比如，当MASTER初始值定得太高，即使script脚本执行失败，也比BACKUP的priority + weight大，就没法进行VIP漂移了。
2. 所以priority和weight值的设定应遵循: abs(MASTER priority - BAKCUP priority) < abs(weight)。一般情况下，初始值MASTER的priority值应该比较BACKUP大，但不能超过weight的绝对值。 另外，当网络中不支持多播(例如某些云环境)，或者出现网络分区的情况，keepalived BACKUP节点收不到MASTER的VRRP通告，就会出现脑裂(split brain)现象，此时集群中会存在多个MASTER节点。

## Keepalived 实战进阶

### Keepalived 安装部署

1. 下载安装包

Keepalived 官网下载地址：https://www.keepalived.org/download.html

![image-20200521210358933](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-21-130359.png)

2. 解压

   ```bash
   tar -zxvf keepalived-2.0.18.tar.gz
   ```

3. 解压后进入到解压出来的目录，看到会有`configure`，那么就可以做配置了（配置安装和nginx一模一样）

   ![image-20200521210723704](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-21-130724.png)

4. 使用`configure`命令配置安装目录与核心配置文件所在位置

   ```bash
   ./configure --prefix=/usr/local/keepalived --sysconf=/etc
   ```

   - prefix：keepalived安装的位置
   - sysconf：keepalived核心配置文件所在位置，固定位置，改成其他位置则keepalived启动不了，/var/log/messages中会报错

   配置过程中可能会出现警告信息，如下所示：

   ![image-20200521211303189](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-21-131303.png)

   解决方法：安装 libnl/libnl-3 依赖 `yum -y install libnl libnl-devel`，重新`configure`一下就好了。

5. 安装keepalived

   ```bash
   make && make install
   ```

6. 进入到`/etc/keepalived`，该目录下为keepalived核心配置文件

### Keepalived 配置

#### 把 Keepalived 注册为系统服务

进入解压缩安装包的 `etc` 文件夹

![image-20200521222326891](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-21-142327.png)

将系统配置文件拷贝至系统 `etc`文件

```bash
cp init.d/keepalived /etc/init.d/
cp sysconfig/keepalived /etc/sysconfig/
```

刷新系统服务，加载新添加的 Keepalived 服务

```bash
systemctl daemon-reload
```

#### 主服务器（Master）配置

1. 通过命令 `vim keepalived.conf` 打开配置文件，详细配置如下

```bash
global_defs {
   # 路由id：当前安装keepalived的节点主机标识符，保证全局唯一
   router_id keep_104
}

vrrp_instance VI_1 {
    # 表示状态是MASTER主机还是备用机BACKUP
    state MASTER
    # 该实例绑定的网卡名称
    interface ens33
    # 保证主备节点一致即可
    virtual_router_id 51
    # 权重，master权重一般高于backup，如果有多个，那就是选举，谁的权重高，谁就当选
    priority 100
    # 主备之间同步检查时间间隔，单位秒
    advert_int 2
    # 认证权限密码，防止非法节点进入
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟出来的ip，可以有多个（vip）
    virtual_ipaddress {
        192.168.1.108
    }
}
```

**附：查看网卡名称**

![image-20200521214418246](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-21-134419.png)

2. 启动 Keepalived

   在sbin目录中进行启动（同nginx），如下图：

   ![image-20200521214728691](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-21-134728.png)

3. 查看进程

   ```bash
   ps -ef|grep keepalived
   ```

   ![image-20200521214838919](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-05-21-134839.png)

4. 查看虚拟 IP（VIP）

   在网卡 ens33 下，多了一个 `192.168.1.108` ，这个就是虚拟ip

   ![image-20200521220643981](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-21-140644.png)

#### Keepalived 实现双机主备高可用

在配置完 Keepalived 主服务器节点后，接下来就可以配置备用服务器节点了，备用服务器配置如下：

```bash
global_defs {
   router_id keep_105
}

vrrp_instance VI_1 {
    # 备用机设置为BACKUP
    state BACKUP
    interface ens33
    virtual_router_id 51
    # 权重低于MASTER
    priority 80
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        # 注意：主备两台的vip都是一样的，绑定到同一个vip
        192.168.1.108
    }
}
```

启动 Keepalived

```bash
# 启动keepalived
systemctl start keepalived
# 停止keepalived
systemctl stop keepalived
# 重启keepalived
systemctl restart keepalived
```

查看备用服务器 Keepalived 进程

![image-20200522235349567](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-22-155350.png)

现在主服务器节点 `192.168.1.104`以及备用服务器节点 `192.168.1.105`以及 Keepalived 虚拟 IP `192.168.1.105`已配置完毕

![image-20200522235637330](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-22-155638.png)

![image-20200522235724708](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-22-155725.png)

这里我在本地将 Keepalived 虚拟IP `192.168.1.108`已映射至 `www.keep.com`

![image-20200522235859619](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-22-155859.png)

所以直接访问 `www.keep.com` 即可访问主服务器节点

![image-20200523000014318](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-22-160014.png)

当主服务器节点的 Keepalived 服务不可用时（这里我直接将主服务器的 Keepalived 服务直接停止`systemctl stop keepalived.service`，便于测试），虚拟IP 自动绑定至备用服务器节点地址

![image-20200523000333384](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-22-160333.png)

#### Keepalived 配置 Nginx 自动重启

1. 增加Nginx重启检测脚本

   ```bash
   vim /etc/keepalived/check_nginx_alive_or_not.sh
   ```

   ```shell
   #!/bin/bash
   
   A=`ps -C nginx --no-header |wc -l`
   # 判断nginx是否宕机，如果宕机了，尝试重启
   if [ $A -eq 0 ];then
       /usr/local/nginx/sbin/nginx
       # 等待一小会再次检查nginx，如果没有启动成功，则停止keepalived，使其启动备用机
       sleep 3
       if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
           killall keepalived
       fi
   fi
   ```

   增加运行权限

   `chmod +x /etc/keepalived/check_nginx_alive_or_not.sh`

2. 在 keepalived.conf 配置定时监听 nginx 状态脚本

   ```bash
   vrrp_script check_nginx_alive {
       script "/etc/keepalived/check_nginx_alive_or_not.sh"
       interval 2 # 每隔两秒运行上一行脚本
       weight 10 # 如果脚本运行成功，则升级权重+10
       # weight -10 # 如果脚本运行失败，则升级权重-10
   }
   ```

3. 在`vrrp_instance`中新增监控的脚本

   ```bash
   track_script {
       check_nginx_alive   # 追踪 nginx 脚本
   }
   ```

4. 重启Keepalived使得配置文件生效

   ```bash
   systemctl restart keepalived
   ```

#### Keepalived 实现双主热备高可用

![image-20200523004644961](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-22-164645.png)

首先需要配置云服务的 DNS 解析配置和负载均衡，详细配置参考：

- 阿里云：https://help.aliyun.com/document_detail/52528.html
- 腾讯云：https://cloud.tencent.com/document/product/302/11358

Keepalived 双主热备详细配置：

规则：以一个虚拟ip分组归为同一个路由

**主节点配置**

```nginx
global_defs {
   router_id keep_104
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.108
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface ens33
    virtual_router_id 52
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.138
    }
}
```

**备用节点配置**

```nginx
global_defs {
   router_id keep_105
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.108
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface ens33
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.138
    }
}
```

## 参考文档

- https://www.huweihuang.com/linux-notes/keepalived/keepalived-introduction.html