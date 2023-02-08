# vulnhub

## 类OSCP靶机

|                           靶场名字                           |                         学习到的内容                         | 是否完成 | 靶场时间    |
| :----------------------------------------------------------: | :----------------------------------------------------------: | -------- | ----------- |
|         [Kioptrix: Level 1](<Kioptrix  Level 1.md>)          |           searchsploit、mod\_ssl RCE、SMB RCE应用            | ⚪        | 17 Feb 2010 |
|       [Kioptrix: Level 1.1](<Kioptrix  Level 1.1.md>)        |             万能密码、命令注入，nc反弹，本地提权             | ⚪        | 11 Feb 2011 |
|        [Kioptrix: Level 1.2](<Kioptrix Level 1.2.md>)        |                SQL手工联合注入、ht编辑器提权                 | ⚪        | 18 Apr 2011 |
|        [Kioptrix: Level 1.3](<Kioptrix Level 1.3.md>)        | Samba枚举、万能密码、终端模式、MySQL提权、文件包含、字符截断、/proc/self/fd写入内容。 | ⚪        | 8 Feb 2012  |
|             [Kioptrix 2014](<Kioptrix 2014.md>)              |                     CMS EXP、EXP本地提权                     | ⚪        | 6 Apr 2014  |
|           [FristiLeaks：1.3](FristiLeaks：1.3.md)🔺           |               base64加解密、上传绕过、sudo提权               | ⚪        | 14 Dec 2015 |
|                [Stapler：1](<Stapler  1.md>)🔺                | FTP匿名登录、smb用户枚举、hydra爆破、文件隐写、wordpress、MySQL提权、字典爆破、EXP本地提权 | ⚪        | 8 Jun 2016  |
|               [PwnLab：init](PwnLab：init.md)⚽               |        本地文件包含、文件上传绕过、环境变量、本地提权        | ⚪        | 1 Aug 2016  |
|                 [Mr-Robot:1](Mr-Robot：1.md)                 |                   wordpress爆破、nmap提权                    | ⚪        | 28 Jun 2016 |
|            [HackLAB：vulnix](HackLAB：vulnix.md)             |         finger枚举用户、**nfs用户伪造访问**、ssh爆破         | ⚪        | 10 Sep 2012 |
|                  [VulnOS 2](<VulnOS 2.md>)                   |                      SQL注入、内核提权                       | ⚪        | 17 May 2016 |
|                [SickOs 1.2](<SickOs 1.2.md>)                 |             **put方法上传文件**、chkrootkit提权              | ⚪        | 27 Apr 2016 |
|                 [PWNOS: 1.0](PWNOS：1.0.md)                  |                CGI文件包含提权、破壳shell提权                | ⚪        | 27 Jun 2008 |
|                  [Pwnos 2.0](PWNOS：2.0.md)                  |                           注入漏洞                           | ⚪        | 4 Jul 2011  |
|     [BSides Vancouver 2018](<BSides Vancouver 2018.md>)      |               FTP匿名登陆、crontab定时计划提权               | ⚪        | 21 Mar 2018 |
|    [Lord of the Root 1.0.1](<Lord of the Root 1.0.1.md>)     |              端口敲门、UDF提权、缓冲区溢出漏洞               | ⚪        | 23 Sep 2015 |
|                [Brainpan:1](<Brainpan 1.md>)🎃                |                        缓冲区溢出漏洞                        | ⚪        | 20 Mar 2013 |
|               [Lin.Security](Lin.Security.md)                |      mount写公钥、strace提权、awk提权、docker 映射提权       | ⚪        | 11 Jul 2018 |
|                   [ZICO2：1](ZICO2：1.md)                    |                文件包含漏洞、zip提权、tar提权                | ⚪        | 19 Jun 2017 |
|           [Web Developer 1](<Web Developer 1.md>)            |             WP插件写shell、tcpdump提权、lxd提权              | ⚪        | 5 Nov 2018  |
|                 [SolidState](SolidState.md)                  |              James服务攻击、rbash转义、pspy使用              | ⚪        | 12 Sep 2018 |
|                 [Wintermute](Wintermute.md)                  |  gobuster使用、GNU Screen提权、mail日志+LFI、socat流量转发   | ⚪        | 5 Jul 2018  |
|       [Wallaby's Nightmare](<Wallaby's Nightmare.md>)        |                         irssi聊天室                          | ⚪        | 22 Dec 2016 |
|                     [DC: 1](<DC  1.md>)                      |                   droopescan使用、find提权                   | ⚪        | 28 Feb 2019 |
|                     [DC: 2](<DC  2.md>)                      |        wpscan使用、vim绕过bash、环境变量配置、git提权        | ⚪        | 22 Mar 2019 |
|                   [DC: 3.2](<DC  3.2.md>)                    |            droopescan使用、sqlmap使用、反弹shell             | ⚪        | 25 Apr 2020 |
|                     [DC: 4](<DC  4.md>)                      |                        hydra暴力破解                         | ⚪        | 7 Apr 2019  |
|                     [DC: 5](<DC 5.md>)⚽                      |           wfuzz使用、LFI日志+反弹shell、screen提权           | ⚪        | 21 Apr 2019 |
|                     [DC: 6](<DC  6.md>)                      |           wpscan爆破用户、wfuzz爆破密码、nmap提权            | ⚪        | 26 Apr 2019 |
|                     [DC: 7](<DC  7.md>)                      |                             社工                             | ⚪        | 31 Aug 2019 |
|                     [DC: 8](<DC  8.md>)                      |                          exim4提权                           | ⚪        | 8 Sep 2019  |
|                      [DC: 9](<DC 9.md>)                      |           sqlmap、hydra、端口敲门、passwd插入root            | ⚪        | 29 Dec 2019 |
|             [Bill Madison](<BILLY MADISON 1.1>)              |     smb共享、FTP爆破、端口敲门、数据包分析、WIFI密码爆破     | ⚪        | 14 Sep 2016 |
|        [/dev/random:sleepy](<DEVRANDOM  SLEEPY.md>)🎃         |                             JDWP                             |          | 2 Oct 2015  |
|                   [Troll 1](<Troll 1.md>)                    |                   FTP匿名登录、数据包分析                    | ⚪        | 14 Aug 2014 |
|                   [Troll 2](<Troll 2.md>)                    |                   破壳漏洞、缓冲区漏洞提权                   | ⚪        | 24 Oct 2014 |
|                   [Troll 3](<Troll 3.md>)                    |                       aircrack-ng爆破                        | ⚪        | 6 Aug 2019  |
|                        [IMF](IMF.md)                         |          端口敲门、SQL注入、weevely使用、缓冲区溢出          | ⚪        | 30 Oct 2016 |
|                 [Sky Tower](<Sky Tower.md>)                  |           squid反向代理、.bashrc限制、文件路径滥用           | ⚪        | 26 Jun 2014 |
|          [Pinkys Palace v1](<Pinkys Palace v1.md>)🎃          |                    squid代理、缓冲区溢出                     |          | 6 Mar 2018  |
|          [Pinkys Palace v2](<Pinkys Palace v2.md>)🎃          |                                                              |          | 18 Mar 2018 |
|          [Pinkys Palace v3](<Pinkys Palace v3.md>)🎃          |                                                              |          | 15 May 2018 |
|          [Pinkys Palace v4](<Pinkys Palace v4.md>)🎃          |                                                              |          | 15 Oct 2018 |
|                   [Violator](Violator.md)                    |                             cewl                             | ⚪        | 4 Jul 2016  |
|                      [Sedna](Sedna.md)                       |                              无                              | ⚪        | 14 Mar 2017 |
|                     [Simple](Simple.md)                      |                              无                              | ⚪        | 9 Oct 2015  |
| [digitalworld.local  MERCY v2](<digitalworld.local  MERCY v2.md>)🔺 |                      smb共享、端口敲门                       | ⚪        | 28 Dec 2018 |
| [digitalworld.local: DEVELOPMENT](<digitalworld.local  DEVELOPMENT.md>)🔺 |                           bash绕过                           | ⚪        | 28 Dec 2018 |
| [digitalworld.local: BRAVERY](<digitalworld.local  BRAVERY>)🔺 |                         NFS、smb共享                         | ⚪        | 28 Dec 2018 |
|   [digitalworld.local: JOY](<digitalworld.local  JOY.md>)🔺   |                         ftp匿名访问                          | ⚪        | 31 Mar 2019 |
| [digitalworld.local: TORMENT](<digitalworld.local  TORMENT.md>)🔺 |                     ftp匿名访问、聊天室                      | ⚪        | 31 Mar 2019 |
| [digitalworld.local: snakeoil](<digitalworld.local  snakeoil.md>)🔺 |                      ffuf使用，API利用                       | ⚪        | 23 Aug 2021 |
|  [digitalworld.local: FALL](<digitalworld.local  FALL.md>)   |                           ffuf使用                           | ⚪        | 6 Sep 2021  |
|                   [Raven:1](<Raven 1.md>)                    |                            wpscan                            | ⚪        | 14 Aug 2018 |
|                   [Raven:2](<Raven 2.md>)                    |                              无                              | ⚪        | 9 Nov 2018  |
|            [Temple of Doom](<Temple of Doom.md>)             |                     nodejs和ss反序列漏洞                     | ⚪        | 8 Jun 2018  |
|                 [Hackme：1](<HACKME  1.md>)                  |                      SQL注入、文件上传                       | ⚪        | 18 Jul 2019 |
|                 [Hackme：2](<HACKME  2.md>)                  |                       命令注入空格绕过                       | ⚪        | 6 Dec 2020  |
|         [Escalate_Linux: 1](<Escalate Linux：1.md>)          |                          Linux提权                           | ⚪        | 30 Jun 2019 |
|                   [Prime 1](<Prime 1.md>)                    |           wfuzz、wordpress theme上传shell、AES解密           | ⚪        | 1 Sep 2019  |
|            [Misdirection 1](<Misdirection 1.md>)🔺            |            sudo指定用户、lxd提权、passwd新增用户             | ⚪        | 24 Sep 2019 |
|                    [Sar：1](<Sar：1.md>)                     |                      LinEnum、定时计划                       | ⚪        | 15 Feb 2020 |
|                  [DJINN: 1](<DJINN  1.md>)                   |                             提权                             | ⚪        | 18 Nov 2019 |
|                    [EVM: 1](<EVM  1.md>)                     |                    wpscan、msf连wordpress                    | ⚪        | 2 Nov 2019  |
|                   [NullByte](NullByte.md)⚽                   |                 exiftool、hashcat、环境变量                  | ⚪        | 1 Aug 2015  |
|                   [Toppo 1](<Toppo 1.md>)                    |                      sudo中awk、python                       | ⚪        | 12 Jul 2018 |
|               [LemonSqueezy](LemonSqueezy.md)                |                wpscan枚举和爆破，MySQL写shell                | ⚪        | 26 Apr 2020 |
|                     [Tiki-1](Tiki-1.md)                      |                         smb匿名共享                          | ⚪        | 31 Jul 2020 |
|              [Healthcare 1](<Healthcare 1.md>)               |                        gobuster、suid                        | ⚪        | 29 Jul 2020 |
|            [Photographer 1](<Photographer 1.md>)             |                     smb匿名共享、php提权                     | ⚪        | 21 Jul 2020 |
|              [Glasglow 1.1](<Glasglow 1.1.md>)               |                        joomscan、cewl                        | ⚪        | 16 Jun 2020 |
|                 [DevGuru 1](<DevGuru 1.md>)                  |   git-dumper、bcrypt加密、Git Hooks反弹shell、sudoers 提权   | ⚪        | 7 Dec 2020  |
|            [Hack Me Please](<Hack Me Please.md>)             |                              无                              | ⚪        | 31 Jul 2021 |
|       [Vulnerable Docker 1](<Vulnerable Docker 1.md>)        |                         docker未授权                         | ⚪        | 27 Sep 2017 |
|                  [Readme 1](<Readme 1.md>)🎃                  |                                                              |          |             |
|                [Election 1](<Election 1.md>)                 |                          Serv-U提权                          | ⚪        | 2 Jul 2020  |
|         [Hacker Kid: 1.0.1](<Hacker Kid 1.0.1.md>)🔺          |            SSTI反弹shell、dig查询域名、getcap提权            | ⚪        | 2 Aug 2021  |
|     [Infosec Prep OSCP Box](<Infosec Prep OSCP Box.md>)      |                           lxd提权                            | ⚪        | 11 Jul 2020 |
|                   [HAWordy](HA：WORDY.md)🔺                   |                     wpscan使用、suid提权                     | ⚪        | 13 Sep 2019 |
|                 [Bottleneck](Bottleneck.md)🎃                 |                          LFI、suid                           | ⚪        | 28 Sep 2019 |
|                    [Lampiao](Lampiao.md)🔺                    |                         hydra、cewl                          | ⚪        | 28 Jul 2018 |
|              [BORN2ROOT: 1](<Born2Root：1.md>)🔺              |                      bopscrk、定时计划                       | ⚪        | 10 Jun 2017 |
|              [BORN2ROOT: 2](<Born2Root  2.md>)🔺              |                             cewl                             | ⚪        | 28 Feb 2019 |
|                  [BTRSys2.1](BTRSys2.1.md)🔺                  |                            wpscan                            | ⚪        | 31 Jul 2017 |
|                       [Dawn](Dawn.md)🔺                       |                   smb共享、pspy64、zsh提权                   | ⚪        | 3 Aug 2019  |
|                      [Dawn2](Dawn2.md)🎃                      |                                                              |          | 15 Feb 2020 |
|                      [Dawn3](Dawn3.md)🎃                      |                                                              |          | 8 Mar 2020  |
|                     [NoName](NoName.md)🔺                     |              命令执行、nc.traditional、find提权              | ⚪        | 15 Feb 2020 |
|              [Inclusiveness](Inclusiveness.md)🔺              |          文件包含反弹shell和VSFTP、环境变量绕过提权          | ⚪        | 10 Feb 2020 |
|                   [Solstice](Solstice.md)🔺                   |                                                              |          | 26 Jun 2020 |
|                   [My-CMSMS](My-CMSMS.md)🔺                   |                                                              |          | 25 Jun 2020 |
|                       [Sumo](Sumo.md)🔺                       |                                                              |          | 13 May 2020 |
|                  [Powergrid](Powergrid.md)🔺                  |                                                              |          | 28 May 2020 |
|                     [Geisha](Geisha.md)🔺                     |                                                              |          | 13 May 2020 |
|                     [Katana](Katana.md)🔺                     |                                                              |          | 13 May 2020 |
|                  [Ha-natraj](Ha-natraj.md)🔺                  |                                                              |          | 4 Jun 2020  |
|                     [Funbox](Funbox.md)🔺                     |                                                              |          | 20 Jul 2020 |
|                    [Gitroot](Gitroot.md)⚽                    |                   git-dumpers、git-export                    | ⚪        | 3 Jun 2020  |
|                    [Seppuku](Seppuku.md)🔺                    |                       hydra、rbash转义                       | ⚪        | 13 May 2020 |
|                   [SoSimple](SoSimple.md)🔺                   |              wpscan、wordpress提权、service提权              | ⚪        | 17 Jul 2020 |
|               [Assertion101](Assertion101.md)🔺               |                  文件包含漏洞、aria2c SUID                   | ⚪        | 28 Jun 2020 |
|                     [Pwned1](Pwned1.md)🎃                     |                                                              |          | 10 Jul 2020 |
|              [Sunset：Decoy](Sunset：Decoy.md)🔺              |             rbash绕过、环境变量、Chkrootkit提权              | ⚪        | 7 Jul 2020  |
|            [CYBERSPLOIT: 1](<CYBERSPLOIT  1.md>)🔺            |   [CyberChef](https://gchq.github.io/CyberChef/)解密二进制   | ⚪        | 9 Jul 2020  |
|                     [Djinn3](Djinn3.md)🔺                     |                             SSTI                             | ~~⚪~~    | 19 Jun 2020 |
|                     [Potato](Potato.md)🔺                     |                        目录遍历、john                        | ⚪        | 2 Aug 2020  |
|               [FunboxRookie](FunboxRookie.md)🔺               |                      zip2john压缩包爆破                      | ⚪        | 27 Jul 2020 |
|               ~~[FunboxEasy](FunboxEasy.md)~~                |                       ~~虚拟机有问题~~                       | ⚪        | 31 Jul 2020 |
|                      [PyExp](PyExp.md)🔺                      |                    mysql爆破、fernet解密                     | ⚪        | 11 Aug 2020 |
|                       [Loly](Loly.md)🔺                       |                     wpscan枚举用户和密码                     | ⚪        | 21 Aug 2020 |
|                       [Wpwn](Wpwn.md)🔺                       |                           命令执行                           | ⚪        | 18 Aug 2020 |
|             [SunsetNoontide](SunsetNoontide.md)              |                              无                              | ⚪        | 9 Aug 2020  |
|            [InsanityHosting](InsanityHosting.md)🔺            |                        SQL注入、john                         | ⚪        | 16 Aug 2020 |
|                [BBSCute](<BBSCute 1.0.2.md>)                 |                              无                              | ⚪        | 24 Sep 2020 |
|             [FunboxEasyEnum](FunboxEasyEnum.md)              |                              无                              | ⚪        | 19 Sep 2020 |
|                 [Monitoring](Monitoring.md)                  |                              无                              | ⚪        | 14 Sep 2020 |
|                     [Y0usef](Y0usef.md)⚽                     |                     X-Forwarded-For绕过                      | ⚪        | 10 Dec 2020 |
|                      [Gaara](Gaara.md)🔺                      |                           gdb提权                            | ⚪        | 13 Dec 2020 |
|                   [TRE: 1](<TRE  1.md>) 🔺                    |                          pspy64使用                          | ⚪        | 13 May 2020 |
|          [SUNSET: MIDNIGHT](<SUNSET MIDNIGHT.md>)🔺           |                      wordpress木马插件                       | ⚪        | 19 Jul 2020 |
|                        [Ted](Ted.md)⚽                        |                                                              |          |             |
|                  [Five86.2](<Five86.2.md>)⚽                  |                                                              |          |             |
|              [LazySysAdmin](<LazySysAdmin.md>)⚽              |                                                              |          |             |
|                  [Covfefe](<Covfefe.md>) ⚽                   |                                                              |          |             |
|             [BrokenGallery](<BrokenGallery.md>)⚽             |                                                              |          |             |
|                 [Powergrid](<Powergrid.md>)⚽                 |                                                              |          |             |
|                  [DepthB2R](<DepthB2R.md>)⚽                  |                                                              |          |             |
|              [Scarecrow1.1](<Scarecrow1.1.md>)⚽              |                                                              |          |             |
|                  [Fowsniff](<Fowsniff.md>)⚽                  |                                                              |          |             |
|            [SunsetTwilight](<SunsetTwilight.md>)⚽            |                                                              |          |             |
|                   [USV2017](<USV2017.md>)⚽                   |                                                              |          |             |
|                    [JISCTF](<JISCTF.md>)⚽                    |                                                              |          |             |
|            [BossPlayersCTF](<BossPlayersCTF.md>)⚽            |                                                              |          |             |


## 其他

|                        靶场名字                        |                   学习到内容                   | 是否完成 | 靶场时间    |
| :----------------------------------------------------: | :--------------------------------------------: | -------- | ----------- |
|          [GREENOPTIC: 1](<GREENOPTIC  1.md>)           | DNS域传送漏洞、LFI+邮件、FTP+SSH、数据包分析。 |          |             |
|                  [Kvasir](Kvasir.md)                   |                                                |          |             |
|                     [xxe](xxe.md)                      |                                                |          |             |
|            [Darknet：1.0](Darknet：1.0.md)             |                                                |          |             |
|           [ACID：SERVER](<ACID  SERVER.md>)🦄           |           gobuster使用、命令执行漏洞           |          |             |
|         [ACID：RELOADED](<ACID  RELOADED.md>)🦄         |               端口敲门，strings                |          |             |
|          [DERPNSTINK: 1](<DERPNSTINK  1.md>)🦄          |                                                |          |             |
|  [RICKDICULOUSLYEASY: 1](<RICKDICULOUSLYEASY  1.md>)🦄  |                    初级CTF                     |          |             |
|           [TOMMY BOY: 1](<TOMMY BOY  1.md>)🦄           |                                                |          |             |
|                [Breach 1](Breach 1.md)🦄                |                                                |          |             |
|             [Breach 2.1](<Breach 2.1.md>)🦄             |                                                |          |             |
|           [Breach 3.0.1](<Breach 3.0.1.md>)🦄           |                                                |          |             |
|               [Bob 1.0.1](Bob 1.0.1.md)🦄               |                                                |          |             |
|             [W34kn3ss 1](<W34kn3ss 1.md>)🦄             |         openssl漏洞、pyc转py、SSL证书          | ⚪        | 14 Aug 2018 |
|            [GoldenEye 1](<GoldenEye 1.md>)🦄            |                 html解码、POP3                 | ~~⚪~~    | 4 May 2018  |
|             [MORIA  1.1](<MORIA  1.1.md>)🦄             |                                                |          | 29 Apr 2017 |
|               [Spydersec](Spydersec.md)🦄               |                                                |          | 4 Sep 2015  |
| [DEFCON Toronto Galahad](<DEFCON Toronto Galahad.md>)🦄 |                                                |          | 1 Jun 2017  |
|                 [Node 1](<Node 1.md>)                  |                                                |          | 7 Aug 2018  |
|               [Deception](Deception.md)🦄               |                       无                       |          | 15 Feb 2020 |
|                 [Vegeta1](Vegeta1.md)🦄                 |                音频摩斯密码解密                | ⚪        | 28 Jun 2020 |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |
|                                                        |                                                |          |             |

> - 🔺：有意思
> - 🎃：pwn
> - 🦄：CTF
> - ⚽：退役机器��