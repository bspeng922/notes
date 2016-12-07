# librevirt


## 使用 libvirt 的应用程序

仅从本文已经展示的一小部分功能上便可看出 libvirt 提供的强大功能。且如您所愿，有大量应用程序正成功构建于 libvirt 之上。其中一个有趣的应用程序就是 virsh（这里所示），它是一种虚拟 shell。还有一种名为 virt-install 的应用程序，它可用于从多个操作系统发行版供应新域。virt-clone 可用于从另一个 VM 复制 VM（既包括操作系统复制也包括磁盘复制）。一些高级应用程序包括多用途桌面管理工具 virt-manager 和安全连接到 VM 图形控制台的轻量级工具 virt-viewer。

构建于 libvirt 之上的一种最重要的工具名为 oVirt。oVirt VM 管理应用程序旨在管理单个节点上的单个 VM 或多个主机上的大量 VM。除了可以简化大量主机和 VM 的管理之外，它还可用于跨平台和架构自动化集群，负载平衡和工作。


## 常用操作

```
mtj@mtj-desktop:~/libvtest$ virsh create react-qemu.xml
Connecting to uri: qemu:///system
Domain ReactOS-on-QEMU created from react-qemu.xml

mtj@mtj-desktop:~/libvtest$ virsh list
Connecting to uri: qemu:///system
 Id Name                 State
----------------------------------
  1 ReactOS-on-QEMU      running

mtj@mtj-desktop:~/libvtest$ virsh suspend 1
Connecting to uri: qemu:///system
Domain 1 suspended

mtj@mtj-desktop:~/libvtest$ virsh list
Connecting to uri: qemu:///system
 Id Name                 State
----------------------------------
  1 ReactOS-on-QEMU      paused

mtj@mtj-desktop:~/libvtest$ virsh resume 1
Connecting to uri: qemu:///system
Domain 1 resumed

[root@compute2 isos]# virsh vncdisplay centos
:0

[root@compute2 ~]# virsh net-start default
Network default started

[root@compute2 ~]# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     no            yes

```


## python api

高级 libvirt API 可划分为 5 个 API 部分：虚拟机监控程序连接 API、域 API、网络 API、存储卷 API 以及存储池 API。

```
import libvirt

conn = libvirt.open('qemu:///system')

for id in conn.listDomainsID():

    dom = conn.lookupByID(id)

    print "Dom %s  State %s" % ( dom.name(), dom.info()[0] )

    dom.suspend()
    print "Dom %s  State %s (after suspend)" % ( dom.name(), dom.info()[0] )

    dom.resume()
    print "Dom %s  State %s (after resume)" % ( dom.name(), dom.info()[0] )

    dom.destroy()
```



### error: Network not found: no network with matching name 'default'

```shell
# virsh create rhel7.xml 
error: Failed to create domain from rhel7.xml
error: Network not found: no network with matching name 'default'

# virsh net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     no            yes

# virsh net-define /dev/stdin <<EOF
> <network connections='1'>
>   <name>default</name>
>   <uuid>18bb3790-11ee-41b1-8847-970590a06e4d</uuid>
>   <forward mode='nat'/>
>   <bridge name='virbr0' stp='on' delay='0' />
>   <mac address='52:54:00:F6:C8:DE'/>
>   <ip address='192.168.122.1' netmask='255.255.255.0'>
>     <dhcp>
>       <range start='192.168.122.2' end='192.168.122.254' />
>     </dhcp>
>   </ip>
> </network>
> EOF
Network default defined from /dev/stdin

# virsh net-start default
Network default started

# virsh net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     no            yes

# virsh create rhel7.xml 
Domain rhel7 created from rhel7.xml

# virsh list
 Id    Name                           State
----------------------------------------------------
 15    rhel7                          running

##############################################################
# ifconfig virbr0 down
# virsh net-start default
error: Failed to start network default
error: Unable to create bridge virbr0: File exists

# ifconfig virbr0 up
# virsh net-start default
error: Failed to start network default
error: internal error: Network is already in use by interface virbr0

# ifconfig virbr0 down
# brctl delbr virbr0
# virsh net-autostart default
Network default marked as autostarted
# virsh net-start default
Network default started

```

