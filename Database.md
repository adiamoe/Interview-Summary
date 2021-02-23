# 数据库

#### 索引

##### 聚簇索引

页表数据顺序和索引顺序相同，非聚簇索引顺序与表数据顺序无关。一张表中最多有一个聚簇索引

聚簇索引可以是稀疏索引，非聚簇索引必须是稠密索引，否则没有意义。

可以通过bucket减少稀疏索引的数量，对相同的value用同一bucket的指针指向，还可以通过求交集减少I/O次数

#### 查找树

用B树的优点，降低树的深度，减少读取磁盘的次数，提升查找效率

##### B树

关键字集合分布在整棵树中，搜索可能在非叶子结点结束

##### B+树

非叶子结点仅用于索引，所有数据都保存在叶子节点。叶子节点之间存在链接指针，便于区间查找和遍历

##### B+树比B树更适合作为文件索引

优点：

1. B+树磁盘读写代价更低

    B+tree的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对B树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了。

2. B+tree的查询效率更加稳定

3. B树在提高了磁盘IO性能的同时并没有解决元素遍历的效率低下的问题。

缺点：

B树的每一个节点都包含key和value，因此经常访问的元素可能离根节点更近，因此访问也更迅速。

##### LSM树

Log-Structured MergeTree 和文件系统中的LSF类似：

copy on write（写时复制），对过去的数据不进行修改

假定内存足够大，可以将最新的数据驻留在内存中，等到积累到足够多之后再使用归并排序将内存中的数据追加到磁盘。每次写入都会有一个小的结构（SSTable），在查询的时候需要从最新的SSTable中依次查询数据（因为不知道在哪个中）

牺牲了部分读的性能，换取大幅提高写性能。

可以用布隆过滤器和合并方式优化性能。

##### 跳表

多层索引链表，查找和插入时间复杂度为O(logn)

#### 事务

ACID: Atomicity, Consistency, isolation, durability

Serializable, repeatable read, read committed, read uncommitted

#### 锁

##### 乐观锁和悲观锁

悲观锁假设最坏情况，认为其他线程或事务会修改数据，因此每次获取数据时都会加锁，阻塞其他线程。如Java中的`synchronized`。

乐观锁则认为其他事务不会修改数据，但是为了避免发生这种情况，在更新前会检查该数据是否被修改。适用于多读的场景。实现方式有版本号和CAS(compare and swap)

乐观锁对于悲观锁的优势在于，减少了获取锁和判断锁的开销，加大了系统的吞吐量。但存在ABA问题，即无法辨别修改后又被改回的问题。同时如果写的数量太多，CAS自旋开销大或者会导致事务多次rollback

##### 两阶段锁(2PL)(悲观锁)

第一阶段：Growing

所有事务尝试获取所需的所有锁，lock manager授予或者拒绝

第二阶段：Shrinking

仅能释放之前获取的锁，不允许获取新锁

可以保证冲突可串行化，但可能会引发级联回滚(cascading abort)、脏读或**死锁**

解决措施：**strong 2PL**

在事务结束时同时释放所有锁，不提前释放。

##### 时间戳顺序(T/O)(乐观锁)

Check timestamps for every operation:
→ If txn tries to access an object "from the future", it aborts and restarts.

##### OPTIMISTIC CONCURRENCY CONTROL(OCC)

1. Read Phase: ensure repeatable read
2. Validation Phase
3. Write Phase : only one txn can be in the Write Phase at a time(Latch)

性能问题：

1. 将数据复制到本地时消耗性能
2. Validation/Write phase bottlenecks.
3. 在所有操作完成之后的验证阶段才会检查是否中止，浪费较大

##### 粒度和意向锁

减小锁的粒度能提高数据的并行性

增大锁的粒度能提高系统性能

意向锁定允许以共享或排他模式锁定更高级别的节点，而不必检查所有的派生节点。如果一个节点在意图模式下被锁定，那么某些txn在树的较低级别上执行显式锁定。(IX,IS,SIX)

##### 动态数据库

当发生插入删除时，可能会出现幻读：获取锁的事务可能只获取了已经存在的数据的锁，而没有考虑可能会添加的数据

解决方案：1. 重复扫描 2. 预测锁 3.索引锁

##### 死锁

条件：1. 非抢占 2. 持有并等待 3. 循环等待 4.互斥

破坏任意一个形成条件

Deadlock Detection 

维护等待图，间隔一定时间检查等待图中有无循环。但要考虑间隔时间，检查过于频繁会降低性能，检查太少会导致事务长时间等待。

Deadlock Prevention

当事务想要获取已经被占用的锁时，直接kill该事务

##### 日志

Steal: 是否允许未提交的事务数据写入磁盘

Force: 是否要求事务提交前将所做修改写入磁盘

NO-STEAL + FORCE 最容易实现，不需要undo和redo(Shadow Paging)

STEAL + NO FORCE: Write Ahead Log

在数据被写入磁盘前和事务提交前，先将日志写入磁盘

在CheckPoints时将缓冲区中的数据写入磁盘，当提交时需要暂停所有事务确保一致性

Fuzzy CheckPoints:





