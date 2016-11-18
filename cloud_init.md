# cloud init

User configurability
Cloud-init ‘s behavior can be configured via user-data.

User-data can be given by the user at instance launch time.
This is done via the --user-data or --user-data-file argument to ec2-run-instances for example.


### 执行shell脚本

Begins with: #! or Content-Type: text/x-shellscript when using a MIME archive.

Example

```shell
$ cat myscript.sh

#!/bin/sh
echo "Hello World.  The time is now $(date -R)!" | tee /root/output.txt
```

修改ubuntu的默认密码，并开启ssh密码登录

```shell
#!/bin/sh
passwd ubuntu<<EOF
ubuntu
ubuntu
EOF
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service ssh restart
```

```
#cloud-config
ssh_pwauth: true
disable_root: 0
user: root
password: abc123
chpasswd:
  expire: false
```

首先, 这个文件把一般云主机里面的 ssh 密码认证打开, 默认关闭的
然后, 通过 disable_root 允许 root 认证, 默认关闭
将 root 的密码设置为 abc123
如果不设置 chpasswd 的 expire 为 false, 那么登陆的时候会提示马上修改密码才能进去


### 同时传递多种格式/多个文件的 user-data

```shell
Content-Type: multipart/mixed; boundary="===============2197920354430400835=="
MIME-Version: 1.0

--===============2197920354430400835==
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloudconfig.txt"

#cloud-config
runcmd:
 - touch /root/cloudconfig
 - source: "ppa:smoser/ppa"

--===============2197920354430400835==
Content-Type: text/x-include-url; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="includefile.txt"

# these urls will be read pulled in if they were part of user-data
# comments are allowed.  The format is one url per line
http://www.ubuntu.com/robots.txt

--===============2197920354430400835==
Content-Type: text/cloud-boothook; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="boothook.txt"

#!/bin/sh
echo "Hello World!"

--===============2197920354430400835==
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userscript.txt"

#!/usr/bin/perl
print "This is a user script (rc.local)\n"

--===============2197920354430400835==--
```



### 无法获取控制台日志

如果无法获取到云主机日志，则需要修改grub文件配置

+ ubuntu 

```shell
# vim /etc/default/grub
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200"

# update-grub           #ubuntu
```

+ centos

```shell
# vi /boot/grub2/grub.cfg
## append "console=tty0 console=ttyS0,115200" to "CentOS ... (Core)" line
```


### 修改云主机domain

默认虚拟机的domain是novadomain，如果要自定义domain名称，需要修改nova的配置文件

```shell
# vi /etc/nova/nova.conf
dhcp_domain=testdomain

# systemctl restart openstack-nova-api.service
```

