# 缓存选 redis 还是 memcache

redis 和 memcache 都是缓存。什么时候选 redis？什么时候选 memcache呢？

以下场景选择 redis:

+ 复杂的数据结构。因为 memcache 无法满足要求。
+ 持久化。memcahe 无持久，无法满足要求。
+ 需要支持高可用。 redis 天然支持高可用，memcache 要支持高可用，需要进行二次开发。
+ 存储内容比较大。memcache 的 KV 存储最大为 1M。存储的值比较大，只能使用 redis.

什么场景选择 memcache？纯 KV，数据量，并发量大时。原因主要有以下几点（最主要是底层实现差异导致）：

+ 内存分配。memcache 使用预分配内存池管理方式，能够省去内存的分配时间，且没有碎片。redis 临时申请空间，有碎片。
+ 虚拟内存的使用。 memcache 所有数据存在物理内存。redis 有自己的 VM 机制，当数据超过物理内存时，会引发 swap 把冷数据刷到磁盘上。
+ 网络模型。memcache 使用非阻塞的 IO 复用模型。redis 也是相同。redis 还提供了非 KV 外的数据，在复杂的 CPU 计算时，有可能阻塞整个 IO 调度。
+ 线程模型。memcache 使用多线程而 redis 是单线程。
