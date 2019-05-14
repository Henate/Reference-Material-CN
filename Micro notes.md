# Micro

## 参考资料
[Micro文档](https://micro.mu/docs/cn/index.html)
[Micro工具集](https://micro.mu/blog/2016/03/20/micro.html)
[Micro GoDoc](https://godoc.org/github.com/micro/go-micro#Service)



## Micro开源库简介

- [go-micro](https://github.com/micro/go-micro) - 基于Go语言的可插拔RPC微服务开发框架；包含服务发现、RPC客户/服务端、广播/订阅机制等等。
- [go-plugins](https://github.com/micro/go-plugins) - go-micro的插件有etcd、kubernetes、nats、rabbitmq、grpc等等。
- [micro](https://github.com/micro/micro) - 微服务工具集包含传统的入口点（entry point）；API 网关、CLI、Slack Bot、代理及Web UI。

### 框架 go-micro
参考地址：[github.com/micro/go-micro](github.com/micro/go-micro)
Go-micro是独立的RPC框架，可分为7个大的模块组成：
`Client` `Server` `Broker` `Codec` `Registry` `Selector` `Transport`
![1df0cf3ee92d02d1036858846b754ba1.png](en-resource://database/487:1)

各个模块在框架代码：[Micro Github](https://github.com/micro/go-micro)上托管为每个模块单独一个文件夹
![6fc27904de6b3474bc80eee9a18cd316.png](en-resource://database/499:1)


#### Registry

Registry注册模块提供可插拔的服务注册与发现功能，当前实现的方式有`Consul`，`etcd`，内存和`k8s`。即使需求不同，接口也比较容易实现。

#### Selector

Selector选择器通过选举提供负载均衡机制。当客户端请求服务时，它首先要向查询到服务注册信息，一般情况是返回所需服务的注册成功的节点列表。选择器会选择其中一个用来提供服务。多次调用选择器会触发均衡算法，当前的算法有`round robin（循环调度）`，`哈希随机`和`黑名单`。

#### Broker

Broker是事件广播/订阅（PUB/SUB）可插件接口。对于事件驱动的微服务架构，消息广播与订阅得放在首要位置。目前的实现有NATs、rabbitmq、http (仍在开发中)。

#### Transport
Transport传输也是可插拔的点到点消息传输接口，目前的实现有http，rabbitmq，nats。通过提供这种抽象，可以无缝地进行交换传输。

#### Client
Client客户端提供发起RPC请求的能力。它集合了注册（registry）、选择器（selector）、broker、传输（transport），当然也具备重试、超时、上下文等等。

#### Server

Server服务就是运行了真实的微服务的程序，它提供的服务通过RPC请求的完成。

#### Plugins
[go-plugins](https://github.com/micro/go-plugins)提供了各个模块的可替换方案。只需要在代码中重新注册某个模块即可替换默认的模块方案。

### 微服务工具库：Micro
参考地址：[github.com/micro/micro](github.com/micro/micro)
![123b9d74630c04db4e8ab880f3c8acb1.png](en-resource://database/503:1)

开源库下的各模块单独为一个目录：
- api - API Gateway 网关。它是独立的HTTP入口，基于服务发现机制实现动态路由。
- web - Web Dashboard web控制台。 提供可视化的发现与管理监控界面。
- cli - Command line interface 命令行接口。提供描述、查询终端服务的交互入口。
- bot - Slack与hipchat bot消息通知工具。也就是通过消息传递的CLI。
- new - 新服务构建模板。
![da7b28585dea1ecf142b57e3112a54f0.png](en-resource://database/505:1)



## Micro模块用于sidecar
参考[文章](https://micro.mu/blog/2016/03/20/micro.html)：
![a6dd6e247c5fcb57af84ead85d657928.png](en-resource://database/489:1)
Go Micro可作为一个sidecar 通过HTTP与父程序进行通信。
特性：
1. Registration with discovery system
2. Host discovery of other services
3. Healthchecking of the main application
4. A proxy to make RPC requests
5. PubSub via WebSockets


## 构建一个Go-micro服务

### 初始化服务器

#### 1.使用option方法创建micro的NewService

代码：创建Service，参数Name为.protoc中定义的package
```go
service := micro.NewService(        
    micro.Name("go.micro.api.example"),
)
```
> 代码详解：

`Option`结构体定义，主要为Micro模块：
```go
type Options struct {        
    Broker    broker.Broker        
    Cmd       cmd.Cmd        
    Client    client.Client        
    Server    server.Server        
    Registry  registry.Registry        
    Transport transport.Transport        
    // Before and After funcs        
    BeforeStart []func() error        
    BeforeStop  []func() error        
    AfterStart  []func() error        
    AfterStop   []func() error
    
    // Other options for implementations of the interface        
    // can be stored in a context 
    
    Context context.Context
}
```
封装结构体`Option`为一个函数
```go
type Option func(*Options)
```

使用`micro.NewService()`创建服务，传入参数类型为` ...Option`
```go
service := micro.NewService(
        micro.Name("greeter"),
        micro.Version("latest"),
)
```

`NewService()`通过接收`Option`类型参数进行初始化
```go
/*
接收类型：...Option
返回类型：Service
执行目的：直接调用newService()
*/
func NewService(opts ...Option) Service {        
    return newService(opts...)
}
```

```go

/*
接收类型：...Option
返回类型：Service
执行目的：newService()参数接收类型为Option，因此只需要使用返回值为Option的函数就可以完成赋值。
*/
func newService(opts ...Option) Service {        
    options := newOptions(opts...)        
    options.Client=&clientWrapper{               
        options.Client,               
        metadata.Metadata{                       
            HeaderPrefix + "From-Service": options.Server.Options().Name,               
        },        
    }        
    return &service{               
        opts: options,        
    }
}
```

```go
/*
接收类型：...Option
返回类型：Options
执行目的：完成Service赋值
*/
func newOptions(opts ...Option) Options {        
    opt := Options{               
        Broker:    broker.DefaultBroker,               
        Cmd:       cmd.DefaultCmd,               
        Client:    client.DefaultClient,               
        Server:    server.DefaultServer,               
        Registry:  registry.DefaultRegistry,               
        Transport: transport.DefaultTransport,               
        Context:   context.Background(),       
    }        
    for _, o := range opts {               
        o(&opt)        
    }        
    return opt
}
```

#### 2.解析标识参数

代码：
```go
service.Init()
```

参考[标识文章](https://godoc.org/github.com/micro/go-micro/cmd#pkg-variables)
Go Micro提供预置的标识，调用`service.Init`执行时就会设置并解析这些参数

#### 3.实现处理器方法

通过.protoc定义的servive可自动生成处理器ExampleService，其代码自动生成如下：
```go
type ExampleService interface {        
    Call(ctx context.Context, in *go_api.Request, opts ...client.CallOption) (*go_api.Response, error)
}
```

以上自动生成的接口方法Call并没有真正地在service实现，需要手动实现处理器ExampleService中定义的方法call作为handler服务。总共需要两步：
```go
//第一步：定义service结构体
type Example struct{}

//第二步：实现方法Call
func (e *Example) Call(ctx context.Context, req *proto.CallRequest, rsp *proto.CallResponse) error {        
    log.Print("Received Example.Call request")        
    if len(req.Name) == 0 {               
        return errors.BadRequest("go.micro.api.example", "no content")        
    }        
    rsp.Message = "got your request " + req.Name       
    return nil
}
```

#### 4.注册Service handler函数

.protoc文件中定义的service：
```go
service Example {        
    rpc Call(go.api.Request) returns(go.api.Response) {};
}
service Foo {        
    rpc Bar(go.api.Request) returns(go.api.Response) {};
}
```

当使用protoc生成protobuf代码代码后，可使用`proto.RegisterXXXXHandler`对.protoc文件中定义的service进行注册
```go
protoc --proto_path=$GOPATH/src:. --micro_out=. --go_out=. greeter.proto
```
> 由于在3中已完成处理器Example的方法Call()，因此可以直接调用生成的代码注册service。

`service`为使用`micro.NewService()`创建的`service`，使用`RegisterExampleHandler()`把`Example`注册到`service`之中
```go
proto.RegisterExampleHandler(service.Server(), new(Example))
```
使用`RegisterFooHandler()`把`Foo`注册到`service`之中
```go
proto.RegisterFooHandler(service.Server(), new(Foo))
```


#### 5.运行服务
调用run方法运行已设置好的服务
```go
if err := service.Run(); err != nil {
        log.Fatal(err)}
```

### 初始化客户端

```go
func main() {        
    // create a new service        
    service := micro.NewService()      
    
    // parse command line flags        
    service.Init()        
    
    // Use the generated client stub        
    cl := hello.NewSayService("go.micro.srv.greeter", service.Client())    
    
    // Make request        
    rsp, err := cl.Hello(context.Background(), &hello.Request{               
        Name: "John",        
    })        
    if err != nil {               
        fmt.Println(err)               
    return       
    }        
    fmt.Println(rsp.Msg)
}
```
- 注意：在使用`NewXXXService`进行创建client端时，需要传递的第一个参数为.protoc定义的package
此例中为：`package go.micro.api.greeter;`







