# 数据库

#### 范式

第一范式：符合1NF的关系中的每个属性都不可再分

第二范式：全部的非主键列依赖于全部主键。

第三范式：对于每个非平凡FD，或者其左边是超键，或者右边仅有主属性构成（主属性是指该属性是键的成员）（分解保证2、3）

BCNF范式：每个非平凡FD的左边都是超键（分解保证1、2）

第四范式：对于每个非平凡的MVD，左侧都是超键（分解保证1、2）

##### 分解性质

1. 消除异常
2. 信息的可恢复性（无损连接）
3. 依赖的保持

#### 索引

页表数据顺序和聚簇索引顺序相同，非聚簇索引顺序与表数据顺序无关。一张表中最多有一个聚簇索引。

聚簇索引可以是稀疏索引，非聚簇索引必须是稠密索引，否则没有意义。

可以通过bucket减少稀疏索引的数量，对相同的value用同一bucket的指针指向，还可以通过求交集减少I/O次数

##### 索引的缺点

索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大。 

当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。 

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

###### Mysql中InnoDB和MyISAM中的索引区别

都是通过B+树组织的

InnoDB是聚集索引，主键索引和数据储存在一起。普通索引通过查询对应的主键，再在主键索引中查询。主键索引不应该太大，因为普通索引也都要储存主键索引。

MyISAM的索引和数据分开存储，主键索引和普通索引没有太大区别，互相独立。

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

* 在CheckPoints时将缓冲区中的数据写入磁盘，当提交时需要停止接受新事物并等待执行中的事务全部结束，再将数据刷入磁盘。暂停事务会对效率和性能造成很大影响。
* 一个较好的方案是：当系统执行CheckPoints的时候暂停**写事务**(禁止获取数据锁)，读事务可以继续执行，不需要等待所有事务结束。会导致内存-磁盘数据不一致，因此需要记录正在执行的事务(ATT)，内存中的脏页(DPT)。但这依然不是一个理想的方案，因为暂停事务依然会影响性能。
* FUZZY CHECKPOINTS : 刷入磁盘和事务同时进行。在日志中记录Checkpoints-begin和Checkpoints-end(包括ATT, DPT)，当恢复时，从最新的一次checkpoints向下记录所有事务，并redo所有记录，将commit的事务刷入磁盘，其余的事务撤销。

如何提升DBMS在undo时的性能

1. 在内存中记录需要回滚的事务，但直到新的事务读写该部分时再修改(Lazy Rollback)
2. 避免使用长事务

#### Join及优化

* 为什么需要join: 将数据储存到关系数据库时，为了减少数据的冗余，节约内存，需要将数据拆分规范化。因此需要join来重建完整数据。

Join算法：

假设左表**M pages，m tuples**, 右表**N pages, n tuples**

1. Nested Loop Join

    1. Simple

        从左表中逐项取数据和右表进行比较，需要大量读取磁盘，效率极低。

        cost: M+(m*N)

    2. Block

        先预先从磁盘中读取部分元组存入内存，再和右表比较，如每次取一页。

        cost: M + (M*N)

    3. Index

        利用索引，可以避免每次比较都需要遍历右表，直接寻找到对应的元组，假设每次索引的代价是常数C

        cost: M + (m*C)

2. Sort-Merge Join

    将两表按照对应数据排序，然后将数据相等的连接

    Sort Cost (R): $2M(1 + ⌈ \log B-1 ⌈M / B⌉ ⌉)$
    Sort Cost (S): $2N(1 + ⌈ \log B-1 ⌈N / B⌉ ⌉)$
    Merge Cost: $(M + N)$
    Total Cost: Sort + Merge

    最差的情况是两个表中的所有元组对应数据都相等。

    在实际数据库中，元组按主键都是排好序的，因此可以省去排序的过程

3. Hash Join

    相同的值通过hash一定能映射到同样的区域。因此通过hash，只需要比较该区域的值即可。

    * 阶段一：Build

        用hash函数h1扫描右表，建立哈希映射表

    * 阶段二：Probe

        对左表中的对应数据依次hash，在映射表中寻找对应的元组

    可以通过布隆过滤器进行优化(Bloom Filter)，在查找映射表之前可以先检查过滤器，如果过滤器中没有，则说明没有对应的数据。

    cost: $B(B-1)$

    * B-1 "spill partitions" in Phase #1
    * Each should be no more than B blocks big

    如果没有足够的内存储存哈希表：Grace Hash Join

    * 用相同的哈希函数分别对左右表建立哈希映射表，将映射表中的对应区域读入内存执行nested loop join

    * 如果对应区域依然不能存入内存，可以用另一个哈希函数h2将较大的区域分割开再存入内存
    * cost: $3(M+N)$

Hash join 几乎总是最优的，除非数据已经是排好序的

#### 执行优化

* 启发式规则：重写表达式避免低效率操作
* 基于代价搜索：利用模型估算执行计划的成本，从多个等效的计划中选择成本最低的
* NP-HARD

###### 启发式规则

1. 尽早执行选择和投影
2. 中断复杂谓词，（从表达式树上）向下推
3. 嵌套查询拆分成两块
4. ……

###### 基于代价的优化

建立数据范围的直方图，通过直方图估算选择的元组个数。

现代的DBMS从元组中选择一定数量的样本进行估算，当表发生显著变化时更新样本。

MULTI-REL ATION QUERY PL ANNING

1. 通过左深连接树，枚举所有所有连接排序，通过动态规划选择代价最小的方案
   
* 是指数时间的算法，代价依然昂贵
  
2. greedy join enumeration algorithm

    在每次循环中，选择使总代价最低的方案

    * 多项式时间算法，但结果不一定最优

3. Randomized algorithm

    随机重写查询方案，利用模拟退火等算法进行优化

4. Genetic algorithm(遗传算法)

    通过连接方案（结合子代）和随机突变进行优化

#### 一致性哈希

将Hash函数的值对应一个环，将节点均匀地分布在换上，要插入的数据得到哈希值之后，沿环顺时针走到的第一个节点即为插入的节点。删除和增加节点时，只需要移动该节点附近的节点数据。

![image-20210309163553427](C:\Users\ZJW\AppData\Roaming\Typora\typora-user-images\image-20210309163553427.png)

当节点太少或分布太密集时，可能会发生数据倾斜，即少数节点储存了大部分数据。一致性哈希引入虚拟节点机制，对每个服务节点计算多个哈希，每个计算结果在环上都对应一个虚拟节点，使得数据能够均匀排布。

