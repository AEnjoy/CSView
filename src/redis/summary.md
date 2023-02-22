### Redis为什么快？ <Badge text="掌握" type="tip" /> 

**基于内存操作**：Redis的绝大部分操作在内存里就可以实现，数据也存在内存中，与传统的磁盘文件操作相比减少了IO，提高了操作的速度。

**高效的数据结构**：Redis有专门设计了STRING、LIST、HASH等高效的数据结构，依赖各种数据结构提升了读写的效率。

**采用单线程**：单线程操作省去了上下文切换带来的开销和CPU的消耗，同时不存在资源竞争，避免了死锁现象的发生。

**I/O多路复用**：采用I/O多路复用机制同时监听多个Socket，根据Socket上的事件来选择对应的事件处理器进行处理。



### 为什么Redis是单线程？<Badge text="掌握" type="tip" /> 

单线程指的是：网络请求模块使用单线程进行处理，其他模块仍用多个线程。

官方答案是：因为CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了。

::: tip redis采用多线程的模块和原因

Redis 在启动的时候，会启动后台线程(BIO)：

- Redis的早期版本会启动2个后台线程，来处理关闭文件、AOF 刷盘这两个任务；
- Redis的后续版本，新增了一个新的后台线程，用来异步释放 Redis 内存。执行 unlink key / flushdb async / flushall async 等命令，会把这些删除操作交给后台线程来执行。

之所以 Redis 为关闭文件、AOF 刷盘、释放内存这些任务创建单独的线程来处理，是因为这些任务的操作都很耗时，把这些任务都放在主线程来处理会导致主线程阻塞，导致无法处理后续的请求。后台线程相当于一个消费者，生产者把耗时任务丢到任务队列中，消费者不停轮询这个队列，拿出任务去执行对应的方法即可。

:::



### Redis6.0为什么要引入多线程？<Badge text="掌握" type="tip" />

因为Redis的瓶颈不在内存，而是在网络I/O模块带来CPU的耗时，所以Redis6.0的多线程是用来处理网络I/O这部分，充分利用CPU资源，减少网络I/O阻塞带来的性能损耗。Redis引入的多线程 I/O 特性对性能提升至少是一倍以上。



### 为什么用Redis作为MySQL的缓存？<Badge text="掌握" type="tip" />

MySQL是数据库系统，对于数据的操作需要访问磁盘，而将数据放在Redis中，需要访问就可以直接从内存获取，避免磁盘I/O，提高操作的速度。

使用Redis+MySQL结合的方式可以有效提高系统QPS。



### Redis和Memcached的联系和区别？<Badge text="了解" type="info" />

**共同点**：

- 都是内存数据库
- 性能都非常高
- 都有过期策略

**区别**：

- **线程模型**：Memcached采用多线程模型，并且基于I/O多路复用技术，主线程接收到请求后分发给子线程处理，这样做好的好处是：当某个请求处理比较耗时，不会影响到其他请求的处理。缺点是CPU的多线程切换存在性能损耗，同时，多线程在访问共享资源时要加锁，也会在一定程度上降低性能；Redis也采用I/O多路复用技术，但它处理请求采用是单线程模型，从接收请求到处理数据都在一个线程中完成。这意味着使用Redis一旦某个请求处理耗时比较长，那么整个Redis就会阻塞住，直到这个请求处理完成后返回，才能处理下一个请求，使用Redis时一定要避免复杂的耗时操作，单线程的好处是，少了CPU的上下文切换损耗，没有了多线程访问资源的锁竞争，但缺点是无法利用CPU多核的性能。

- **数据结构**：Memcached支持的数据结构很单一，仅支持string类型的操作。并且对于value的大小限制必须在1MB以下，过期时间不能超过30天；而Redis支持的数据结构非常丰富，除了常用的数据类型string、list、hash、set、zset之外，还可以使用geo、hyperLogLog数据类型；使用Memcached时，我们只能把数据序列化后写入到Memcached中。然后再从Memcached中读取数据，再反序列化为我们需要的格式，只能“整存整取”；Redis提供的数据结构提升了操作的便利性。

- **淘汰策略**：Memcached必须设置整个实例的内存上限，数据达到上限后触发LRU淘汰机制，优先淘汰不常用使用的数据。它的数据淘汰机制存在一些问题：刚写入的数据可能会被优先淘汰掉，这个问题主要是它本身内存管理设计机制导致的；Redis没有限制必须设置内存上限，如果内存足够使用，Redis可以使用足够大的内存，同时Redis提供了多种内存淘汰策略。

- **持久化**：Memcached不支持数据的持久化，如果Memcached服务宕机，那么这个节点的数据将全部丢失。Redis支持AOF和RDB两种持久化方式。

- **集群**：Memcached没有主从复制架构，只能单节点部署，如果节点宕机，那么该节点数据全部丢失，业务需要对这种情况做兼容处理，当某个节点不可用时，把数据写入到其他节点以降低对业务的影响；Redis拥有主从复制架构，主节点从可以实时同步主的数据，提高整个Redis服务的可用性。

  



### 如何理解Redis原子性操作原理？<Badge text="掌握" type="tip" />

**API**：Redis提供的API都是单线程串行处理的。

**网络模型**：采用单线程的epoll的网络模型，用来处理多个Socket请求。

**请求处理**：Redis会fork子进程来出来类似于RDB和AOF的操作，不影响主进程工作。