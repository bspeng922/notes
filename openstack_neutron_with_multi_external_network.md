# Openstack 创建多个外部网络

Vxlan + linux bridge

1. 修改ml2_conf.ini文件（控制节点）
    
```
# vim /etc/neutron/plugins/ml2/ml2_conf.ini

flat_networks = provider,admodel
```

2. 修改linuxbridge_agent.ini文件（控制节点 + 计算节点）

```
# vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini

physical_interface_mappings = provider:enp2s0f0, admodel:enp2s0f1
```

3. 在web页面上创建外部网络即可


