# 部署PPTP VPN

### 1.1环境

本案例要求搭建一个`PPTP` `VPN`环境，并测试该`VPN`网络是否能够正常通讯，要求如下:

- 使用`PPTP`协议创建一个支持身份验证的隧道连接
- 使用`MPPE`对数据进行加密
- 为客户端分配192.168.3.0/24的地址池
- 客户端连接的用户名为`jacob`，密码为123456

### 1.2方案

准备实验所需的虚拟机环境，实验环境所需要的主机及对应的IP设置列表如表所示，正确配置IP地址、主机名称，并且为每台主机配置YUM源。

![](/run/media/root/96A1-15E3/计算机资料/table002.png)

实验拓扑图

![](/run/media/root/96A1-15E3/计算机资料/image002.png)

### 1.3步骤

实现此案例需要按照如下步骤进行。

**步骤一：部署`VPN`服务器**

1）安装软件包

```bash
[root@proxy ~]# yum install pptpd-1.4.0-2.el7.x86_64.rpm
[root@proxy ~]# rpm -qc pptpd
/etc/ppp/options.pptpd
/etc/pptpd.conf
/etc/sysconfig/pptpd
```

2)修改配置文件

```bash
[root@proxy ~]# vim /etc/pptpd.conf
.. ..
localip 201.1.2.5                                    //服务器本地IP
remoteip 192.168.3.1-50                            //分配给客户端的IP池
[root@proxy ~]# vim /etc/ppp/options.pptpd
require-mppe-128                                    //使用MPPE加密数据
ms-dns 8.8.8.8                                    //DNS服务器
[root@proxy ~]# vim /etc/ppp/chap-secrets            //修改账户配置文件
jacob           *               123456      *
//用户名     服务器名称    密码      客户端IP
```

3）启动服务

```bash
[root@proxy ~]# systemctl start pptpd
[root@proxy ~]# systemctl enable pptpd
```

~~4）翻墙设置（非必需操作）~~

```bash
[root@proxy ~]# echo "1" > /proc/sys/net/ipv4/ip_forward    //开启路由转发
[root@proxy ~]# iptables -t nat -A POSTROUTING -s 192.168.3.0/24 \
>  -j SNAT --to-source 201.1.2.5
```

**步骤二：客户端设置**

启动一台`Windows`虚拟机，将虚拟机网卡桥接到`public2`，配置IP地址为201.1.2.20。

新建网络连接（具体操作如图所示），输入`VPN`服务器账户与密码（具体操作如图所示），连接`VPN`并测试网络连通性（如图所示）。
![](/run/media/root/96A1-15E3/计算机资料/image003.png)

![](/run/media/root/96A1-15E3/计算机资料/image004.png)

```shell
C:\Users....\>ping  201.1.2.5
```

```bash
C:\Users....\>ping  192.168.88.5
```

