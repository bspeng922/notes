## Ceilometer L版 IPMI 代码分析

####setup.cfg
ceilometer.notification =
    ...
    hardware.ipmi.temperature = ceilometer.ipmi.notifications.ironic:TemperatureSensorNotification
    hardware.ipmi.voltage = ceilometer.ipmi.notifications.ironic:VoltageSensorNotification
    hardware.ipmi.current = ceilometer.ipmi.notifications.ironic:CurrentSensorNotification
    hardware.ipmi.fan = ceilometer.ipmi.notifications.ironic:FanSensorNotification

ceilometer.poll.ipmi =
    hardware.ipmi.node.power = ceilometer.ipmi.pollsters.node:PowerPollster
    hardware.ipmi.node.temperature = ceilometer.ipmi.pollsters.node:InletTemperaturePollster
    hardware.ipmi.node.outlet_temperature = ceilometer.ipmi.pollsters.node:OutletTemperaturePollster
    hardware.ipmi.node.airflow = ceilometer.ipmi.pollsters.node:AirflowPollster
    hardware.ipmi.node.cups = ceilometer.ipmi.pollsters.node:CUPSIndexPollster
    hardware.ipmi.node.cpu_util = ceilometer.ipmi.pollsters.node:CPUUtilPollster
    hardware.ipmi.node.mem_util = ceilometer.ipmi.pollsters.node:MemUtilPollster
    hardware.ipmi.node.io_util = ceilometer.ipmi.pollsters.node:IOUtilPollster
    hardware.ipmi.temperature = ceilometer.ipmi.pollsters.sensor:TemperatureSensorPollster
    hardware.ipmi.voltage = ceilometer.ipmi.pollsters.sensor:VoltageSensorPollster
    hardware.ipmi.current = ceilometer.ipmi.pollsters.sensor:CurrentSensorPollster
    hardware.ipmi.fan = ceilometer.ipmi.pollsters.sensor:FanSensorPollster


####pollster/node.py
通过nodemanager来读取相应的值 
get_samples
read_data


####pollster/sensor.py
温度、电流、电压、风扇传感器


####platform/ipmi_sensor.py
IPMI sensor to collect various sensor data of compute node
创建IPMISensor类（单例），检查是否支持ipmi，类方法执行(调用ipmitool.execute_ipmi_cmd方法)相应sdr命令


####platform/ipmitool.py
Utils to run ipmitool for data collection

execute_ipmi_cmd()
执行ipmi命令，并格式化返回输出信息


####platform/intel_node_manager.py
Node manager engine to collect power and temperature of compute node.
This file provides Node Manager engine to get simple system power and temperature data based on ipmitool.


####notifications/ironic.py
Converters for producing hardware sensor data sample messages from notification events


### IPMI简介
IPMI是智能型平台管理接口（Intelligent Platform Management Interface）的缩写，是管理基于 Intel结构的企业系统中所使用的外围设备采用的一种工业标准，该标准由英特尔、惠普、NEC、美国戴尔电脑和SuperMicro等公司制定。用户可以利用IPMI监视服务器的物理健康特征，如温度、电压、风扇工作状态、电源状态等。

a)   raw：发送一个原始的IPMI请求，并且打印回复信息。
b)   lan：配置网络（lan）信道(channel)
c)   chassis ：查看底盘的状态和配置电源
d)   event：向BMC发送一个已定义的事件（event），可用于测试配置的SNMP是否成功
e)   mc：  查看MC（Management Contollor）状态和各种允许的项
f)   sdr：打印传感器仓库中的任何监控项和从传感器读取到的值。
g)   sensor：打印周详的传感器信息。
h)   Fru：打印内建的Field Replaceable Unit (FRU)信息
i)   sel： 打印 System Event Log (SEL)      
j)   pef： 配置 Platform Event Filtering (PEF)，事件过滤平台用于在监控系统发现有event时候，用PEF中的策略进行事件过滤，然后看是否需要报警。
k)   sol/isol：用于配置通过串口的Lan进行监控
l)   user：配置BMC中用户的信息 。
m)  channel：配置Management Controller信道。

#### 配置：
IPMI需要配置一个局域网管理地址、用户名、密码，以上信息都可以通过bios进行配置。对于bios中不能创建用户的机器，可以先在bios中设置好IP地址，然后通过命令来设置用户&密码:

1. load impi drivers:
```shell
# modprobe ipmi_msghandler
# modprobe ipmi_devintf
# modprobe ipmi_si
```

2. install ipmitool: maybe use rpm package to install
```shell
# apt-get install ipmitool
```

3. to find the LAN channel
```shell
# ipmitool lan print 1
```

4. configure IP address setting:
dynamic:  ipmitool lan set 1 ipsrc dhcp
static :
```shell
# ipmitool lan set 1 ipaddr 192.168.111.111
# ipmitool lan set 1 netmask 255.255.0.0
# ipmitool lan set 1 defgw ipaddr 192.168.0.1
```

5. Establish an administration account with username and password.
Set BMC to require password authentication for ADMIN access over LAN. For example:
```shell
# ipmitool lan set 1 auth ADMIN MD5,PASSWORD
```
List the account slots on the BMC and identify an unused slot：
```shell
# ipmitool channel getaccess 1
```
Assign the administrator user name and password and enable messaging for the identified slot.
```shell
 # ipmitool user set name 4 myuser
 # ipmitool user set password 4 mypassword
 # ipmitool user enable 4
 # ipmitool channel setaccess 1 4 callin=off ipmi=on link=on privilege=4
```

6. Verify the setup after you complete configuration using the command ipmitool lan print.
```shell
# ipmitool lan print 1
# ipmitool channel getaccess 1 4
```

#### 在客户端的IPMI远程操作
在远程操作的机器上也需要安装impitool，然后通过一些命令来操作被控制的机器。
```shell
# ipmitool -H yourIP -U myuser -P mypassword power status  （查看服务器的电源状态）
# ipmitool -H yourIP -U myuser -P mypassword power reset   （关闭电源并重启机器）
# ipmitool -H yourIP -U myuser -P mypassword power on   （开电源启动机器）
# ipmitool -H yourIP -U myuser -P mypassword power off   （关闭电源）
# ipmitool -H 192.168.12.84 -I lanplus -U test -P 123456 sdr type Temperature （服务器温度）
```



#### Connecting to the Server With IPMItool

To connect over a remote interface you must supply a user name and password. The default user with admin-level access is root with password changeme. This means you must use the -U and -P parameters to pass both user name and password on the command line, as shown in the following example:
```shell
ipmitool -I lanplus -H <IPADDR> -U root -P changeme chassis status
```

> Note - If you encounter command-syntax problems with your particular operating system, you can use the ipmitool -h command and parameter to determine which parameters can be passed with the ipmitool command on your operating system. Also refer to the IPMItool man page by typing man ipmitool.

> Note - In the example commands shown in this appendix, the default user name, root, and default password, changeme are shown. Type the user name and password that has been set for the server.

**Enabling the Anonymous User**


> Note - Enabling anonymous user using IPMItool, is not supported in ILOM 3.0.

To enable the Anonymous/NULL user you must alter the privilege level on that account. This lets you connect without supplying a -U user option on the command line. The default password for this user is anonymous.
```shell
ipmitool -I lanplus -H <IPADDR> -U root -P changeme channel setaccess 1 1 privilege=4
ipmitool -I lanplus -H <IPADDR> -P anonymous user list
```

**Changing the Default Password**

You can also change the default passwords for a particular user ID. First get a list of users and find the ID for the user you wish to change, and then supply it with a new password, as shown in the following command sequence:
```shell
ipmitool -I lanplus -H <IPADDR> -U root -P changeme user list

ID  Name    Callin  Link Auth  IPMI Msg   Channel Priv Limit
1   false   false      true       NO ACCESS
2   root    false   false      true       ADMINISTRATOR

ipmitool -I lanplus -H <IPADDR> -U root -P changeme user set password 2 newpass

ipmitool -I lanplus -H <IPADDR> -U root -P newpass chassis status
```
where IPADDR is IP address of the server.
For example:
xxx.xx.xx.xxx.

