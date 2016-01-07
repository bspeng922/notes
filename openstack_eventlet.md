## Eventlet

### epoll

epoll是linux实现的一个基于事件的异步IO库, 在之前类似的异步IO库poll上改进而来.


### greenlet

greentlet是python中实现我们所谓的"Coroutine(协程)"的一个基础库.
可以发现greenlet仅仅是实现了一个最简单的"coroutine", 而eventlet中的greenthread是在 greenlet的基础上封装了一些更high-level的功能, 比如greenlet的调度等.


### eventlet.green

从epoll的运行机制可以看出, 要使用异步IO, 必须要将相关IO操作改写成non-blocking的方式.但是我们用eventlet.spawn()的函数, 并没有针对epoll做任何改写, 那eventlet是怎么实现 异步IO的呢?

这也是eventlet这个package最凶残的地方, 它自己重写了python标准库中IO相关的操作, 将它们 改写成支持epoll的模式, 放在eventlet.green中.

