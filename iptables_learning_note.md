### iptables

首先介绍iptables的结构：iptables -> Tables -> Chains -> Rules. 简单地讲，tables由chains组成，而chains又由rules组成。

#### iptables的表与链

iptables具有Filter, NAT, Mangle, Raw四种内建表:

1. Filter表
Filter表示iptables的默认表，因此如果你没有自定义表，那么就默认使用filter表，它具有以下三种内建链：
INPUT链 – 处理来自外部的数据。
OUTPUT链 – 处理向外发送的数据。
FORWARD链 – 将数据转发到本机的其他网卡设备上。

2. NAT表
NAT表有三种内建链：
PREROUTING链 – 处理刚到达本机并在路由转发前的数据包。它会转换数据包中的目标IP地址（destination ip address），通常用于DNAT(destination NAT)。
POSTROUTING链 – 处理即将离开本机的数据包。它会转换数据包中的源IP地址（source ip address），通常用于SNAT（source NAT）。
OUTPUT链 – 处理本机产生的数据包。

3. Mangle表
Mangle表用于指定如何处理数据包。它能改变TCP头中的QoS位。Mangle表具有5个内建链：
PREROUTING
OUTPUT
FORWARD
INPUT
POSTROUTING

4. Raw表
Raw表用于处理异常，它具有2个内建链：
PREROUTING chain
OUTPUT chain


#### IPTABLES 规则(Rules)

牢记以下三点式理解iptables规则的关键：
Rules包括一个条件和一个目标(target)
如果满足条件，就执行目标(target)中的规则或者特定值。
如果不满足条件，就判断下一条Rules。

**目标值（Target Values）**
下面是你可以在target里指定的特殊值：
ACCEPT – 允许防火墙接收数据包
DROP – 防火墙丢弃包
QUEUE – 防火墙将数据包移交到用户空间
RETURN – 防火墙停止执行当前链中的后续Rules，并返回到调用链(the calling chain)中。

输出包含下列字段：
num – 指定链中的规则编号
target – 前面提到的target的特殊值
prot – 协议：tcp, udp, icmp等
source – 数据包的源IP地址
destination – 数据包的目标IP地址

#### 清空所有iptables规则
在配置iptables之前，你通常需要用iptables --list命令或者iptables-save命令查看有无现存规则，因为有时需要删除现有的iptables规则：
> iptables --flush

或者
> iptables -F

#### 永久生效
+ Ubuntu

> iptables-save > /etc/iptables.rules
> iptables-restore < /etc/iptables.rules

+ CentOS
保存iptables规则

> service iptables save

重启iptables服务
> service iptables stop
> service iptables start
> cat  /etc/sysconfig/iptables

#### 追加iptables规则
可以使用iptables -A命令追加新规则，其中-A表示Append。因此，新的规则将追加到链尾。
一般而言，最后一条规则用于丢弃(DROP)所有数据包。如果你已经有这样的规则了，并且使用-A参数添加新规则，那么就是无用功。
iptables -A chain firewall-rule
-A chain – 指定要追加规则的链
firewall-rule – 具体的规则参数

**规则的基本参数**
以下这些规则参数用于描述数据包的协议、源地址、目的地址、允许经过的网络接口，以及如何处理这些数据包。

+ -p 协议（protocol）
指定规则的协议，如tcp, udp, icmp等，可以使用all来指定所有协议。
如果不指定-p参数，则默认是all值。这并不明智，请总是明确指定协议名称。
可以使用协议名(如tcp)，或者是协议值（比如6代表tcp）来指定协议。映射关系请查看/etc/protocols
还可以使用–protocol参数代替-p参数

+ -s 源地址（source）
指定数据包的源地址
参数可以使IP地址、网络地址、主机名
例如：-s 192.168.1.101指定IP地址
例如：-s 192.168.1.10/24指定网络地址
如果不指定-s参数，就代表所有地址
还可以使用–src或者–source

+ -d 目的地址（destination）
指定目的地址
参数和-s相同
还可以使用–dst或者–destination

+ -j 执行目标（jump to target）
-j代表”jump to target”
-j指定了当与规则(Rule)匹配时如何处理数据包
可能的值是ACCEPT, DROP, QUEUE, RETURN
还可以指定其他链（Chain）作为目标

+ -i 输入接口（input interface）
-i代表输入接口(input interface)
-i指定了要处理来自哪个接口的数据包
这些数据包即将进入INPUT, FORWARD, PREROUTE链
例如：-i eth0指定了要处理经由eth0进入的数据包
如果不指定-i参数，那么将处理进入所有接口的数据包
如果出现! -i eth0，那么将处理所有经由eth0以外的接口进入的数据包
如果出现-i eth+，那么将处理所有经由eth开头的接口进入的数据包
还可以使用–in-interface参数

+ -o 输出（out interface）
-o代表”output interface”
-o指定了数据包由哪个接口输出
这些数据包即将进入FORWARD, OUTPUT, POSTROUTING链
如果不指定-o选项，那么系统上的所有接口都可以作为输出接口
如果出现! -o eth0，那么将从eth0以外的接口输出
如果出现-i eth+，那么将仅从eth开头的接口输出
还可以使用–out-interface参数


#### 状态 (state)匹配

> iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

“-m”是“匹配”的意思，-m state的意思是匹配数据包状态，用户发来的数据包分别带有不同的状态，即 NEW, ESTABLISHED, 和 RELATED。NEW 就是开头搭讪，ESTABLISHED，就是搭讪完了之后后续的数据包，RELATED就是与已经存在的连接相关的数据包。总之这句话的意思是，接受已经建立了连接的数据包，即搭讪之后的数据包。
为毛要允许状态ESTABLISHED 和 RELATED的入站数据呢？因为你的服务器同时也是台电脑，还要从别的服务器下载东西。下载时，你的服务器先向别的服务器发出连接请求(new)，别的服务器允许你连接，连接建立(ESTABLISHED)之后，就需要接受别的服务器发来的数据，对于你的服务器来讲属于INPUT。也就是说，如果没有iptables -A INPUT -m state –state ESTABLISHED,RELATED -j ACCEPT这句，wget curl啥的就都不工作了。
