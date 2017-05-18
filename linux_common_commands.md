### linux常用命令


+ 查看登陆日志
```shell
root@compute2:~# last
root     pts/0        192.168.50.101   Tue Dec 15 13:56   still logged in   
root     pts/16       192.168.50.101   Mon Dec 14 15:19 - 18:05  (02:45)    
root     pts/0        192.168.50.116   Fri Dec 11 00:56 - 10:33 (4+09:36)   
root     tty1                          Fri Dec 11 00:49   still logged in   
root     pts/2        192.168.13.11    Fri Dec 11 00:47 - 00:47  (00:00)    
```

+ 查看端口占用
```
root@compute2:~# netstat -nl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN 
tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN     
udp        0      0 0.0.0.0:43159           0.0.0.0:*                         
```

+ 查看端口被哪个进程占用
```
root@compute2:~# lsof -i :5900
COMMAND    PID         USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
qemu-syst 6902 libvirt-qemu   15u  IPv4 594323      0t0  TCP *:5900 (LISTEN)
qemu-syst 6902 libvirt-qemu   97u  IPv4 629295      0t0  TCP 192.168.13.32:5900->192.168.13.11:54428 (ESTABLISHED)
```

+ 查看外面的流量出口
```
$ sudo iftop -np 
```

+ 查看有无异常进程
```shell
$ ps aux 
```

+ 查看系统资源占用有无异常
```shell
$ top
```

+ 有没有新增异常用户
```shell
$ cat /etc/passwd
```

+ 查看了root用户的命令历史记录
```shell
$ history
```

+ 查看当前在线终端
```shell
root@controller:~# w
 14:36:55 up 4 days,  5:11,  3 users,  load average: 1.52, 0.95, 1.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/11   192.168.50.101   13:50   45:59   0.09s  0.09s -bash
root     pts/0    192.168.50.101   13:49    7.00s  0.07s  0.00s w
root     pts/4    192.168.50.163   Mon09    1:51   1.81s  1.81s -bash
```

+ 向终端发送一条信息
```shell
root@controller:~# write root pts/11
what are you doing

ctrl+d    #结束消息
```

+ 踢掉在线终端
```shell
root@controller:~# pkill -kill -t pts/4
```

+ 查看授权日志
```shell
$ sudo tail -f /var/log/auth.log
```

+ 禁止IP访问
```shell
$ sudo vi /etc/hosts.deny
```

+ 禁止ssh root登陆
```shell
$ sudo vi /etc/ssh/sshd_config
#把PermitRootLogin 属性 yes 改为 no
PermitRootLogin no
```

