读操作：

* 先访问cache 
* 如果 cache hit  直接返回数据
* 如果cahe miss 则访问db  并且将数据set到缓存



写操作：（四种方式）（cache一定要设置过期时间）

1）先DEL缓存再写数据库

2）先写数据库再DEL 缓存

3）先update 缓存再写数据库

4）先写数据库再更新缓存

5）先DEL cache  再操作数据库 延迟500ms再DEL cahe



上面的4种写操作方式都有可能出现数据不一致的情况 

因为写和读是并发的，没法保证顺序。

1）如果删了缓存，还没有来得及写库，另一个线程就来读取，发现缓存为空，则去数据库中读取数据写入缓存，此时缓存中为脏数据。

2）如果先写了库，再删除缓存前，写库的线程宕机了，没有删除掉缓存，则也会出现数据不一致情况。

3）更新了缓存，然后写数据库的时候失败直接就有数据一致性问题了

4）先写数据库，然后更新缓存操作失败，数据不一致

5)  第5种方案，性能方面，有个sleep（500）的动作，自然好不到哪去，也依然存在不一致的问题,kn

总结：总和分析，我个人觉得还是，先写数据库再del的方案相对好点，尽管上面的方案有数据问题，但并不影响其适用性，在一些一致性要求不高的场景，依然可以好用



强一致性的实现：

强一致性的做法有两类：

* mysql的binlog复制数据（自己没做过），Gearman、Canal等等
* 重试+补偿（这个方案就没有一定的多确定的方案），比如一般会用mq去建立请求的消息队列~

实际上，强一致性，很少用到，分布式环境下，强一致性是很难做的，可能由于一系列的问题所有的线程，执行到一半，直接挂了，也是很有可能的。

如果要“保证”数据的安全性，那么会带来开销的进一步提升，以至于使用redis带来的性能优势都会丧失。正确的做法是区分不同的业务，使得并不需要“保证”数据一致性的场合，可以使用redis优化。而敏感的场合依然使用mysql或者其他数据库









