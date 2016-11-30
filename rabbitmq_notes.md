# RabbitMQ

RabbitMQ is a message broker. The principal idea is pretty simple: it accepts and forwards messages. 

Producing means nothing more than sending. A program that sends messages is a producer. 

A queue is the name for a mailbox. It lives inside RabbitMQ. Although messages flow through RabbitMQ and your applications, they can be stored only inside a queue. A queue is not bound by any limits, it can store as many messages as you like ‒ it's essentially an infinite buffer. Many producers can send messages that go to one queue, many consumers can try to receive data from one queue.

Consuming has a similar meaning to receiving. A consumer is a program that mostly waits to receive messages.



AMQP当中有四个概念非常重要：虚拟主机（virtual host），交换机（exchange），队列（queue）和绑定（binding）。一个虚拟主机持有一组交换机、队列和绑定。


队列（Queues）是你的消息（messages）的终点，可以理解成装消息的容器。消息就一直在里面，直到有客户端（也就是消费者，Consumer）连接到这个队列并且将其取走为止。
需要记住的是，队列是由消费者（Consumer）通过程序建立的，不是通过配置文件或者命令行工具。

如果要把消息放入队列，就需要一个交换机（Exchange），交换机可以理解成具有路由表的路由程序,每个消息都有一个称为路由键（routing key）的属性，就是一个简单的字符串。交换机当中有一系列的绑定（binding），即路由规则（routes）