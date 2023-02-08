# PWNOS：2.0

下载地址：https://download.vulnhub.com/pwnos/pWnOS_v2.0.7z



## 实战演练

靶机的IP为静态地址10.10.10.100

![](../../.gitbook/assets/1554693388_5caabd0c3e7d2.png)



![](../../.gitbook/assets/1554694670_5caac20e45fbe.png)



![](../../.gitbook/assets/1554694907_5caac2fbf22db.png)



![](../../.gitbook/assets/1554695434_5caac50a20765.png)

这里面有注入漏洞，sqlmap跑一下

![](../../.gitbook/assets/1554695515_5caac55b07d54.png)

注入漏洞

![](../../.gitbook/assets/1554695657_5caac5e91b7d2.png)

写入反弹shell

![](../../.gitbook/assets/1554696629_5caac9b5af8c1.png)



![](../../.gitbook/assets/1554696686_5caac9ee3d127.png)

nc监听

![](../../.gitbook/assets/1554696889_5caacab92274c.png)

查找密码，找到了这个密码不能登录

![](../../.gitbook/assets/1554697172_5caacbd461971.png)

找到了这个密码，可以登录

![](../../.gitbook/assets/1554697328_5caacc70deaa2.png)

另外一种思路

![](../../.gitbook/assets/1554702181_5caadf6522235.png)

blog系统版本

![](../../.gitbook/assets/1554702244_5caadfa4e66f2.png)

查找漏洞的版本

![](../../.gitbook/assets/1554702441_5caae0696e3d7.png)

使用exp

![](../../.gitbook/assets/1554702616_5caae11847de4.png)

不知道为什么没有生成cookie，就这样把。。。
