# 使用docker发布服务

《docker-从入门到实践》https://yeasy.gitbook.io/docker_practice/

## 发布httpd服务

### 1、下载镜像服务并导入到宿主机上

```bash
#下载获取镜像tar包：myos.tar.xz
#清空已有镜像
docker rmi  镜像
docker load -i myos.tar.xz
docker images
```

### 2、在宿主机上发布apache

```bash
docker run -itd --name apache myos:httpd
docker ps
#查询开启apache服务的ip
docker inspect apache
}
...
"IPAddress": "172.17.0.2"
...
}
```

### 3、在宿主机上访问apache服务

```bash
curl http://172.17.0.2
curl http://172.17.0.2/info.php
#都可以访问
```

### 4、在客户端访问apache服务

```bash
curl http://172.17.0.2
curl http://172.17.0.2/info.php
#在客户端直接访问都失败
```

**为了能在客户端访问我们在宿主机上创建的apache服务，我们有以下解决方案：**

**方案一：我们在客户端上设置路由转发：**

```bash
ip route add 172.17.0.0/16 via 宿主机ip   #凡是访问172.17.0.0的网络都转发到 宿主机

#这时访问就可以成功
```

但是这种方法我们只能路由到一台设备，如果有两台以上的设备我们就无法访问，而且我们需要知道宿主机的ip才能设置，但往往我们不一定知道宿主机的ip，那么我们并不能使用这种方法去访问我们发布的服务。

**方案二：端口映射：**

```bash
#先删除上面已经发布的apache服务
docker rm -f apache
```

```bash
#容器服务端口与宿主机端口映射
docker  run -itd -p 80:80 --name apache myos:httpd

#通过访问宿主机的ip即可访问服务
```

## 发布nginx、PHP服务

```bash
#创建文件存放目录
mkdir /var/webconf
#生成容器
docker run -itd --name nginx myos:nginx
#得到nginx配置文件
docker cp nginx:/usr/local/nginx/conf/nginx.conf  /var/webconf
docker  rm -f nginx
#修改nginx.conf,使其支持php
vim /var/webconf/nginx.conf
...
...

```

