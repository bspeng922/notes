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
