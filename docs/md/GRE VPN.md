# GRE VPN

### 1.1环境

要求搭建一个`GRE VPN`环境，并测试该`VPN`网络是否能够正常通讯，要求如下：

- 启用内核模块`ip_gre`
- 创建一个虚拟VPN隧道`10.10.10.0/24`
- 实现两台主机点到点的隧道通讯

### 1.2方案

使用`lsmod`查看当前计算机已经加载的模块，使用`modeprobe`加载`linux`内核模块，使用`modinfo`可以查看内核模块的信息。

准备实验所需的虚拟机环境，实验环境所需要的主机以及对应的IP设置列表如图所示，正确配置IP地址、主机名称，配置`yum`源

![table001](/run/media/root/96A1-15E3/计算机资料/table001.png)

拓扑图如下：

![image001](/run/media/root/96A1-15E3/计算机资料/image001.png)

### 1.3步骤

实现此案例需要按照如下步骤进行。

**步骤一**：启用`GRE`模块（`client`和`proxy`都需要操作）

1）查看计算机当前加载的模块

```bash
[root@client ~]# lsmod                        #显示模块列表
[root@client ~]# lsmod  | grep ip_gre            #确定是否加载了gre模块
```

2)加载模块`ip_gre`

```bash
[root@client ~]# modprobe  ip_gre 
```

3）查看模块信息

```bash
[root@client ~]# modinfo ip_gre
filename:       /lib/modules/3.10.0-693.el7.x86_64/kernel/net/ipv4/ip_gre.ko.xz
… …    
```

**步骤二**：`Client`主机创建`VPN`隧道

1）创建隧道

```bash
[root@client ~]# ip tunnel add tun0  mode gre \ 
>  remote 201.1.2.5 local 201.1.2.10
//ip tunnel add创建隧道（隧道名称为tun0），ip tunnel help可以查看帮助
//mode设置隧道使用gre模式
//local后面跟本机的IP地址，remote后面是与其他主机建立隧道的对方IP地址
```

2）启用该隧道（类似与设置网卡`up`）

```bash
[root@client ~]# ip link show
[root@client ~]# ip link set tun0 up         //设置UP
[root@client ~]# ip link show
```

3）为`VPN`配置隧道`IP`地址

```bash
[root@client ~]# ip addr add 10.10.10.10/24 peer 10.10.10.5/24 \
>  dev tun0
//为隧道tun0设置本地IP地址（10.10.10.10.10/24）
//隧道对面的主机IP的隧道IP为10.10.10.5/24
[root@client ~]# ip a s                      //查看IP地址
```

**步骤三**：`Proxy`主机创建`VPN`隧道

1）查看计算机当前加载的模块

```bash
[root@client ~]# lsmod                        //显示模块列表
[root@client ~]# lsmod  | grep ip_gre            //确定是否加载了gre模块
```

2)加载模块`ip_gre`

```bash
[root@client ~]# modprobe  ip_gre
```

3）创建隧道

```bash
[root@proxy ~]# ~]# ip tunnel add tun0  mode gre \ 
>  remote 201.1.2.10 local 201.1.2.5
//ip tunnel add创建隧道（隧道名称为tun0），ip tunnel help可以查看帮助
//mode设置隧道使用gre模式
//local后面跟本机的IP地址，remote后面是与其他主机建立隧道的对方IP地址
```

4）启用该隧道（类似与设置网卡`up`）

```bash
[root@proxy ~]# ip a  s
[root@proxy ~]# ip link set tun0 up         //设置UP
[root@proxy ~]# ip a  s
```

5）为`VPN`配置隧道`IP`地址

```bash
[root@proxy ~]# ip addr add 10.10.10.5/24 peer 10.10.10.10/24 \
>  dev tun0
//为隧道tun0设置本地IP地址（10.10.10.10.5/24）
//隧道对面的主机IP的隧道IP为10.10.10.10/24
[root@proxy ~]# ip a s                      //查看IP地址
```

6)测试连通性

```bash
[root@client ~]#  ping 10.10.10.5
[root@proxy ~]#   ping 10.10.10.10
```

