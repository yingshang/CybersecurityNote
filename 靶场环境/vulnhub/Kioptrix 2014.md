# Kioptrix 2014

下载地址

```
https://download.vulnhub.com/kioptrix/kiop2014.tar.bz2
```

## 实战操作

使用netdiscover命令查找靶机的IP。 靶机下载下来之后，直接运行是检测不到IP地址的，需要删除靶机原来的网卡，再重新添加网卡上去。

靶机IP地址：`192.168.0.106`。

扫描靶机端口

```
┌──(root💀kali)-[~]
└─# nmap -sV -p1-65535 192.168.0.106                                                                                                                    
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-19 02:02 EST
Nmap scan report for 192.168.0.12
Host is up (0.00032s latency).
Not shown: 65532 filtered ports
PORT     STATE  SERVICE VERSION
22/tcp   closed ssh
80/tcp   open   http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
8080/tcp open   http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
MAC Address: 00:0C:29:08:07:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 117.09 seconds
```

扫描80和8080端口，看看有啥东西。

```
┌──(root💀kali)-[~]
└─# nikto -h http://192.168.0.106/  
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.0.106
+ Target Hostname:    192.168.0.106
+ Target Port:        80
+ Start Time:         2021-12-19 02:05:06 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
+ Server may leak inodes via ETags, header found with file /, inode: 67014, size: 152, mtime: Sat Mar 29 13:22:52 2014
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ OpenSSL/0.9.8q appears to be outdated (current is at least 1.1.1). OpenSSL 1.0.0o and 0.9.8zc are also current.
+ Apache/2.2.21 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ mod_ssl/2.2.21 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ PHP/5.3.8 appears to be outdated (current is at least 7.2.12). PHP 5.6.33, 7.0.27, 7.1.13, 7.2.1 may also current release for each branch.
+ mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ 8724 requests: 0 error(s) and 11 item(s) reported on remote host
+ End Time:           2021-12-19 02:06:29 (GMT-5) (83 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

┌──(root💀kali)-[~]
└─# nikto -h http://192.168.0.106:8080/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.0.106
+ Target Hostname:    192.168.0.106
+ Target Port:        8080
+ Start Time:         2021-12-19 02:07:31 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ All CGI directories 'found', use '-C none' to test none
+ OpenSSL/0.9.8q appears to be outdated (current is at least 1.1.1). OpenSSL 1.0.0o and 0.9.8zc are also current.
+ mod_ssl/2.2.21 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ PHP/5.3.8 appears to be outdated (current is at least 7.2.12). PHP 5.6.33, 7.0.27, 7.1.13, 7.2.1 may also current release for each branch.
+ Apache/2.2.21 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ 26546 requests: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2021-12-19 02:11:41 (GMT-5) (250 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

查看80端口

![](../../.gitbook/assets/image.png)

查看页面源代码，找到一个路径

![](<../../.gitbook/assets/image (21) (1) (1) (1) (1) (1).png>)

访问这个目录

![](<../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1).png>)

google一下，这个版本的系统存在目录遍历漏洞，EXP

![](<../../.gitbook/assets/image (27) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

由于这个系统是freebsd，所以可以知道apache的位置

```
In FreeBSD, the main Apache HTTP Server configuration file is installed as /usr/local/etc/apache2 x /httpd.conf , where x represents the version number. This ASCII text file begins comment lines with a # . The most frequently modified directives are: ServerRoot "/usr/local"
```

看到配置文件，8080端口设置了一个环境变量，需要user-agent等于Mozilla/4.0，才可以访问。

![](<../../.gitbook/assets/image (26) (1) (1) (1) (1) (1).png>)

正常访问8080端口，403

![](<../../.gitbook/assets/image (16) (1) (1) (1) (1) (1).png>)





修改user-agent访问，找到了一个目录

![](<../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1) (1).png>)

访问phptax目录

```
curl -H "User-Agent:Mozilla/4.0" http://192.168.0.106:8080/phptax/ 
```

好像没什么东西，搜搜这个目录有没有payload，找到了

![](<../../.gitbook/assets/image (20) (1) (1) (1) (1) (1).png>)

没有python，设置不了交互式的终端

![](<../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png>)

freeesd9.0存在提权的exp。用echo命令直接复制粘贴过来

```
`echo' exp code ' > /tmp/exp.c`
```

找到flag

```
cd /root
ls
.cshrc
.history
.k5login
.login
.mysql_history
.profile
congrats.txt
folderMonitor.log
httpd-access.log
lazyClearLog.sh
monitor.py
ossec-alerts.log
cat congrats.txt
```











