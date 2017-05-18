
corosync是用于高可用环境中的提供通讯服务的，它位于高可用集群架构中的底层（Message Layer），扮演着为各节点（node）之间提供心跳信息传递这样的一个角色；
pacemaker是一个开源的高可用资源管理器(CRM)，位于HA集群架构中资源管理、资源代理(RA)这个层次，它不能提供底层心跳信息传递的功能，它要想与对方节点通信需要借助底层的心跳传递服务，将信息通告给对方。通常它与corosync的结合方式有两种：
pacemaker作为corosync的插件运行；
pacemaker作为独立的守护进程运行；


/etc/hosts

All hosts must know about each other by the node name, so wind has:

192.168.1.109 fire
and fire has:

192.168.1.108 wind
kernel tweaking


Add this to your /etc/sysctl.conf:

# needed to bind to VIP
net.ipv4.ip_nonlocal_bind=1
Load this with sysctl -p and make it persistent by running it at boot time, such as from /etc/rc.local


## Install

```
# all
yum install -y pacemaker pcs corosync fence-agents resource-agents libqb0

systemctl enable pcsd.service
systemctl start pcsd.service

echo hapass | passwd --stdin hacluster

# one
pcs cluster auth controller1 controller2 controller3 -u hacluster -p hapass --force
pcs cluster setup --force --name openstack-cluster controller1 controller2 controller3
pcs cluster start --all

```

After installing the Corosync package, you must create the /etc/corosync/corosync.conf configuration file.
Corosync can be configured to work with either multicast or unicast IP addresses or to use the votequorum library.For environments that do not support multicast, Corosync should be configured for unicast.

```
# ``corosync.conf``

totem {
        #...
        interface {
                ringnumber: 0
                bindnetaddr: 10.0.0.0
                broadcast: yes (1)
                mcastport: 5405
        }
        interface {
                ringnumber: 1
                bindnetaddr: 10.0.42.0
                broadcast: yes
                mcastport: 5405
        }
        transport: udpu (2)
}

nodelist { (3)
        node {
                ring0_addr: 10.0.0.12
                ring1_addr: 10.0.42.12
                nodeid: 1
        }
        node {
                ring0_addr: 10.0.0.13
                ring1_addr: 10.0.42.13
                nodeid: 2
        }
        node {
                ring0_addr: 10.0.0.14
                ring1_addr: 10.0.42.14
                nodeid: 3
        }
}
```

To force Pacemaker to start after Corosync and stop before Corosync

```

pcs property set pe-warn-series-max=1000 \
  pe-input-series-max=1000 \
  pe-error-series-max=1000 \
  cluster-recheck-interval=5min

pcs property set stonith-enabled=false

pcs property set no-quorum-policy=ignore
pcs resource defaults migration-threshold=1

pcs resource create vip ocf:heartbeat:IPaddr2 params ip="10.0.0.11" cidr_netmask="24" op monitor interval="30s"

pcs resource create vip ocf:heartbeat:IPaddr2 params ip="192.168.1.111" nic="eth0" cidr_netmask="24" op monitor interval="30s"
pcs resource create api_vip ocf:heartbeat:IPaddr2 params ip="192.168.1.3" nic="eth0" cidr_netmask="24" op monitor interval="30s"

pcs resource create openstack-keystone systemd:openstack-keystone --clone interleave=true

```
## install
yum install -y gcc 

yum install -y haproxy

wget http://www.haproxy.org/download/1.7/src/haproxy-1.7.5.tar.gz
tar xvf haproxy-1.7.5.tar.gz 
cd haproxy-1.7.5
make TARGET=linux2628 ARCH=x86_64 PREFIX=/usr/local/haproxy
make install PREFIX=/usr/local/haproxy

#参数说明
TARGET=linux26 #内核版本，使用uname -r查看内核，如：2.6.18-371.el5，此时该参数就为linux26；kernel 大于2.6.28的用：TARGET=linux2628

/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg 

http://192.168.1.22:1080/stats


## Configuring HAProxy

mkdir -p /etc/haproxy
vim /etc/haproxy/haproxy.cfg
vim /usr/local/haproxy/haproxy.cfg 

```
global
  chroot  /var/lib/haproxy
  daemon
  group  haproxy
  maxconn  4000
  pidfile  /var/run/haproxy.pid
  user  haproxy

defaults
  log  global
  maxconn  4000
  option  redispatch
  retries  3
  timeout  http-request 10s
  timeout  queue 1m
  timeout  connect 10s
  timeout  client 1m
  timeout  server 1m
  timeout  check 10s

 listen dashboard_cluster
  bind <Virtual IP>:443
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 10.0.0.12:443 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:443 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:443 check inter 2000 rise 2 fall 5

 listen galera_cluster
  bind <Virtual IP>:3306
  balance  source
  option  mysql-check
  server controller1 10.0.0.12:3306 check port 9200 inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:3306 backup check port 9200 inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:3306 backup check port 9200 inter 2000 rise 2 fall 5

 listen glance_api_cluster
  bind <Virtual IP>:9292
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 10.0.0.12:9292 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:9292 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:9292 check inter 2000 rise 2 fall 5

 listen glance_registry_cluster
  bind <Virtual IP>:9191
  balance  source
  option  tcpka
  option  tcplog
  server controller1 10.0.0.12:9191 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:9191 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:9191 check inter 2000 rise 2 fall 5

 listen keystone_admin_cluster
  bind <Virtual IP>:35357
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 10.0.0.12:35357 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:35357 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:35357 check inter 2000 rise 2 fall 5

 listen keystone_public_internal_cluster
  bind <Virtual IP>:5000
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 10.0.0.12:5000 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:5000 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:5000 check inter 2000 rise 2 fall 5

 listen nova_ec2_api_cluster
  bind <Virtual IP>:8773
  balance  source
  option  tcpka
  option  tcplog
  server controller1 10.0.0.12:8773 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:8773 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:8773 check inter 2000 rise 2 fall 5

 listen nova_compute_api_cluster
  bind <Virtual IP>:8774
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 10.0.0.12:8774 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:8774 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:8774 check inter 2000 rise 2 fall 5

 listen nova_metadata_api_cluster
  bind <Virtual IP>:8775
  balance  source
  option  tcpka
  option  tcplog
  server controller1 10.0.0.12:8775 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:8775 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:8775 check inter 2000 rise 2 fall 5

 listen cinder_api_cluster
  bind <Virtual IP>:8776
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 10.0.0.12:8776 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:8776 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:8776 check inter 2000 rise 2 fall 5

 listen ceilometer_api_cluster
  bind <Virtual IP>:8777
  balance  source
  option  tcpka
  option  tcplog
  server controller1 10.0.0.12:8777 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:8777 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:8777 check inter 2000 rise 2 fall 5

 listen nova_vncproxy_cluster
  bind <Virtual IP>:6080
  balance  source
  option  tcpka
  option  tcplog
  server controller1 10.0.0.12:6080 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:6080 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:6080 check inter 2000 rise 2 fall 5

 listen neutron_api_cluster
  bind <Virtual IP>:9696
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 10.0.0.12:9696 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:9696 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:9696 check inter 2000 rise 2 fall 5

 listen swift_proxy_cluster
  bind <Virtual IP>:8080
  balance  source
  option  tcplog
  option  tcpka
  server controller1 10.0.0.12:8080 check inter 2000 rise 2 fall 5
  server controller2 10.0.0.13:8080 check inter 2000 rise 2 fall 5
  server controller3 10.0.0.14:8080 check inter 2000 rise 2 fall 5
```

## config kernel

Configure the kernel parameter to allow non-local IP binding. This allows running HAProxy instances to bind to a VIP for failover. 

```
vim /etc/sysctl.conf

net.ipv4.ip_nonlocal_bind=1
sysctl -p

```
Add HAProxy to the cluster and ensure the VIPs can only run on machines where HAProxy is active:

```
$ pcs resource create lb-haproxy systemd:haproxy --clone
$ pcs constraint order start vip then lb-haproxy-clone kind=Optional
$ pcs constraint colocation add lb-haproxy-clone with vip

```


## HaProxy


```
global
log 127.0.0.1 local2
chroot /var/lib/haproxy
pidfile /var/run/haproxy.pid
maxconn 16384
user haproxy
group haproxy
daemon
# turn on stats unix socket
stats socket /var/run/haproxy.cmd

defaults
mode http
log global
option httplog
option dontlognull
option httpclose
option forwardfor except 127.0.0.0/8
option redispatch
retries 3
timeout http-request 10s
timeout queue 1m
timeout connect 10s
timeout client 45s
timeout server 45s
timeout check 10s
maxconn 16384

listen stats :9000
mode http
stats enable
stats uri /haproxy
stats realm HAProxy\ Statistics
stats auth haproxy:password
stats admin if TRUE

listen http :8080
balance leastconn
option http-server-close
option forwardfor
server node1 192.168.1.3:80 check inter 3000 rise 2 fall 3
server node2 192.168.1.4:80 check inter 3000 rise 2 fall 3

```

```
net.ipv4.ip_nonlocal_bind = 1 意思是启动haproxy的时候，允许忽视VIP的存在

vi /etc/sysctl.conf        #修改内核参数
net.ipv4.ip_nonlocal_bind = 1  #没有就新增此条记录

sysctl -p        #保存结果，使结果生效

net.ipv4.ip_forward = 1
```
