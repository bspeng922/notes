##Ceilometer

###Architecture
![ceilometer架构图](images/ceilometer-core.png)
Ceilometer使用了两种收集数据的方式，一种是消费OpenStack各个服务内发出的notification消息(对应上图中的蓝色箭头)，一种 是通过调用各个服务的API去主动的获取数据(对应上图中的黑色箭头)。

Ceilometer由5个重要的组件以及一个Message Bus组成
(1) Compute Agent
该组件用来收集计算节点上的信息，在每一个计算节点上都要运行一个Compute Agent，该Agent通过Stevedore管理了一组pollster插件， 分别用来获取虚拟机的CPU, Disk IO, Network IO, Instance这些信息，值得一提的是这些信息大部分是通过调用Hypervisor的API来获取的， 目前，Ceilometer仅提供了Libvirt的API。
(2) Central Agent
Central Agent运行在控制节点上，它主要收集其它服务(Image, Volume, Objects, Network)的信息，实现逻辑和Compute Agent类似，但是 是通过调用这些服务的REST API去获取这些数据的。
(3) Collector
这个应该是最为核心的组件了，它的主要作用是监听Message Bus，将收到的消息以及相应的数据写入到数据库中，它是在核心架构中唯一一个 能够对数据库进行写操作的组件。除此之外，它的另一个作用是对收到的其它服务发来的notification消息做本地化处理，然后再重新发送到 Message Bus中去，随后再被其收集。
目前，通过上面三个组件可以收集到的数据，参看这里。
(4)Storage
数据存储现在支持MongoDB, MySQL, Postgresql和HBase，现在H3又新增加了对DB2的支持，其中MongoDB是支持最好的。
(5)REST API
像其它服务一样，Ceilometer也提供的是REST API，API是唯一一个能对数据库进行读操作的组件，虽然后来加入的Alarm API能够直接对数据库 进行读写，但是Alarm应该是一个较为独立的功能，并且它的读写量也不大
(6) Message Bus
Message Bus是整个数据流的瓶颈，所有的数据都要经过Message Bus被Collector收集而存到数据库中，目前Message Bus采用RabbitMQ实现。
(7) Pipeline
Pipeline虽然不是其中一个组件，但是也是一个重要的机制，它是Agent和Message Bus以及外界之间的桥梁，Agent将收集来的数据发送到pipeline中， 在pipeline中，先经过一组transformer的处理，然后通过Multi Publisher发送出去，可以通过Message Bus发送到Collector，或者是发送到其它的 地方。Pipeline是可配置的，Agent poll数据的时间间隔，以及Transformers和Publishers都是通pipeline.yaml文件进行配置。
整体上看，Ceilometer对数据流的处理，给人一种大禹治水的感觉。

###Plugin
Ceilometer中有四种类型的Plugin：Poller，Publisher，Notification和Transformer。
*Poller主要负责被Agent调用去查询数据，返回Counter类型的结果给Agent框架；
*Notification负责在MQ中监听相关topic的消息（虚拟机创建等），并把他转换成Counter类型的结果给Agent框架。
*Transformer负责转换Counter（目前在代码中还没有发现具体用li）
*Publisher负责将Agent框架中Counter类型的结果转换成消息（包括签名），并将消息发送到MQ；


**meter**
计量值，被跟踪资源的测量值。一个实例有很多的计量值，比如实例运行时长、CPU使用时间、请求磁盘的数量等。在Ceilometer中有三种计量值：
*Cumulative: 累计的，随着时间增长(如磁盘读写)
*Gauge: 计量单位，离散的项目(如浮动IP，镜像上传)和波动的值(如对象存储数值)
*Delta: 增量，随着时间的改变而增加的值(如带宽变化)
