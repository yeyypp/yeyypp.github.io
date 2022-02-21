# Interview
## Operating System
- 进程，线程，协程
  - 进程是操作系统分配资源的最小单位，是一个程序在一段数据集合上运行的过程，线程是cpu调度的最小单位，是一段执行流，调度时会将此线程的寄存器存在内存，，共享进程地址空间，协程是用户态线程
  - 进程和线程由操作系统调度，协程由用户调度
  - 进程间通信：管道，共享内存，消息队列，信号量，信号，
## Go
- [GMP](https://morsmachine.dk/go-scheduler)
  - G:代表goroutine执，M:代表系统线程，P:代表一个调度的上下文，在运行时这里叫处理器，为了运行G任务，每一个M必须持有一个P，每个P会有自己的一个运行队列，当执行go statement时，会将其加入。一般M数等于系统核数，这样不会频繁触发系统的线程调度，
  调度发生在用户态。
  - 当发生系统调用时，M会放弃当前持有的P，调度器会确保有另外的M来持有这个P，当系统调用
  完成后，M会尝试偷另外一个线程的context，如果失败，则会把这个完成系统调用的G放到一个全
  局队列中，自己休眠。M
  - 当P的队列中没有G时，就会从全局队列中获取，context也会不定时检查全局队列中的G，如果全局队列也为空，则会尝试偷去其他上下文队列中的G
- G状态
  - runnable：在队列中
  - running：在执行中，与M，P绑定
  - syscall：在执行系统调用，与M绑定
  - waiting：被阻塞（IO，GC，chan）
  - dead：
- GC
  - 三色标记：全局变量以及局部变量为根变量，标记时会依次遍历对象的指针，黑色为已经遍历完的，灰色为正在遍历的，白色为没有遍历到。初始均为白色，可达的为灰色，当遍历完后，只会有黑色和白色对象，对白色进行回收。
  - 写屏障：编译时加入在写入指针前的一小段代码
  - 与Java的对比，Java分代回收，Go不分代

## select, poll, epoll
- select  
  - 允许程序监视多个fd直到有一个或多个事件变成ready。
  - select监视窗口有数量限制
  - 主要参数为三个fd的set，读写异常，注意，每一次调用后，set中只会留有就绪的事件，需要对每个set进行遍历，找到就绪事件，再使用时还需要初始化set。
- poll
  - 与select类似，同样可以监视多个fd，但是没有最大数量限制
  - 主要参数为一个pollfd的数组，pollfd里有三个参数，具体fd，感兴趣的events，真正返回的revents
  - 成功时，poll返回非零数，表示数组中有几个revents为非零
  - 不需要重新初始化数组
- [epoll](https://man7.org/linux/man-pages/man7/epoll.7.html)
  - 比poll更有效率
  - 主要思想是通过一个epoll instance，这是一个内核数据结构，是一个包含着两个链表的容器
  一个是interest list，一个是ready list
  - epoll create创造一个epoll instance 返回一个指向他的fd ，epoll ctl将感兴趣的fd加入
  到interest list，epoll wait 在没有ready事件时会阻塞，否则会返回一个ready item 从
  epoll ready list。
  - 只需检查epoll wait，不需要轮询
  - edge-triggered 模式中，会阻塞在同一个fd上，尽管这个fd的buffer中还存有数据，因为此模式下
  只关心fd是否有变化，若想使用此模式，需配合nonblocking fd，只在读完数据后才继续wait事件
  - level-triggered与poll一样

## Network
- DNS
  - 客户端像本地服务器迭代查询，服务器向上级服务器递归查询
- http
  - 数据格式：请求行
  - https：向服务器发起安全连接请求，通过非对称加密，对称加密，以及证书完成加密。A，B两端，B端将自己的公钥交给CA，CA用自己的私钥对其进行加密，并交给A端，A通过浏览器内置的CA公钥进行解密，并用B的公钥加密对称加密的密钥，发送B端，B收到后用私钥解密，得到密钥。
- TCP
  - 三次握手：客户端发送syn包，进入syn_sent状态，随机序列号 A，服务器收到后发送SYN+ACK， 以及acksequence A+1，squence B，客户端收到后返回ACK，sequence A+1，acksequenceB+1
  - 为什么需要三次握手：1，三次握手是确保连接成立的最小次数，2，两次握手，如果客户端没有没有收到服务器的SYN+ACK，可能会再次发送一个SYN，最后可能建立多条连接
  - 与UDP区别：tcp是可靠的，udp是不可靠的，tcp面向字节流，udp面向报文，面向字节流的意思是上层协议也不知道一次读取了多少数据，需要根据数据中头部信息判断数据大小，udp的读每次会读一个数据包
  - 为什么可靠：确认，重传机制，以及流量控制，和拥塞控制
  - 流量控制：避免接受者来不及接受。通过接收方返回的ACK中会包含自己能接受的窗口大小。当发送0后，发送方会开始一个  计时器，时间一到会发送报文询问窗口大小。
  - 拥塞控制：避免网络中的数据过多，负载过大。
    - 慢开始：拥塞窗口成指数性增长
    - 拥塞避免：窗口大于阈值时，大小每次增加1
    - 当发生丢包了，会把阈值设为当前窗口的一半，窗口大小变为1，从慢开始
    - 快重传：收到三个相同ACK后，就知道丢包了，忽略计时器，发送丢失数据
    - 快恢复：收到三个ACK后，认为网络没有拥塞，会把窗口设为阈值一半，开始拥塞避免
    - Nagle:任何时刻，连接中只允许有一个未被确认的小包
  - 四次挥手：A发送FIN，seq M并进入FIN WAIT1，B收到，并发送ACK M + 1并进入CLOSE WAIT，A收到进入FIN WAIT2，B发送FIN N并进入LAST ACK，A收到发送ACK N+1并进入TIME WAIT。
  - 客户端异常断开会出现close wait，无法超时消失，通过epoll设置
  - 为什么四次握手：保证双方都没后续数据要发送
  - 为什么等time wait 是两个MSL：1，保证这个连接中发送的数据在网络中全部消失，2，防止服务器没有收到FIN包。
## Redis
- Why redis is single threaded 
## Database
- 事务
  - 隔离级别：
    - 未提交读：允许脏读，A，B，A读到了B修改的但未提交的数据，B失败后，读到的就是脏数据
    - 提交读：只能读取到已经提交的数据，不能防止可重复读，A，B，A事务中，前后两次读取到在B修改的数据，不一致
    - 可重复读：InnoDB默认级别，一次事务中两次操作，读取的数据一样。通过锁来避免，当
    - 幻读:主要发生在insert
  - [InnoDB中的多版本控](https://tech.meituan.com/2014/08/20/innodb-lock.html)
  - MVCC,通过undo log 实现，每一条record后还有三个隐藏字段,其中一个是修改数据的事务id。在事务开始时，读数据会创建一个read view，里边记录当前活跃的事务id，最小的最大的分别为低水位及高水位，发生读操作时，通过对比该操作的事务id，如果是在低水位以前，说明是已经提交的，可见，否则顺着undolog的指针向上查找，直到记录的事务id小于低水位。
  - redolog用来crash recovery，在将page改动写入redolog
  - double write buffer，数据从buffer pool先写到dwb，dwb先写入自己的磁盘文件，dwb再入磁盘数据文件
- 索引
  - 最左前缀：现在联合索引是a，b，c，查询a，ab，abc时会使用联合索引，ac会使用a的索引查询
  - 索引失效：
  
## Redis
- LRU
  - lkeys-lru, volatile-lru(only evict expire keys), use maxmemory to set the memory limit.
  - The lru in redis is a approximated lru algorithms.(In some database management, the buffer pool manager uses approximated lru to evict pages)
    - In redis, it samples 5 or more keys, then evict the obj which has the most idle time.
  
```
type LRUCache struct {
	head, tail *Node
	m map[int]*Node
	count, capacity int
}

func Constructor(capacity int) *LRUCache{
	head := &Node{
		key: 0,
		value: 0,
	}

	tail := &Node{
		key:0,
		value:0,
	}
	head.next = tail
	tail.pre = head
	return &LRUCache{
		head:head,
		tail: tail,
		count: 0,
		capacity: capacity,
		m: make(map[int]*Node),
	}
}

func (lru *LRUCache) Put(key, value int)  {
	if node, ok := lru.m[key]; ok {
		node.value = value
		lru.moveToForward(node)
		return
	}
	node := &Node{
		key: key,
		value: value,
	}
	lru.m[key] = node
	lru.addToHead(node)
	lru.count++
	if lru.count > lru.capacity {
		last := lru.tail.pre
		lru.removeNode(last)
		delete(lru.m, last.key)
		last = nil
	}
}

func (lru *LRUCache) Get(key int) int {
	if node, ok := lru.m[key]; ok {
		lru.moveToForward(node)
		return node.value
	}
	return -1
}

type Node struct {
	key, value int
	pre, next *Node
}

func (lru *LRUCache) removeNode(node *Node) {
	pre, next := node.pre, node.next
	pre.next = next
	next.pre = pre
	node.next = nil
	node.pre = nil
}

func (lru *LRUCache) addToHead(node *Node) {
	pre, next := lru.head, lru.head.next
	node.pre = pre
	node.next = next
	lru.head.next = node
	next.pre = node
}

func (lru *LRUCache) moveToForward(node *Node) {
	lru.removeNode(node)
	lru.addToHead(node)
}
```
  
- Distributed locks
  - use set key value nx ex time (only set if key doesn't). unlock with a lua
  ```
  if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
  else
  return 0
  end
  ```
  - Red lock,accquire lock from redis node one by one, if get N = all/2 + 1 lock, then we think it gets the lock.
  - 5 data types
  
## MapReduce
- 为什么需要：数据量大，单机无法完成，需要多机器协作，必然会出现网络分区错误，或者节点crash，需要能有自恢复功能
- 难点：任务如何分割，才不会iu造成某一节点的任务时间过久。大部分任务需要经过多个map， reduce过程，维护成本高。
- 
## Rate Limit / Break
[break](https://zjykzk.github.io/posts/cs/dist/circuit-breaker/)
## Distributed ID
