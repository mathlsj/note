# 数据库读性能提升

数据库工程设计，需要设计什么呢？需要设计库的表结构和索引的结构。

库表结构，索引结构的设计依据是什么？根据`业务模式`设计库表结构，根据`访问模式`设计索引结构。

除此之外，数据库工程结构还需要考虑哪些问题？

+ 读性能提升
+ 数据库高可用
+ 数据一致性
+ 扩展性
+ 垂直拆分

## 如何提升数据查询性能

常见方法：

+ 建立索引。建立索引潜在问题是会导致写性能降低，同时索引占用内存大时，buffer 命中率降低，读性能也会降低。
+ 为不同的数据库实例，建立不同的索引。存在问题是会带来运维的复杂。
+ 增加从库。增加从库解决的是读性能提升和读高可用的问题。增加从库会带来数据主从一致的问题。
+ 增加缓存。需要注意防止缓存挂掉，导致数据库雪崩。为了避免雪崩，可使用缓存高可用或缓存水平切分。增加缓存会带来数据一致性问题，业务流程也会变得更复杂。
