# Interview
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
  - 三色标记：初始均为白，然后从root开始标记，可达的均为灰，将灰色对象取出，标记其引用为灰，并标记自己为黑，直到所有灰色均为黑，此时白色的就为垃圾，进行回收
  - 写屏障：编译时加入在写入指针前的一小段代码
  - 定

## select, poll, epoll
- select  
  - 允许程序监视多个fd直到有一个或多个事件变成ready。
  - select监视窗口有数量限制
  - 主要参数为三个fd的set，读写异常，注意，每一次调用后，set中只会留有就绪的事件，
  通过轮询的方式读数据，当再次使用select前，需要初始化set，把fd放进set中。
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
  - 
## Redis
- Why redis is single threaded 
## Git
- git rebase
