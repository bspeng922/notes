# librevirt


## 使用 libvirt 的应用程序

仅从本文已经展示的一小部分功能上便可看出 libvirt 提供的强大功能。且如您所愿，有大量应用程序正成功构建于 libvirt 之上。其中一个有趣的应用程序就是 virsh（这里所示），它是一种虚拟 shell。还有一种名为 virt-install 的应用程序，它可用于从多个操作系统发行版供应新域。virt-clone 可用于从另一个 VM 复制 VM（既包括操作系统复制也包括磁盘复制）。一些高级应用程序包括多用途桌面管理工具 virt-manager 和安全连接到 VM 图形控制台的轻量级工具 virt-viewer。

构建于 libvirt 之上的一种最重要的工具名为 oVirt。oVirt VM 管理应用程序旨在管理单个节点上的单个 VM 或多个主机上的大量 VM。除了可以简化大量主机和 VM 的管理之外，它还可用于跨平台和架构自动化集群，负载平衡和工作。


## 常用操作

```

1. 查看当前所有的虚拟机
virsh list --all

2. 查看虚拟机对应的端口号
virsh vncdisplay centos

3. 查看当前的网络
virsh net-list --all

4. 启动default网络
virsh net-start default

5. 创建虚拟磁盘文件
qemu-img create -f qcow2 centos.qcow2 10G

6. 启动虚拟机
virt-install --virt-type kvm --name centos --ram 1024 \
  --disk centos7.qcow2,format=qcow2 \
  --network network=default \
  --graphics vnc,listen=0.0.0.0 --noautoconsole \
  --os-type=linux --os-variant=rhel7 \
  --location=CentOS-7-x86_64-Minimal-1511.iso


7. 关闭虚拟机
virsh shutdown centos

8. 删除虚拟机
virsh destroy centos

9. 删除域
virsh undefine centos

10. 克隆虚拟机
virt-clone \
     --original demo \
     --name newdemo \
     --file /var/lib/xen/images/newdemo.img

11. 暂停虚拟机
virsh suspend centos

12. 继续虚拟机
virsh resume centos

13. 压缩磁盘文件
qemu-img convert -c -O qcow2 centos.qcow2 centos-c.qcow2

14. 查看磁盘文件信息
qemu-img info centos.qcow2

15. 查看虚拟机配置文件
virsh dumpxml centos

16. 编辑虚拟机配置
virsh edit centos

17. 挂载iso文件
virsh attach-disk --type cdrom --mode readonly centos "centos.iso" hdc

18. 删除mac地址文件（linux）
virt-sysprep -d centos

19. 查看虚拟机的IP地址
arp | grep `virsh dumpxml centos-su | sed -n '/mac address/p' | awk -F"[']" '{print $2}'` | awk '{printf $1}'

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




### Failed to bind socket: Permission denied

```
Unable to complete install: 'internal error: process exited while connecting to monitor: 2015-07-29T22:39:54.330608Z qemu-system-x86_64: -chardev socket,id=charchannel0,path=/var/lib/libvirt/qemu/channel/target/centos7_base.org.qemu.guest_agent.0,server,nowait: Failed to bind socket: Permission denied'

解决方法是将xml文件中的 unix 那段删掉
virsh edit centos

```
