

1. **提示找不到oslo.config**
oslo新版本包名更换为oslo_config等，需要使用旧版本的oslo库，相应的client也需要使用旧版本

2. **debug的时候提示/bin/sh: 1: lesscpy: not found**
it looks like python-lesscpy isn't a hard dependency of the Ubuntu package, so you probably need to install it explictly if you want to use compression:
sudo apt-get install python-lesscpy
[https://bugs.launchpad.net/horizon/+bug/1242766](https://bugs.launchpad.net/horizon/+bug/1242766)

3. **提示无法引入base等包**
openstack/horizon/openstack_dashboard/api/__init__.py
去掉前面的from openstack_dashboard.api
[https://bugs.launchpad.net/horizon/+bug/1125622](https://bugs.launchpad.net/horizon/+bug/1125622)

4. **执行neutron命令的时候提示 unsupported locale setting**
export LC_ALL=C，然后再执行neutron命令
[https://bugs.launchpad.net/devstack/+bug/1249131](https://bugs.launchpad.net/devstack/+bug/1249131)

5. **登陆后无法进入Overview页面，提示Connection to neutron failed: Maximum attempts reached**
无法连接neutron，查看neutron配置文件，查看配置的IP本地是否能ping通


6. **error: [Errno 32] Broken pipe**


7. **分配floating ip后无法ping通，但是从路由上可以ping通私网地址**
需要先创建安全组规则，添加ICMP进出规则：ALL ICMP

8. **Python打包格式**
+ tar.gz格式：这个就是标准压缩格式，里面包含了项目元数据和代码，可以使用python setup.py sdist命令生成。
+ egg格式：这个本质上也是一个压缩文件，只是扩展名换了，里面也包含了项目元数据以及源代码。这个格式由setuptools项目引入。可以通过命令python setup.py bdist_egg命令生成。
+ whl格式：这个是Wheel包，也是一个压缩文件，只是扩展名换了，里面也包含了项目元数据和代码，还支持免安装直接运行。whl分发包内的元数据和egg包是有些不同的。这个格式是由PEP 427引入的。可以通过命令python setup.py bdist_wheel生成。

9. 将vdx 更改为sdx
可以通过修改镜像的属性（hw_disk_bus）来实现硬盘设备名称的更改，也可以在创建镜像的时候添加,更新完之后通过镜像创建云主机，硬盘设备名称将会变成sdxx
```
$ glance image-update \
 --property hw_disk_bus=scsi \
 --property hw_cdrom_bus=ide \
 --property hw_vif_model=e1000 \
 IMAGEID
```

    **Disk and CD-ROM bus model values**
    qemu or kvm 

    + virtio
    + scsi
    + ide
    + virtio

    **VIF model values**
    qemu or kvm 

    + virtio
    + ne2k_pci
    + pcnet
    + rtl8139
    + e1000

    vmware  

    + VirtualE1000
    + VirtualPCNet32
    + VirtualVmxnet

    参考页面：[Create or update an image (glance)](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/5/html/Administration_User_Guide/cli_manage_images.html)

