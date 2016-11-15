# 使用spice

## Controller

```
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/s/spice-html5-0.1.7-1.el7.noarch.rpm
rpm -ivh spice-html5-0.1.7-1.el7.noarch.rpm

yum install spice-server spice-protocol openstack-nova-spicehtml5proxy -y

vim /etc/nova/nova.conf
[default]
vnc_enabled=false
[spice]
html5proxy_host=10.10.52.31
html5proxy_port=6082
keymap=en-us

systemctl stop openstack-nova-novncproxy.service
systemctl disable openstack-nova-novncproxy.service

systemctl enable openstack-nova-spicehtml5proxy.service
systemctl start openstack-nova-spicehtml5proxy.service

systemctl restart httpd.service
```


## Compute

```
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/s/spice-html5-0.1.7-1.el7.noarch.rpm
rpm -ivh spice-html5-0.1.7-1.el7.noarch.rpm

yum install spice-server spice-protocol

vim /etc/nova/nova.conf
[default]
vnc_enabled=false
[spice]
html5proxy_base_url=http://10.10.52.31:6082/spice_auto.html
server_listen=0.0.0.0
server_proxyclient_address=10.10.52.32   <当前compute节点IP>
enabled=true
keymap=en-us

systemctl restart openstack-nova-compute.service
```


安装完成后，需要硬重启虚拟机才能正常使用spice协议

```
[root@controller ~]# nova get-spice-console 123123 spice-html5
+-------------+------------------------------------------------------------------------------------+
| Type        | Url                                                                                |
+-------------+------------------------------------------------------------------------------------+
| spice-html5 | http://10.10.52.31:6082/spice_auto.html?token=17671c6b-fcaf-484f-8311-a128d6428c8a |
+-------------+------------------------------------------------------------------------------------+
```


PS：
Newton 版本需要修改horizon 的local_setting 配置文件，否则页面上还会引用vnc路径，控制台页面提示控制节点“拒绝了我们的连接请求”。 

需要将

```
CONSOLE_TYPE = "AUTO"
```

改为

```
CONSOLE_TYPE = "SPICE"
```




