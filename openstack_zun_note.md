# 安装 ZUN

### 安装 zun

https://review.openstack.org/#/c/504537/11/doc/source/install/controller-install-ubuntu.rst

https://review.openstack.org/#/c/504537/11/doc/source/install/compute-install-ubuntu.rst


### 安装网络驱动 kuryr-libnetwork

https://docs.openstack.org/kuryr-libnetwork/latest/install/compute-install-ubuntu.html

### 安装UI zun-ui

https://docs.openstack.org/zun-ui/latest/install/index.html#manual-installation


```
usage: zun create [-n <name>] [--cpu <cpu>] [-m <memory>] [-e <KEY=VALUE>]
                  [--workdir <workdir>] [--rm] [--label <KEY=VALUE>]
                  [--image-pull-policy <policy>] [--restart <restart>] [-i]
                  [--image-driver <image_driver>]
                  [--security-group <security-group>] [--hint <key=value>]
                  [--net <auto, network=network, port=port-uuid,v4-fixed-ip=ip-addr,v6-fixed-ip=ip-addr>]
                  [--runtime <runtime>] [--hostname <hostname>]
                  <image> ...

usage: zun run [-n <name>] [--cpu <cpu>] [-m <memory>] [-e <KEY=VALUE>]
               [--workdir <workdir>] [--rm] [--label <KEY=VALUE>]
               [--image-pull-policy <policy>] [--restart <restart>] [-i]
               [--image-driver <image_driver>]
               [--security-group <security-group>] [--hint <key=value>]
               [--net <auto, network=network, port=port-uuid,v4-fixed-ip=ip-addr,v6-fixed-ip=ip-addr>]
               [--runtime <runtime>] [--hostname <hostname>]
               <image> ...

def container_create(request, **kwargs):
    args, run = _cleanup_params(CONTAINER_CREATE_ATTRS, True, **kwargs)
    response = None
    if run:
        response = zunclient(request).containers.run(**args)
    else:
        response = zunclient(request).containers.create(**args)
    return response


openstack appcontainer run --name container --net network=$NET_ID cirros ping 8.8.8.8

openstack appcontainer list

openstack appcontainer exec --interactive container /bin/sh

openstack appcontainer stop container

openstack appcontainer delete container

### 拷贝文件

Usage:
zun cp container:src_path dest_path|-
zun cp src_path|- container:dest_path

--------------------------------------------

### 创建网络

neutron API


### 创建路由

neutron API


### 绑定端口

neutron API


### 从镜像创建容器

 zun run --name ubuntu-web --net network=df94a80d-62a9-4551-ae6b-8d0af3f44fc2 ubuntu-webbb tail -f /etc/hosts


### 绑定浮动IP

申请浮动IP  --  获取容器网络port id

Container Addresses
{ "df94a80d-62a9-4551-ae6b-8d0af3f44fc2": [ { "version": 4, "addr": "172.16.1.14", "port": "eb6cafe1-c19c-4555-826d-7188e903566e" } ] }


neutron  floatingip-associate 32c35af6-9d72-416a-a8cc-220522106452 b3f02c2e-aa76-4db6-b8dc-7defcf67b5a3

floating-ip 设备id（device id） -  端口ID

1. 创建端口（指定网络） -- 创建容器（指定端口） -- 生成浮动IP -- 绑定浮动IP
2. 创建容器（指定网络） -- 创建容器 -- 查找端口 -- 生成浮动IP  -- 绑定浮动IP


### 容器初始化

 拼接初始化脚本文件 -- 拷贝zip文件到容器中 -- 执行初始化 （解压zip文件  -- 执行安装脚本 -- 执行init脚本）


### 从容器转为镜像

 zun commit ubuntuc ubuntu-webbb    
 ---- zun pull ubuntu-webbb ----
 zun image-list 


### 从docker file生成镜像 (暂不支持)

 TODO

```


### zun commit 后的镜像磁盘格式为qcow2

```
vim /usr/lib/python2.7/site-packages/zun/image/glance/driver.py

change qcow2 to raw

```




