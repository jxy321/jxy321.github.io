# web集群中，keepalived检测服务是否正常的脚本

```bash
check_nginx.sh
```

```shell
#!/bin/bash
#时间变量，用于记录日志
d=`date --date  today +%Y%m%d_%H:%M:%S`
#计算nginx进程数量
n=`ps -C nginx --no-heading|wc -l`
#如果进程为0，则启动Nginx，并且再次检测nginx进程数
#如果进程数还为0，则说明nginx无法启动，此时需要关闭Keepalived
if [ $n -eq "0" ]; then
   /usr/local/nginx/sbin/nginx
   n2=`ps -C nginx --no-heading|wc -l`
   if [ $n2 -eq "0" ]; then
      echo "$d nginx down,keepalived will stop" >> /var/log/check_nginx.log
      systemctl stop keepalived
   fi
fi

```

