# Openstack Windows 镜像制作

### 下载virtio驱动文件
地址：
smb://192.168.51.100/platform/01Software/02Image/windows/virtio-win-0.1.102_amd64.vfd
smb://192.168.51.100/platform/01Software/02Image/windows/virtio-win-0.1.102.iso

> 官方下载： [https://fedoraproject.org/wiki/Windows_Virtio_Drivers](https://fedoraproject.org/wiki/Windows_Virtio_Drivers)

### 创建一个img文件
```shell
$ qemu-img create -f qcow2 win7_x64.qcow2 40G
```

### 安装系统

```shell
$ kvm -m 2048 -cdrom cn_windows_7_ultimate_with_sp1_x64_dvd_u_677408.iso -drive file=win7_x64.qcow2,if=virtio -fda virtio-win-0.1.102_amd64.vfd -boot d -nographic -vnc :0
```

**连接VNC进行系统安装**
这里的端口号根据上一步 -vnc :0 推移，如果是-vnc :1则是5901端口
```shell
$ vncview localhost:5900
```

**磁盘选择**
安装选择磁盘时显示是空的，这个时候需要手动去加载驱动
步骤： 加载驱动程序 -> 确定 -> 软盘驱动器A -> i386 -> win7 -> 继续
选择硬盘后就按照正常系统安装方式进行安装，安装完成后关闭系统。

### 安装驱动
需要安装网卡驱动、显卡驱动（qxl）、声卡驱动（ac94）

**安装网卡驱动**
```shell
$ kvm -m 2048 -cdrom virtio-win-0.1.102.iso -drive file=win7_x64.qcow2,if=virtio -net nic,model=virtio -net user -boot d -nographic -vnc :0
```
安装完网卡驱动后关闭电脑，用命令重新开机, 也可以和网卡驱动一起安装
```shell
$ kvm -m 2048 -drive file=win7_x64.qcow2,if=virtio -net nic,model=virtio -net user -soundhw ac97 -vga qxl -boot d -nographic -vnc :0
```
**安装显卡、声卡驱动**
驱动地址：smb://192.168.51.100/platform/01Software/03Windows/kvm drivers
将驱动拷贝到系统桌面，然后在设备管理页面更新驱动程序，选择对应的驱动目录。

> 如果需要安装一些常用软件，可以再次打开系统进行安装
> 网络参数可以不加，默认即为 -net nic -net user
> 如果安装时鼠标不能移动，需要加上参数 -usb -usbdevice tablet 

### 镜像压缩
系统安装完之后可能会占用很大空间，如果对大小有要求可以使用命令压缩一下
```shell
$ qemu-img convert -c -O qcow2 win7_x64.qcow2 win7_x64_new.qcow2
$ ll -h
-rw-r--r--  1 cctdesktop cctdesktop  13G 11月 13 14:07 win7_x64.qcow2
-rw-r--r--  1 cctdesktop cctdesktop 6.3G 11月 13 14:21 win7_x64_new.qcow2
```

### 上传镜像
```shell
$ glance image-create --name="Win7 x64" --is-public=true --container-format=ovf --disk-format=qcow2 < win7_x64_new.qcow2
```

### 调整源磁盘大小
```shell
#qemu-img resize filename [+|-]size
```

当使用此命令收缩磁盘镜像之前,必须使用客户机的文件系统和分区工具来收缩文件系统和分区,然后再执行resize操作,不然会可能丢失数据。当使用此命令扩大了磁盘镜像之后,必须使用客户机的文件系统和分区工具来使用新增加的磁盘容量。这很好理解,KVM支持的客户机操作系统多种多样,而且都有成熟的文件系统和分区操作工具,resize操作只是简单的扩大或缩小磁盘镜像大小,而不能也无需来了解客户机怎么应对这个改变,这是客户机的事情。

不幸的是resize尚不支持qcow2格式的磁盘镜像收缩, 但是扩大qcow2磁盘镜像没有问题。

**使用命令查看qcow2文件信息**

```shell
$ qemu-img info win2008_r2_x64.qcow2 
image: win2008_r2_x64.qcow2
file format: qcow2
virtual size: 60G (64424509440 bytes)
disk size: 7.1G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
```



***


# Openstack Ubuntu 镜像制作
sudo apt-get install kvm-pxe

wget http://mirrors.usc.edu/pub/linux/distributions/centos/6.2/isos/x86_64/CentOS-6.2-x86_64-LiveDVD.iso

kvm-img create -f qcow2 centos.qcow2 10G

sudo kvm -m 1024 -cdrom CentOS-6.2-x86_64-LiveDVD.iso -drive file=centos.qcow2,if=virtio,index=0 -boot d -net nic -net user -nographic -vnc :2

vncviewer :2

sudo kvm -m 1024 -drive file=centos.qcow2,if=virtio,index=0,boot=on -boot c -net nic -net user -nographic -vnc :2


## Openstack 直接使用原始启动盘作为镜像
创建镜像时直接上传原始启动盘即可