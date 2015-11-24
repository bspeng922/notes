

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
