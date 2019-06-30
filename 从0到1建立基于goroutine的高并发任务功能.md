## 开始一个最小的多线程模型

在Go语言中，我们可以在函数或方法前添加 go 关键字能够在新的 goroutine 中调用它。当调用完成后， 该 goroutine 也会安静地退出。

在某函数前调用：
```go
go Handle(req)
```

在函数内使用闭包调用：
```go
func Printinfo() { 
    go func() {         
    fmt.Println("hello goroutine") 
    }()
}
```
以上的实现的函数功能没有任何实用性，因为没有实现完成时的信号处理。而完成信号处理，需要我们今天的主角：信道(channel)

## Channel
```go
ci := make(chan int) // 整数类型的无缓冲信道 
cj := make(chan int, 0) // 整数类型的无缓冲信道 
cs := make(chan *os.File, 100) // 指向文件指针的带缓冲信道
```

### 使用无缓冲channel进行简单通信
无缓冲channel在通信时会阻塞地同步交换数据，确保两个通信的goroutine处于确定的状态。

在下面实例中，GetInfo()一定会在执行PrintInfo()完毕之前都处于阻塞状态。通过channel确保两个goroutine的执行顺序逻辑。
```go
var ch = make(chan interface{}, 0)

func Printinfo() { 
    go func() {         
        fmt.Println("hello goroutine")         
        ch <- "SUCCESS"
    }()
    
}func GetInfo() { 
    go func() {         
        var value = <-ch         
        fmt.Println("Get goroutine:", value) 
    }()
}
```
因此我们可以确定：
- 若信道是不带缓冲的，那么在接收者收到值前， 发送者会一直阻塞

### 使用带缓冲的channel进行简单通信

带缓冲的信道可被用作信号量，例如限制吞吐量。同一时间只允许执行一定数量的goroutine：
```go
var Orderch = make(chan int, MaxOutstanding)

//使用serve创建的goroutine,同时只允许MaxOutstanding个goroutine运行
func serveOrder(orders ...interface{}) { 
    for _, order := range orders {         
        Orderch <- 1    //必须要在此位置阻塞，几时在goroutine内部阻塞，主线程还是会不断使用栈内存开新线程造成内存浪费。         
        go func() {                 
            processOrder(order)                
            <-Orderch         
        }() 
    }
}
```

若你此时进行debug，你会发现一个很明显的bug。在for循环中的变量order在每次迭代都会被重用，即order**变量在所有goroutine间共享**，这显然不符合我们的通信要求。

> 以下举两个例子可以避免以上的BUG。

#### Fix Bug 1.往goroutine闭包传递参数

```go
func serveOrder(orders ...interface{}) { 
    for _, order := range orders {         
        Orderch <- 1         
        go func(order interface{}) {                 
            processOrder(order)                
            <-Orderch         
        }(order) 
    }
}
```

#### Fix Bug 2. 使用同名变量作goroutine参数

```go
func serveOrder(orders ...interface{}) { 
    for _, order := range orders {   
        order := order      //创建同名变量
        Orderch <- 1         
        go func() {                 
            processOrder(order)                
            <-Orderch         
        }() 
    }
}
```
注：不使用同名变量也可以达到同样效果

### 在channel中使用channel回复
- 我们假设channel通信由发送方端发起的，那么我们如何才能用最简单直接的方法回复给发送方呢？

我们首先定义一个带有channel成员的request结构体。
```go
type request struct { 
    arg      []int 
    f        func([]int) int 
    resultch chan int   //通信
}
```

在接受方中就可以直接使用上述定义的channel进行回复：
```go
func Serve(queue *request) {
    for req := range queue {        
        req.resultch <- req.f(req.arg) 
    }
}
```