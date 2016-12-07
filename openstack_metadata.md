# OpenStack 的 metadata 服务机制

http://www.ibm.com/developerworks/cn/cloud/library/1509_liukg_openstackmeta/

在创建虚拟机的时候，用户往往需要对虚拟机进行一些配置，比如：开启一些服务、安装某些包、添加 SSH 秘钥、配置 hostname 等等。在 OpenStack 中，这些配置信息被分成两类：metadata 和 user data。Metadata 主要包括虚拟机自身的一些常用属性，如 hostname、网络配置信息、SSH 登陆秘钥等，主要的形式为键值对。而 user data 主要包括一些命令、脚本等。User data 通过文件传递，并支持多种文件格式，包括 gzip 压缩文件、shell 脚本、cloud-init 配置文件等。虽然 metadata 和 user data 并不相同，但是 OpenStack 向虚拟机提供这两种信息的机制是一致的，只是虚拟机在获取到信息后，对两者的处理方式不同罢了。

在 OpenStack 中，虚拟机获取 Metadata 信息的方式有两种：Config drive 和 metadata RESTful 服务。

Config drive 机制是指 OpenStack 将 metadata 信息写入虚拟机的一个特殊的配置设备中，然后在虚拟机启动时，自动挂载并读取 metadata 信息，从而达到获取 metadata 的目的。

OpenStack 提供了 RESTful 接口，虚拟机可以通过 REST API 来获取 metadata 信息。提供该服务的组件为：nova-api-metadata。当然，要完成从虚拟机至网络节点的请求发送和相应，只有 nova-api-metadata 服务是不够的，此外共同完成这项任务的服务还有：Neutron-metadata-agent 和 Neutron-ns-metadata-proxy。


虚拟机获取 metadata 的大致流程为：首先请求被发送至 neutron-ns-metadata-proxy，此时会在请求中添加 router-id 和 network-id，然后请求通过 unix domian socket 被转发给 neutron-metadata-agent，根据请求中的 router-id、network-id 和 IP，获取 port 信息，从而拿到 instance-id 和 tenant-id 加入请求中，最后请求被转发给 nova-api-metadata，其利用 instance-id 和 tenant-id 获取虚拟机的 metadata，返回相应。