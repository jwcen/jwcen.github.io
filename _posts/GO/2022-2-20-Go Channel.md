---
layout: post
title:  "Go语言-channel的使用和底层原理"
categories: [Go]
tags: [channel]
permalink: /posts/:categories/:title:output_ext
description: Go语言-channel的使用和底层原理
math: true
published: true 
---

> 如需转载，请附上链接：[https://jwcen.github.io/](https://jwcen.github.io/)
{: .prompt-tip}

* This will become a table of contents (this text will be scrapped).
{:toc}

Go并发哲学--**不要通过共享内存的方式进行通信，而是应该通过通信的方式共享内存**。

Go 中用于在goroutine之間数据传递的一种数据结构。
## 设计原理
- 先入先出FIFO
  - 先从 Channel 读取数据的 Goroutine 会先接收到数据；
  - 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利；

## 声明、传值、关闭
{% details Golang code %}
~~~go
//声明和创建
var ch chan int      // 声明一个传递int类型的channel, nil chan 读写会死锁
ch := make(chan int) // 使用内置函数make()定义一个channel
ch2 := make(chan interface{})         // 创建一个空接口类型的通道, 可以存放任意格式

type Equip struct{ /* 一些字段 */ }
ch2 := make(chan *Equip)             // 创建Equip指针类型的通道, 可以存放*Equip

//传值
ch <- value          // 将一个数据value写入至channel，这会导致阻塞，直到有其他goroutine从这个channel中读取数据
value := <-ch        // 从channel中读取数据，如果channel之前没有写入数据，也会导致阻塞，直到channel中被写入数据为止

ch := make(chan interface{})  // 创建一个空接口通道
ch <- 0 // 将0放入通道中
ch <- "hello"  // 将hello字符串放入通道中

//关闭
close(ch)            // 关闭channel
~~~
{% enddetails %}


## channel 类型
### 无缓冲和有缓冲区别
对无缓冲 channel 类型的发送与接收操作，一定要放在两个不同的 Goroutine 中进行，否则会导致 deadlock
- 同步。没有数据缓冲区，须收发双方到场直接交换数据。
  - 阻塞，直到有另一方准备妥当 或 通道关闭。
  - 可通过 cap == 0 判断为无缓冲通道。

- 异步。通道自带固定大小缓冲区（buffer）。
  - 有数据或有空位时，发送或接收不需要阻塞等待。
  - 用 cap、len 获取缓冲区大小和当前缓冲数据量。

### 无缓冲和有缓冲通道的应用
无缓冲channel的典型应用
用作信号传递
- 1 对 1 通知信号
{% details Golang code %}
~~~go
func worker() {
	println("worker is working...")
	time.Sleep(1 * time.Second)
}

func spawn(f func()) <-chan signal {
	c := make(chan signal)
	go func() {
		println("worker start to worker.")
		f()
		c <- signal(struct{}{})
	}()
	return c
}

func main() {
	println("start a worker")
	c := spawn(worker)
	<-c // 通知 main goroutine
	println("done")
}
~~~
{% enddetails %}

- 1 对 n 通知信号
{% details Golang code %}
~~~go
func worker(i int) {
	fmt.Printf("worker %d is working...\n", i)
	time.Sleep(1 * time.Second)
	fmt.Printf("worker %d is done\n", i)
}

func spawnGroup(f func(i int), num int, groupSignal <-chan signal) <-chan signal {
	c := make(chan signal)
	var wg sync.WaitGroup
	for i := 0; i < num; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			<-groupSignal
			fmt.Printf("worker %d start to worker.\n", i)
			c <- signal(struct{}{})
			f(i)
		}(i + 1)
	}

	go func() {
		wg.Wait()
		c <- signal(struct{}{})
	}()
	return c
}

func main() {
	println("start a group of workers...")
	groupSignal := make(chan signal)
	c := spawnGroup(worker, 5, groupSignal)
	time.Sleep(5 * time.Second)
	println("the group of workers start to work...")
	close(groupSignal) // 向所有 worker goroutine 广播 “开始工作”的信号
	<-c
	println("done!")
}
~~~
{% enddetails %}

关闭一个无缓冲 channel 会让所有阻塞在这个 channel 上的接收操作返回，从而实现了一种 1 对 n 的“广播”机制。
用于替代锁机制
有缓冲channel应用	
1. 消息队列
2. 用作计数信号量
  - 通道当前数据个数：当前同时处于活动状态（处理业务）的 Goroutine 的数量
  - 通道的容量，允许同时处于活动状态的 Goroutine 的最大数量。
  - 向channel 的一个发送操作表示获取一个信号量，从channel一个接收操作表示释放一个信号量。

## 数据结构
它底层是一个 hchan 的结构体。占用 8 个字节。
{% details Golang code %}
~~~go
type hchan struct {
    qcount uint   // 队列中的总数据
    dataqsiz uint   // 循环队列的大小
    buf unsafe.Pointer // 指向dataqsiz元素数组 指向环形队列
    elemsize uint16  //当前使用量
    closed uint32
    elemtype *_type // 元素类型
    sendx uint // 发送索引
    recvx uint // 接收索引
    recvq waitq // 接待员名单， 因recv而阻塞的等待队列。
    sendq waitq // 发送服务员列表， 因send而阻塞的等待队列。
    //锁定保护hchan中的所有字段，以及几个在此通道上阻止的sudogs中的字段。
      //按住此锁定时不要更改另一个G的状态（尤其是不要准备G），因为这可能会导致死锁堆栈缩小。
    lock mutex
}
~~~
{% enddetails %}
- 其核心是存放`channel`数据的环形队列
- lock 用来保护数据安全，goroutine 来访问 channel 的 buf 之前，需要先获取锁。
- 另一重要部分是`recvq`和`sendq`两个双向链表，前者是等待读通道(`<-channel`)的 goroutine 队列，后者是等待写通道(`channel <- xxx`)的 goroutine 队列。  
- 若一个 goroutine 阻塞于`channel`了，那么它就被挂在`recvq`或`sendq`队列中。`waitq`是链表的定义，包含一个头结点和一个尾结点：
`sudog` 表示一个在等待列表中的 Goroutine，该结构中存储了两个分别指向前后
```go
type waitq struct {
    first *sudog
    last  *sudog
} 

struct  SudoG
{
    G*    g;        // g和selgen构成
    uint32    selgen;        // 指向g的弱指针
    SudoG*    link;
    int64    releasetime;
    byte*    elem;        // 数据元素
};
```

SudoG里主要结构是一个`g`和一个`elem`。`elem`用于存储 goroutine 的数据。
- 读通道时，数据会从`Hchan`的`buf队列`中拷贝到`SudoG`的`elem`域。
- 写通道时，数据则是由`SudoG`的`elem`域拷贝到`Hchan`的队列中。


### 创建
```go 
ch := make(chan string)
```
`runtime.makechan`  

创建 channel 的逻辑主要分为三大块：  
- 当前 channel 不存在缓冲区，也就是元素大小为 0 的情况下，就会调用 mallocgc 方法分配一段连续的内存空间
- 当前 channel 存储的类型**存在指针引用**，就会为当前的 Channel 和底层的数组分配一块连续的内存空间。
- 通用情况，默认分配相匹配的连续内存空间。

> channel 的创建都是调用的 mallocgc 方法，也就是 channel 都是创建在堆上的。
> 因此 channel 是会被 GC 回收的，自然也不总是需要 close 方法来进行显示关闭了。

### 发送数据
```go 
go func() {
    ch <- "煎鱼"
}()
```

`runtime.chansend1`
1. 前置处理
   - 一开始 `chansend` 方法在会先判断当前的 `channel` 是否为 `nil`。
     - 若为 `nil`，在逻辑上来讲就是向 `nil channel`发送数据，就会调用 `gopark` 方法使得当前 Goroutine 休眠，进而出现死锁崩溃，表象就是出现 `panic` 事件来快速失败。
   - 对非阻塞的 channel 进行一个上限判断，看看是否快速失败
     - 若非阻塞且未关闭，`dataqsiz` 为 0（缓冲区无元素），则会返回失败
     - 若是 `qcount` 与 `dataqsiz` 大小相同（缓冲区已满）时，则会返回失败
2. 上互斥锁
   - 前置判断后，即将在进入发送数据的处理前，channel 会进行上锁，保住并发安全
3. 直接发送
   - 当前 channel 有正在阻塞等待的接收方，那么只需要直接发送即可
- 缓冲发送
  - 有缓冲通道数据没有装满
  - 首先会使用 runtime.chanbuf **计算出下一个可以存储数据的位置**，
  - 然后通过 runtime.typedmemmove **将发送的数据拷贝到缓冲区中**并增加 sendx 索引和 qcount 计数器
  - 当 sendx 等于 dataqsiz 时会重新回到数组开始的位置
- 阻塞发送
  - 当 Channel 没有接收者能够处理数据时，向 Channel 发送数据会被下游阻塞（使用 select 关键字可以向 Channel 非阻塞地发送消息）
  - 




> 快速回答：
> 通道写入数据的过程中，首先判断是否有正在等待读取的协程，如果有的话，复制数据给此协程； 否则继续判断是否有空闲缓冲区，如果有的话把数据复制到缓冲区；否则，把当前goroutine放入等待写入队列。

### 接收数据 




----
参考

> 如需转载，请附上链接：[https://jwcen.github.io/](https://jwcen.github.io/)
{: .prompt-tip}