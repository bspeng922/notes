### SNMP 学习笔记

简单网络管理协议（SNMP, Simple Network Management Protocol）是由互联网工程任务组（IETF, Internet Engineering Task Force）定义的一套网络管理协议｡

SNMP 采用了 Manager/Agent 的架构，Agent 负责收集信息，Manager 负责更新管理信息库（MIB, Management Information Base）。

Manager 和 Agent 之间使用 SNMP 协议来通信，一共有以下几种消息类型：

+ Get-Request：Manager 向 Agent 请求信息。
+ Get-Next-Request：类似于 Get-Request，但是要求返回下一个节点。
+ Get-Response：Agent 答复 Manager 的请求。
+ Set-Request：Manager 对网络设备进行远程配置。
+ Trap：Agent 主动向服务器推送信息（事件监听）。
+ GetBulk：用于请求大量的数据。


#### 安装
```shell
$ sudo apt-get install -y snmpd
$ sudo service snmpd restart    # 重启服务
```


#### 配置
**配置 SNMP Manager**
/etc/snmp/snmp.conf

**配置 SNMP Agent Machine**
/etc/snmp/snmpd.conf

*修改 agentAddress*
默认是只允许本机连接，将其注释掉，然后去掉下面一行的注释就行
```
# /etc/snmp/snmpd.conf

#  Listen for connections from the local system only
#agentAddress  udp:127.0.0.1:161
#  Listen for connections on all interfaces (both IPv4 *and* IPv6)
agentAddress udp:161,udp6:[::1]:161
```

*添加一个临时的账户*
```
createUser bootstrap MD5 temp_password DES
```

*为用户设置权限*
可以设置为只读 rouser，和读写 rwuser。也可以在最后添加 priv 来强制要求加密。
```
rwuser bootstrap priv
rwuser demo priv  # 为我们之后要创建的用户也设置一个权限
```

