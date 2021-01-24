# Interview
## Go
- [GMP](https://morsmachine.dk/go-scheduler)
  - G:代表goroutine执，M:代表系统线程，P:代表一个调度的上下文，在这里叫处理器，为了
  运行G任务，每一个M必须持有一个P，

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
## Redis
- Why redis is single threaded  
