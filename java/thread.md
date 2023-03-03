## java锁类型

### 乐观锁/悲观锁

    乐观锁：同一时间对同一个数据操作，不会发生修改，只有在更新数据的时候通过cas的方式去更新数据；适用于读比较多的场景
    悲观锁：同一时间对同一个变量进行操作，认为一定会有修改，所以必须加锁；试用写比较多的场景

### 公平锁/非公平锁

    公平锁：按照线程申请锁的顺序来获取锁

    非公平锁：和公平锁相反；吞吐量大于公平锁；ReentrantLock默认非公平；Synchronized只能是非公平锁

## 独享锁/共享锁

    独享锁：只能被一个线程拥有ReentrantLock

    共享锁：可以多个线程持有；ReentrantReadWriteLock

### 可重入锁

    ReentrantLock，sychrnozied都是可重入锁

### 分段锁

    ConcurrentHashMap



    

### 
