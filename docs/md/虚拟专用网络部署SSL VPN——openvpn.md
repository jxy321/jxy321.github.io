# 虚拟专用网络部署SSL VPN——openvpn

**openvpn 工作原理--部署过程 每一步在做什么**

- 需要先关注保证数据安全传输的三要素：数据机密性 数据完整性 身份认证
- 需要掌握秘钥加密技术应用实现；
- 需要掌握证书概念的企业应用；

------

***虚拟专用网络部署***

**虚拟专用网络部署架构**

在部署构建openvpn虚拟专用网络前，需要准备好基本的架构环境：

![img](https://pic4.zhimg.com/80/v2-8b19bbcb49550de8d475bf3fcc66f177_720w.webp)

**操作系统基础优化配置**

***01 系统默认selinux安全策略优化说明\***

```python
# 临时关闭selinux策略
[root@oldboy ~]# setenforce 0
[root@oldboy ~]# getenforce 
Permissive

# 永久关闭selinux策略
[root@oldboy ~]# cat /etc/selinux/config 
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
       -- 表示selinux安全策略功能是启用状态
#     permissive - SELinux prints warnings instead of enforcing.
       -- 表示selinux安全策略只是显示警告信息，不会进行安全处理
#     disabled - No SELinux policy is loaded.
       -- 表示selinux安全策略功能彻底禁用
SELINUX=enforcing
[root@oldboy ~]# sed -i '7s#enforcing#disabled#g' /etc/selinux/config
[root@oldboy ~]# reboot
```


***02 系统默认防火墙服务优化说明\***

```python
# 临时关闭防火墙
[root@oldboy ~]# systemctl stop firewalld.service

# 永久关闭防火墙
[root@oldboy ~]# systemctl disable firewalld.service
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

# 操作配置查看确认
[root@oldboy ~]# systemctl status firewalld
firewalld.service - firewalld - dynamic firewall daemon
Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
Active: inactive (dead)
[root@oldboy ~]# systemctl is-active firewalld.service 
unknown
[root@oldboy ~]# systemctl is-enabled firewalld.service 
disabled
```

***03 系统软件程序下载优化方法\***

```python
# 配置官方源更新地址：
[root@oldboy ~]# curl -s -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# 配置第三方epel源更新地址：
[root@oldboy ~]# curl -s -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

***04 系统基础软件程序下载安装\***

```python
# 企业应用基础工具程序：
[root@oldboy ~]# yum install -y  tree nmap lrzsz dos2unix nc lsof wget -y

# 企业应用扩展工具程序：
[root@oldboy ~]# yum install -y  psmisc net-tools bash-completion vim-enhanced git -y
```

**虚拟专用网络证书制作**

根据之前的openvpn软件工作原理说明，在部署虚拟专用网络服务之前，需要进行相关证书文件制作；

证书文件制作过程，需要使用到easy-rsa-old.zip工具进行制作证书。

制作证书工具下载：

```text
https://github.com/OpenVPN/easy-rsa-old
```

将证书制作工具上传到主机中：

```python
# 上传证书制作工具
[root@xiaoQ ~]# rz -y
[root@xiaoQ ~]# ll easy-rsa-old-master.zip 
-rw-r--r--. 1 root root 59661 6月  21 03:56 easy-rsa-old-master.zip

# 解压证书制作工具压缩包
[root@xiaoQ ~]# unzip easy-rsa-old-master.zip

# 查看证书制作工具命令信息
[root@xiaoQ ~]# cd easy-rsa-old-master/easy-rsa/2.0/
[root@xiaoQ 2.0]# ls
build-ca      build-key               build-key-server   clean-all         openssl-0.9.6.cnf  pkitool         vars
build-dh      build-key-pass      build-req               inherit-inter  openssl-0.9.8.cnf   revoke-full  whichopensslcnf
build-inter  build-key-pkcs12  build-req-pass      list-crl            openssl-1.0.0.cnf   sign-req
```

编写vars文件，修改证书创建的配置文件参数信息：

```python
# 编辑证书创建配置参数文件
[root@xiaoQ 2.0]# vim vars
 67 export KEY_COUNTRY="cn"
 68 export KEY_PROVINCE="beijing"
 69 export KEY_CITY="beijing"
 70 export KEY_ORG="oldboy"
 71 export KEY_EMAIL="test@qq.com"
 72 export KEY_CN=xiaoq
 73 export KEY_NAME=xiaoq
 74 export KEY_OU=xiaoq
 75 export PKCS11_MODULE_PATH=changeme
 76 export PKCS11_PIN=1234
-- 以上信息表示证书请求文件的参数信息

# 加载配置文件修改参数信息
[root@xiaoQ 2.0]# source vars
NOTE: If you run ./clean-all, I will be doing a rm -rf on /root/easy-rsa-old-master/easy-rsa/2.0/keys
-- 执行./clean-all会在目录中创建出keys目录，专门用于存放证书文件信息
[root@xiaoQ 2.0]# ./clean-all 
[root@xiaoQ 2.0]# ls
.. 省略部署信息...
keys
-- 此目录中稍后会生成出所创建的证书和私钥文件信息
```

生成根证书文件和私钥文件信息：

```python
[root@xiaoQ 2.0]# ./build-ca 
Generating a 4096 bit RSA private key
...++
.................................++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [cn]:
State or Province Name (full name) [beijing]:
Locality Name (eg, city) [beijing]:
Organization Name (eg, company) [oldboy]:
Organizational Unit Name (eg, section) [xiaoq]:
Common Name (eg, your name or your server's hostname) [xiaoq]:
Name [xiaoq]:
Email Address [test@qq.com]:
-- 一路回车操作，用于产生根证书和私钥文件信息

[root@xiaoQ 2.0]# ll keys/
总用量 12
-rw-r--r--. 1 root root 2346 6月  21 04:14 ca.crt   -- 根证书
-rw-------. 1 root root 3272 6月  21 04:14 ca.key  -- 私钥
```

生成服务端证书和秘钥文件信息：

```python
[root@xiaoQ 2.0]# ./build-key-server server
Generating a 4096 bit RSA private key
........................................................................................................................++
...........................................................................++
writing new private key to 'server.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [cn]:
State or Province Name (full name) [beijing]:
Locality Name (eg, city) [beijing]:
Organization Name (eg, company) [oldboy]:
Organizational Unit Name (eg, section) [xiaoq]:
Common Name (eg, your name or your server's hostname) [server]:
Name [xiaoq]:
Email Address [test@qq.com]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /root/easy-rsa-old-master/easy-rsa/2.0/openssl-1.0.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'cn'
stateOrProvinceName   :PRINTABLE:'beijing'
localityName          :PRINTABLE:'beijing'
organizationName      :PRINTABLE:'oldboy'
organizationalUnitName:PRINTABLE:'xiaoq'
commonName            :PRINTABLE:'server'
name                  :PRINTABLE:'xiaoq'
emailAddress          :IA5STRING:'test@qq.com'
Certificate is to be certified until Jun 17 20:19:13 2032 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
-- 一路回车操作，最后输入两个y，用于产生服务端证书和私钥文件信息

[root@xiaoQ 2.0]# ll keys/
-rw-r--r--. 1 root root 8090 6月  21 04:19 server.crt   -- 服务端证书
-rw-r--r--. 1 root root 1752 6月  21 04:19 server.csr   -- 服务端请求证书文件 
-rw-------. 1 root root 3272 6月  21 04:19 server.key  --  服务端私钥文件[root@xiaoQ 2.0]# ./build-key-server server
Generating a 4096 bit RSA private key
........................................................................................................................++
...........................................................................++
writing new private key to 'server.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [cn]:
State or Province Name (full name) [beijing]:
Locality Name (eg, city) [beijing]:
Organization Name (eg, company) [oldboy]:
Organizational Unit Name (eg, section) [xiaoq]:
Common Name (eg, your name or your server's hostname) [server]:
Name [xiaoq]:
Email Address [test@qq.com]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /root/easy-rsa-old-master/easy-rsa/2.0/openssl-1.0.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'cn'
stateOrProvinceName   :PRINTABLE:'beijing'
localityName          :PRINTABLE:'beijing'
organizationName      :PRINTABLE:'oldboy'
organizationalUnitName:PRINTABLE:'xiaoq'
commonName            :PRINTABLE:'server'
name                  :PRINTABLE:'xiaoq'
emailAddress          :IA5STRING:'test@qq.com'
Certificate is to be certified until Jun 17 20:19:13 2032 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
-- 一路回车操作，最后输入两个y，用于产生服务端证书和私钥文件信息

[root@xiaoQ 2.0]# ll keys/
-rw-r--r--. 1 root root 8090 6月  21 04:19 server.crt   -- 服务端证书
-rw-r--r--. 1 root root 1752 6月  21 04:19 server.csr   -- 服务端请求证书文件 
-rw-------. 1 root root 3272 6月  21 04:19 server.key  --  服务端私钥文件[root@xiaoQ 2.0]# ./build-ca 
Generating a 4096 bit RSA private key
...++
.................................++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [cn]:
State or Province Name (full name) [beijing]:
Locality Name (eg, city) [beijing]:
Organization Name (eg, company) [oldboy]:
Organizational Unit Name (eg, section) [xiaoq]:
Common Name (eg, your name or your server's hostname) [xiaoq]:
Name [xiaoq]:
Email Address [test@qq.com]:
-- 一路回车操作，用于产生根证书和私钥文件信息

[root@xiaoQ 2.0]# ll keys/
总用量 12
-rw-r--r--. 1 root root 2346 6月  21 04:14 ca.crt   -- 根证书
-rw-------. 1 root root 3272 6月  21 04:14 ca.key  -- 私钥
```

生成客户端证书和秘钥文件信息：

```python
[root@xiaoQ 2.0]# ./build-key client
Generating a 4096 bit RSA private key
..........................................................++
....................................................................................................................................................................................................................................++
writing new private key to 'client.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [cn]:
State or Province Name (full name) [beijing]:
Locality Name (eg, city) [beijing]:
Organization Name (eg, company) [oldboy]:
Organizational Unit Name (eg, section) [xiaoq]:
Common Name (eg, your name or your server's hostname) [client]:
Name [xiaoq]:
Email Address [test@qq.com]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /root/easy-rsa-old-master/easy-rsa/2.0/openssl-1.0.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'cn'
stateOrProvinceName   :PRINTABLE:'beijing'
localityName          :PRINTABLE:'beijing'
organizationName      :PRINTABLE:'oldboy'
organizationalUnitName:PRINTABLE:'xiaoq'
commonName            :PRINTABLE:'client'
name                  :PRINTABLE:'xiaoq'
emailAddress          :IA5STRING:'test@qq.com'
Certificate is to be certified until Jun 17 20:23:35 2032 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
-- 一路回车操作，最后输入两个y，用于产生客户端证书和私钥文件信息

[root@xiaoQ 2.0]# ll keys/
-rw-r--r--. 1 root root 7972 6月  21 04:23 client.crt  -- 客户端证书
-rw-r--r--. 1 root root 1752 6月  21 04:23 client.csr  -- 客户端请求证书文件
-rw-------. 1 root root 3272 6月  21 04:23 client.key -- 客户端私钥文件
```

生成秘钥交换文件信息：

```python
[root@xiaoQ 2.0]# ./build-dh
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
.........................................
-- 用于产生秘钥交换文件信息

[root@xiaoQ 2.0]# ll keys/
-rw-r--r--. 1 root root  424 6月  21 04:27 dh2048.pem
```

> 利用证书制作工具easy-ras.zip，最终会生成重要的7个文件，ca两个，server两个，client两个，秘钥交换一个。

**虚拟专用网络服务配置**

在安装部署虚拟专用网络服务时，需要对openvpn服务配置文件进行修改调整；

下载安装openvpn服务程序包：

```python
[root@xiaoQ ~]# yum install -y openvpn
```

编写修改openvpn服务配置文件：

```python
# 建立存放openvpn服务加载证书文件目录
[root@xiaoQ ~]# cd /etc/openvpn/
[root@xiaoQ openvpn]# mkdir keys

# 将之前生成的证书文件信息进行拷贝迁移
[root@xiaoQ openvpn]# cp /root/easy-rsa-old-master/easy-rsa/2.0/keys/server.crt ./keys/
[root@xiaoQ openvpn]# cp /root/easy-rsa-old-master/easy-rsa/2.0/keys/server.key ./keys/
[root@xiaoQ openvpn]# cp /root/easy-rsa-old-master/easy-rsa/2.0/keys/ca.crt ./keys/
[root@xiaoQ openvpn]# cp /root/easy-rsa-old-master/easy-rsa/2.0/keys/dh2048.pem ./keys/

# 检查确认是否拷贝迁移成功
[root@xiaoQ openvpn]# ll ./keys/
总用量 20
-rw-r--r--. 1 root root 2346 6月  21 04:38 ca.crt
-rw-r--r--. 1 root root  424 6月  21 04:38 dh2048.pem
-rw-r--r--. 1 root root 8090 6月  21 04:38 server.crt
-rw-------. 1 root root 3272 6月  21 04:38 server.key

# 拷贝openvpn服务模板配置文件
[root@xiaoQ openvpn]# cp /usr/share/doc/openvpn-2.4.12/sample/sample-config-files/server.conf ./
[root@xiaoQ openvpn]# ll server.conf 
-rw-r--r--. 1 root root 10784 6月  21 04:41 server.conf

# 编辑openvpn服务模板配置文件
[root@xiaoQ openvpn]# vim server.conf 
78 ca keys/ca.crt
79 cert keys/server.crt
80 key keys/server.key  # This file should be kept secret
85 dh keys/dh2048.pem
-- 更改配置加载的证书文件路径信息
101 server 10.0.1.0 255.255.255.0
-- 当vpn拨号建立连接成功后，会生成的隧道连接网段信息
143 push "route 10.0.1.0 255.255.255.0"
144 push "route 172.16.30.0 255.255.255.0"
-- 表示路由信息推送，可以让拨号的客户端主机路由表中，添加以上两个路由条目信息；
-- 最终实现和企业私网以及vpn隧道私网建立通信
246 tls-auth keys/ta.key 0
-- 表示拒绝服务攻击的证书
254 cipher AES-256-GCM
-- 表示设置数据传输的加密模式，从2.4版本之后不能使用CBC模式了，需要改为GCM
```

设置开启openvpn服务路由转发：

```python
[root@xiaoQ openvpn]# echo "net.ipv4.ip_forward = 1" >>/etc/sysctl.conf 
[root@xiaoQ openvpn]# sysctl -p
net.ipv4.ip_forward = 1
-- 实现让vpn服务器具有路由器的功能，进行数据包的路由转发。
```

在openvpn服务端建立ta.key文件，主要用于拒绝服务攻击证书文件：

```python
[root@xiaoQ openvpn]# cd keys/
[root@xiaoQ keys]# pwd
/etc/openvpn/keys
[root@xiaoQ keys]# openvpn --genkey --secret ta.key
[root@xiaoQ keys]# ls
ta.key
-- 生成此文件主要作用就是加固openvpn服务的安全性
```

启动运行openvpn服务程序：

```python
[root@xiaoQ openvpn]# pwd
/etc/openvpn
[root@xiaoQ openvpn]# openvpn --daemon --config server.conf 
[root@xiaoQ openvpn]# netstat -lntup | grep 1194
udp        0      0 0.0.0.0:1194            0.0.0.0:*                           2238/openvpn 
```

**虚拟专用网络客户配置**

缩写配置openvpn客户端程序配置文件：

```python
# 创建保存客户端文件信息的目录，并将客户端模板文件进行拷贝迁移
[root@xiaoQ ~]# mkdir client   
[root@xiaoQ ~]# cp /usr/share/doc/openvpn-2.4.12/sample/sample-config-files/client.conf /root/client/

# 编写客户端配置文件信息：
[root@xiaoQ ~]# vim client/client.conf
 44 remote 192.168.30.101 1194
-- 表示设置客户端要和哪个vpn服务器建立连接，设置为vpn服务器外网接口公网地址和服务端口1194信息
116 cipher AES-256-GCM
-- 表示设置数据传输的加密模式，从2.4版本之后不能使用CBC模式了，需要改为GCM
```

导出保存openvpn客户端证书相关文件：

```python
# 汇总拷贝整理客户端相关证书文件
[root@xiaoQ ~]# cp easy-rsa-old-master/easy-rsa/2.0/keys/client.key /root/client/
[root@xiaoQ ~]# cp easy-rsa-old-master/easy-rsa/2.0/keys/client.crt /root/client/
[root@xiaoQ ~]# cp easy-rsa-old-master/easy-rsa/2.0/keys/ca.crt /root/client/
[root@xiaoQ ~]# cp /etc/openvpn/keys/ta.key /root/client/

# 检查确认客户端数据信息情况
[root@xiaoQ ~]# ll /root/client/
总用量 24
-rw-r--r--. 1 root root 2346 6月  21 05:17 ca.crt
-rw-r--r--. 1 root root 3613 6月  21 05:15 client.conf
-rw-r--r--. 1 root root 7972 6月  21 05:17 client.crt
-rw-------. 1 root root 3272 6月  21 05:17 client.key
-rw-------. 1 root root  636 6月  21 05:17 ta.key

# 修改客户端文件后缀名称信息
[root@xiaoQ ~]# cd /root/client/
[root@xiaoQ client]# mv client.conf client.ovpn
[root@xiaoQ client]# ll client.ovpn 
-rw-r--r--. 1 root root 3613 6月  21 05:15 client.ovpn
```

将所有openvpn客户端所需的文件数据打包并下载保存

```python
[root@xiaoQ ~]# pwd
/root
[root@xiaoQ ~]# zip client.zip client/*
  adding: client/ (stored 0%)
[root@xiaoQ ~]# ll client.zip 
-rw-r--r--. 1 root root 164 6月  21 05:21 client.zip
[root@xiaoQ ~]# sz -y client.zip 
```