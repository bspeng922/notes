## TripleO


> TripleO的目标是简化Openstack部署，它开发了一个自运维的Opestack基础设施， 它由裸机安装部分(nova baremetal or ironic), 软件安装部分(diskimage-builder), Orchestration工具（Heat），镜像内DevOps工具(Puppet Or Chef)组成。

和其他部署工具不同，TripleO利用Openstack本来的基础设施来部署Openstack，基于Nova, Neutron，Ironic和Heat，来自动化部署和伸缩Openstack集群。


#### diskimage-builder

diskimage-builder就是为了解决这个问题而生的，通过一次生成镜像的过程把所有的安装全部完成，然后部署机器的时候只需要把镜像复制过去。
另外它也可以可以用来定制ramdisk，这个ramdisk通过PXE启动Client端以接受复制。

这里定制最大的好处在于所有软件只用安装一次，安装到镜像中去，最后dd到操作系统上去，相当于所有的安装操作只用执行一次，其余的全是复制。


#### tripeo-image-elements

部署openstack的镜像生成element，这些element中定义了一部分的软件和配置，这些软件和配置是给制作diskimage-builder用的，通过这些element制作出来的镜像。


#### tripeo-heat-template

部署openstack的heat模板， 描述glance， keystone还有其他服务。
sec

#### os-apply-config

将metadata中的信息应用到对应机器上去。

{“keystone”: {“database”: {“host”: “127.0.0.1″, “user”: “keystone”, “password”: “foobar”}}}
这样的metadata信息转化为
connection = mysql://keystone:foobar@127.0.0.1/keystone 


#### os-refresh-config

这个组件类似于puppet的agent，去服务器上获取metadata信息来修改配置


#### ironic

Ironic实现了物理机的添加，删除，电源管理和安装部署。


社区开发计划：

   1. 目前整个TripleO体系分为三个部分，seedcloud，undercloud和overcloud。

   2. seedcloud是用来安装undercloud的，但是需要手动搭建seedcloud， 它是由台kvm的虚拟机组成。

3. undercloud是物理机cloud，用openstack来管理物理机，换句话说，nova的driver是物理机。

4. overcloud是虚拟化cloud，用openstack来管理虚拟机。

1. 三个cloud变为两个，undercloud和overcloud。

2. 利用heat来实现openstack的HA。

3. undercloud的初始化仍然由seedcloud来完成。

1. 两个cloud最终变为一个。

2. 只有管理员面板上能管理物理机，在物理机上安装服务。

3. Low负载的服务可以被重新安装到虚拟机上，而不是物理机。

4. Glance，Swift，Keystone应该放到1个Cloud上去，没有复制的需求。




TripleO是一个用openstack来部署openstack的工程（即所谓的openstack over openstack)，即先准备一个openstack控制器的镜像(https://github.com/stackforge/diskimage-builder/)，然后用这个镜像通过openstack的bare-metal功能（现在改名了，叫Ironic,https://wiki.openstack.org/wiki/Baremetal)再去部署裸机，再通过heat在裸机上部署openstack。
它包括下列工程：
In-instanceconfigureation management (os-apply-config + os-refresh-config,and/or Chel/Puppet)
diskimage-builder,https://github.com/stackforge/diskimage-builder
triple-image-elements,https://github.com/stackforge/tripleo-image-elements
triple-heat-templates，https://github.com/stackforge/tripleo-heat-templates





Install Undercloud


deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse

deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse

deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse

deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse

deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse

deb-src http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse

deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse

deb-src http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse

deb-src http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse

deb-src http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse

-----------


The solution is 
1. update lvm2 
2. update device-mapper

Order is important!!! If you try to update device-mapper before lvm2, you will likely to get the error

Transaction check error:
  file /usr/lib/systemd/system/blk-availability.service from install of device-mapper-7:1.02.107-5.el7.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64
  file /usr/sbin/blkdeactivate from install of device-mapper-7:1.02.107-5.el7.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64
  file /usr/share/man/man8/blkdeactivate.8.gz from install of device-mapper-7:1.02.107-5.el7.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64

----


+ source /usr/libexec/openstack-tripleo/devtest_variables.sh
/home/stack/instack/instack-undercloud/scripts/instack-virt-setup:行40: /usr/libexec/openstack-tripleo/devtest_variables.sh: 没有那个文件或目录

++ OVERCLOUD_CONTROL_DIB_EXTRA_ARGS='rabbitmq-server cinder-tgt'
++ export OVERCLOUD_BLOCKSTORAGE_DIB_EXTRA_ARGS=cinder-tgt
++ OVERCLOUD_BLOCKSTORAGE_DIB_EXTRA_ARGS=cinder-tgt
+++ dirname /usr/libexec/openstack-tripleo/devtest_variables.sh
++ source /usr/libexec/openstack-tripleo/set-os-type
/usr/libexec/openstack-tripleo/devtest_variables.sh:行135: /usr/libexec/openstack-tripleo/set-os-type: 没有那个文件或目录

cp /home/stack/instack/tripleo-incubator/scripts/* /usr/libexec/openstack-tripleo/ 

------------

++ sudo virsh net-list --all --persistent
++ grep default
++ awk 'BEGIN{OFS=":";} {print $2,$3}'
+ default_net=$'\346\264\273\345\212\250:\346\230\257'
+ state=$'\346\264\273\345\212\250'
+ autostart=$'\346\230\257'
+ '[' $'\346\264\273\345\212\250' '!=' active ']'
+ virsh net-start default
错误：开始网络 default 失败
错误：所需操作无效：网络已经激活

中文
#vi /etc/sysconfig/i18n

LANG="en_US.UTF-8"

或者 
#export LANG="en_US.UTF-8"


+ target_tag=02-lsb
+ date +%s.%N
+ /tmp/tmpXfB3Pn/pre-install.d/02-lsb
install: 无法获取"/opt/stack/lsb-release/lsb_release" 的文件状态(stat): 没有那个文件或目录
INFO: 2016-04-12 10:53:38,148 -- ############### End stdout/stderr logging ###############
ERROR: 2016-04-12 10:53:38,149 --     Hook FAILED.
ERROR: 2016-04-12 10:53:38,150 -- Failed running command ['dib-run-parts', u'/tmp/tmpXfB3Pn/pre-install.d']
  File "/home/stack/instack/instack/instack/main.py", line 153, in main
    em.run()
  File "/home/stack/instack/instack/instack/runner.py", line 79, in run
    self.run_hook(hook)
  File "/home/stack/instack/instack/instack/runner.py", line 174, in run_hook
    raise Exception("Failed running command %s" % command)


---

http://huifeng.me/2015/09/01/NingHao-Browser-sync-01/
Error: Cannot find module 'browser-sync'
    at Function.Module._resolveFilename (module.js:338:15)
    at Function.Module._load (module.js:280:25)
    at Module.require (module.js:364:17)
    at require (module.js:380:17)
    at Object.<anonymous> (/home/stack/rdo-director-ui/gulpfile.js:4:19)
    at Module._compile (module.js:456:26)
    at Object.Module._extensions..js (module.js:474:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Module.require (module.js:364:17)

npm install -g browser-sync

npm init

npm install browser-sync --save-dev
--save-dev 参数用来将 browser-sync 作为开发依赖放到package.json 里去。