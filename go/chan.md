# Channel
## 并发模型 : 顺序通信进程 Communicating sequential processes，CSP

## 运行规则
1. 先从 Channel 读取数据的 Goroutine 会先接收到数据；
2. 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利；
   
## 无锁管道
锁是一种常见的并发控制技术,一般分为乐观锁和悲观锁, 无锁队列更准确的描述是使用乐观并发控制的队列.        
乐观并发控制本质上是基于验证的协议，我们使用原子指令 CAS（compare-and-swap 或者 compare-and-set）在多线程中同步数据，无锁队列的实现也依赖这一原子指令。

## lock字段
当前的channel仍然使用互斥锁来避免数据竞争
问题:
1.  锁导致的休眠和唤醒会带来额外的上下文切换
2.  如果临界区过大，加锁解锁导致的额外开销就会成为性能瓶颈

## 2014 年提出了无锁 Channel 的实现方案
1.  同步channel 不需要缓冲区,数据直达接受方
2.  异步channel 基于环形缓冲的传统生产消费模型
3.  chan struct{} 类型的异步 Channel — struct{} 类型不占用内存空间，不需要实现缓冲区和直接发送（Handoff）的语义；

## 发送数据

1. 在发送数据的逻辑执行之前会先为当前 Channel 加锁，防止多个线程并发修改数据
2. 如果 Channel 已经关闭，那么向该 Channel 发送数据时会panic “send on closed channel” 错误并中止程序
### ch <- i 表达式向 Channel 发送数据时遇到的几种情况
1. 如果当前ch的recvq上有已经被阻塞的gr,那么会直接将数据发送给当前的gr,并将其设置为下一个执行的gr
2. 如果 ch 上有缓存区,并还有空闲容量, 我们会直接将数据存储在buf上sendx的位置上,
3. 如果上面两种情况都不满足,会创建一个sudog,并将其加入到ch的sendq队列中,当前gr陷入阻塞等待其他gr读取数据

### 触发调度
1.  发送数据时,发现ch 上存在等待接受数据的gr,立刻设置p的runnext属性,但不会立刻触发调度
2.  发送数据时, 没有找到接受方, 并且缓冲区已经满了,  这时 会将自己加入chan的sendq并调用runtime.goparkunlock触发gr调度让出处理器的 使用权
