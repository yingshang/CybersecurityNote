# MORIA: 1.1

> https://download.vulnhub.com/moria/Moria1.1.rar

靶场IP：`192.168.32.227`

扫描对外端口服务

```
┌──(root💀kali)-[/tmp]
└─# nmap -p 1-65535 -sV  192.168.32.227                                                                                                                                                                                                                                                                                
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-11 08:06 EDT
Nmap scan report for 192.168.32.227
Host is up (0.00051s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
22/tcp open  ssh     OpenSSH 6.6.1 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
MAC Address: 00:0C:29:A0:85:AE (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.61 seconds

```

![image-20220911183041725](../../.gitbook/assets/image-20220911183041725.png)

访问路径：`http://192.168.32.227/w/h/i/s/p/e/r/the_abyss/`，每次刷新内容都会变化

![image-20220911202959786](../../.gitbook/assets/image-20220911202959786.png)
