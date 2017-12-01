## 网络节点:

1. Add the QoS service to the service_plugins setting in /etc/neutron/neutron.conf. For example:

```
service_plugins = ... ,qos
```


2. In /etc/neutron/plugins/ml2/ml2_conf.ini, add qos to extension_drivers in the [ml2] section. For example:

```
[ml2]
extension_drivers = port_security, qos
```


3. If the Open vSwitch agent/Linux bridge is being used, set extensions to qos in the [agent] section of /etc/neutron/plugins/ml2/[openvswitch_agent|linuxbridge_agent].ini. For example:

```
[agent]
extensions = qos
```


## 计算节点:

In /etc/neutron/plugins/ml2/[openvswitch_agent|linuxbridge_agent].ini, add qos to the extensions setting in the [agent] section. For example:

```
[agent]
extensions = qos
```


## 常用命令

```
# 创建qos策略
[root@controller ~]# neutron qos-policy-create bw-limiter

# 列出qos策略
[root@controller ~]# neutron qos-policy-list
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
+--------------------------------------+------------+----------------------------------+
| id                                   | name       | tenant_id                        |
+--------------------------------------+------------+----------------------------------+
| d3499b26-806d-43dd-80f1-68f0787af486 | bw-limiter | 08b173e0db2049e59d19bdd0e45f6b36 |
+--------------------------------------+------------+----------------------------------+

# 创建带宽限制规则
[root@controller ~]# neutron qos-bandwidth-limit-rule-create bw-limiter --max-kbps 3 --max-burst-kbps 3

# 列出当前qos策略下面的规则
[root@controller ~]# neutron qos-bandwidth-limit-rule-list bw-limiter
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
+-----------+--------------------------------------+----------------+----------+
| direction | id                                   | max_burst_kbps | max_kbps |
+-----------+--------------------------------------+----------------+----------+
| egress    | 0379b40f-4b55-49d7-9e32-77a9c3508424 |              3 |        3 |
+-----------+--------------------------------------+----------------+----------+

# 更新带宽限制规则
[root@controller ~]# neutron qos-bandwidth-limit-rule-update  0379b40f-4b55-49d7-9e32-77a9c3508424 bw-limiter --max-kbps 200 --max-burst-kbps 200

# 为端口设置qos策略
[root@controller ~]# neutron port-update b90df288-6239-4a07-ae66-9e67bd4c44b6 --qos-policy bw-limiter

# 为整个网络设置qos策略
[root@controller ~]# neutron net-update private --qos-policy bw-limiter

# 查看当前端口详情
[root@controller ~]# neutron port-show b90df288-6239-4a07-ae66-9e67bd4c44b6
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
+-----------------------+------------------------------------------------------------------------------------+
| Field                 | Value                                                                              |
+-----------------------+------------------------------------------------------------------------------------+
| admin_state_up        | True                                                                               |
| allowed_address_pairs |                                                                                    |
| binding:host_id       | compute1                                                                           |
| binding:profile       | {}                                                                                 |
| binding:vif_details   | {"port_filter": true}                                                              |
| binding:vif_type      | bridge                                                                             |
| binding:vnic_type     | normal                                                                             |
| created_at            | 2017-11-08T02:31:47Z                                                               |
| description           |                                                                                    |
| device_id             | 880c6c8e-2348-46cf-ba47-5e1916d5843b                                               |
| device_owner          | compute:nova                                                                       |
| extra_dhcp_opts       |                                                                                    |
| fixed_ips             | {"subnet_id": "d3e38396-6227-4514-b56b-539ff61888f7", "ip_address": "192.168.1.8"} |
| id                    | b90df288-6239-4a07-ae66-9e67bd4c44b6                                               |
| mac_address           | fa:16:3e:2a:df:5b                                                                  |
| name                  |                                                                                    |
| network_id            | c192f87f-2c6c-47aa-af9a-97cd9e58a958                                               |
| port_security_enabled | True                                                                               |
| project_id            | 08b173e0db2049e59d19bdd0e45f6b36                                                   |
| qos_policy_id         | d3499b26-806d-43dd-80f1-68f0787af486                                               |
| revision_number       | 369649                                                                             |
| security_groups       | 59d7794d-ffc8-4c0a-8bb7-34624bc14821                                               |
| status                | ACTIVE                                                                             |
| tags                  |                                                                                    |
| tenant_id             | 08b173e0db2049e59d19bdd0e45f6b36                                                   |
| updated_at            | 2017-11-30T11:33:15Z                                                               |
+-----------------------+------------------------------------------------------------------------------------+

# 删除端口上面的qos策略
[root@controller ~]# neutron port-update b90df288-6239-4a07-ae66-9e67bd4c44b6 --no-qos-policy

```


## 检查计算节点

```
# 检查tc规则，最后一条为新增的qos规则
[root@compute1 ~]# tc qdisc show
qdisc noqueue 0: dev lo root refcnt 2

... ...

qdisc noqueue 0: dev docker0 root refcnt 2
qdisc noqueue 0: dev brqdf94a80d-62 root refcnt 2
qdisc noqueue 0: dev vxlan-7 root refcnt 2
qdisc noqueue 0: dev tap50fe349c-88 root refcnt 2
qdisc noqueue 0: dev tapb5f2adfd-cc root refcnt 2
qdisc noqueue 0: dev tapeb6cafe1-c1 root refcnt 2
qdisc noqueue 0: dev brqc192f87f-2c root refcnt 2
qdisc noqueue 0: dev vxlan-10 root refcnt 2
qdisc pfifo_fast 0: dev tapb90df288-62 root refcnt 2 bands 3 priomap  1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc ingress ffff: dev tapb90df288-62 parent ffff:fff1 ----------------
```

