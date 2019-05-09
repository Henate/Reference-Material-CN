# gRPC 101

## 安装依赖
### 安装gPRC
`go get -u google.golang.org/grpc`


### 安装Protobuf
#### win10环境
在Protocbuf的[github release](https://github.com/protocolbuffers/protobuf/releases)处下载Protocbuf最新版本。并把$LOCAL\protoc-3.7.1-win64\bin加到环境变量中以使用Protocbuf.

执行命令：
```go
go get -u github.com/golang/protobuf/protoc-gen-go
```
此步成功后将会在$GOPATH/bin目录下出现protoc-gen-go.exe，需要把$GOPATH/bin加入到环境变量才能使用protoc-gen-go对proto文件进行编译。

#### Linux环境
执行命令：
```bash
wget https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-all-3.5.1.zip
unzip protobuf-all-3.5.1.zip
cd protobuf-3.5.1/
./configure
make
make install
```



## Protobuf

### Protobuf特性

![3283ef24196a6b27b78a10bc2d78173e.png](en-resource://database/472:1)


>protocol buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。

注：序列化：将结构数据或对象转换成能够被存储和传输（例如网络传输）的格式，同时应当要保证这个序列化结果在之后（可能在另一个计算环境中）能够被重建回原来的结构数据或对象。

### Protobuf/XML/JSON对比

- XML、JSON、ProtoBuf 都具有数据结构化和数据序列化的能力。
1. XML、JSON 更注重**数据结构化**，关注人类可读性和语义表达能力；
2. ProtoBuf 更注重**数据序列化**，关注效率、空间、速度，人类可读性差，语义表达能力不足（为保证极致的效率，会舍弃一部分元信息）；
3. ProtoBuf 的应用场景更为明确，XML、JSON 的应用场景更为丰富。

#### .proto文件的信息结构体
在.proto文件中使用message结构体
```go
message ProtocStruct {
   string query = 1;
   int32 page_number = 2;
   int32 result_per_page = 3;
   
   enum Foo {
      FIRST_VALUE = 0;
      SECOND_VALUE = 1;
   }
  StreamPoint pt = 4;
  reserved 4, 6, 9 to 11;
  reserved "foo", "bar";
}

message StreamPoint {  
    string name = 1;  
    int32 value = 2;
}
```

编译后将生成同名结构体：
```go
type ProtocStruct struct {
    query string
    page_number int32
    result_per_page int32
    pt  *StreamPoint
    
}

type StreamPoint struct {        
    Name string   `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`      
    Value  int32    `protobuf:"varint,2,opt,name=value,proto3" json:"value,omitempty"`
 }

```



### 编译.proto文件
在 proto 文件夹下的 search.proto 文件中，写入如下内容
```go
syntax = "proto3";

package proto;

service SearchService {
    rpc Search(SearchRequest) returns (SearchResponse) {}
}

message SearchRequest {
    string request = 1;
}

message SearchResponse {
    string response = 1;
}
```
- 其中：service SearchService 为定下的Server&Client的请求回复任务Search。任何需要使用gRPC的方法都要在这里定义相关方法
   
- 在当前文件夹处执行：
`protoc --go_out=plugins=grpc:. *.proto`
或者
`Get-ChildItem *.proto |Resolve-Path -Relative | %{protoc $_ --go_out=.}`

- P.S：注意要在全部依赖注入后才生成proto，以保证生成函数的完整性。



## gRPC的最小程序

![4e69f94b51088937e992c043ed6a88bd.png](en-resource://database/470:1)

### Server

1. 调用grpc包中的NewServer()创建Server对象
2. 定义一个 Server handler函数，并与第一步中创建的Server对象注册到proto中
3. 监听tcp&port

```go
package main

import (
    "context"
    "log"
    "net"
    "google.golang.org/grpc"
    pb "github.com/Henate/gRPC-go/proto"
)

type SearchService struct{}

func (s *SearchService) Search(ctx context.Context, r *pb.SearchRequest) (*pb.SearchResponse, error) {
    return &pb.SearchResponse{Response: r.GetRequest() + " Server"}, nil
}

const PORT = "9001"

func main() {
    server := grpc.NewServer()
    pb.RegisterSearchServiceServer(server, &SearchService{})

    lis, err := net.Listen("tcp", ":"+PORT)
    if err != nil {
        log.Fatalf("net.Listen err: %v", err)
    }

    server.Serve(lis)
}
```

### Client

1. 连接Server监听的port
2. 使用第一步获得的连接创建Client对象
3. 发送RPC请求

```go
package main

import (
    "context"
    "log"
    "google.golang.org/grpc"
    pb "github.com/Henate/gRPC-go/proto"
)

const PORT = "9001"

func main() {
    conn, err := grpc.Dial(":"+PORT, grpc.WithInsecure())
    if err != nil {
        log.Fatalf("grpc.Dial err: %v", err)
    }
    defer conn.Close()

    client := pb.NewSearchServiceClient(conn)
    resp, err := client.Search(context.Background(), &pb.SearchRequest{
        Request: "gRPC",
    })
    if err != nil {
        log.Fatalf("client.Search err: %v", err)
    }

    log.Printf("resp: %s", resp.GetResponse())
}
```

### 创建Server与Client的连接

```go
go run Server.go
```

```go
go run Client.go
```

## gRPC Streaming Client and Server

### 使用 Streaming RPC的优势
1. 大规模数据包
2. 实时场景


### 编写.protoc
关键字的使用：
1. 使用service关键字定义一个gRPC服务
2. 使用stream关键字定义request或response是否为流方法

该service定义了3个方法：
1. List：服务器端流式 RPC
2. Record：客户端流式 RPC
3. Route：双向流式 RPC

```go
syntax = "proto3";

package proto;

service StreamService {
    rpc List(StreamRequest) returns (stream StreamResponse) {};

    rpc Record(stream StreamRequest) returns (StreamResponse) {};

    rpc Route(stream StreamRequest) returns (stream StreamResponse) {};
}


message StreamPoint {
  string name = 1;
  int32 value = 2;
}

message StreamRequest {
  StreamPoint pt = 1;
}

message StreamResponse {
  StreamPoint pt = 1;
}
```

### 编译.protoc生成函数详解

- ServiceDesc：represents an RPC service's specification.
```go
type ServiceDesc struct {        
    ServiceName string        
    // The pointer to the service interface. Used to check whether the user       
    // provided implementation satisfies the interface requirements.        
    HandlerType interface{}        
    Methods     []MethodDesc    //初始化非Stream方法        
    Streams     []StreamDesc      //初始化Stream方法  
    Metadata    interface{}
}
```

- 编译Stream.proto后将根据Strem这个名字生成以Stream开头的变量： `_StreamService_serviceDesc`

```go
var _StreamService_serviceDesc = grpc.ServiceDesc{        
    ServiceName: "proto.StreamService",        
    HandlerType: (*StreamServiceServer)(nil),        
    Methods:     []grpc.MethodDesc{},       
    Streams: []grpc.StreamDesc{               
        {                       
            StreamName:    "List",                       
            Handler:       _StreamService_List_Handler,                       
            ServerStreams: true,              
        },              
        {                       
            StreamName:    "Record",                      
            Handler:       _StreamService_Record_Handler,                      
            ClientStreams: true,               
        },               
        {                       
            StreamName:    "Route",                      
            Handler:       _StreamService_Route_Handler,                       
            ServerStreams: true,                       
            ClientStreams: true,               
        },        
    },        
    Metadata: "stream.proto",
}
```

- 在Service_serviceDesc中的Handler已自动生成，但其返回函数需要手动编写。如下例中为List

```go
func _StreamService_List_Handler(srv interface{}, stream grpc.ServerStream) error {        
    fmt.Println("List_Handler")        
    m := new(StreamRequest)       
    if err := stream.RecvMsg(m); err != nil {               
        return err        
    }        
    return srv.(StreamServiceServer).List(m, &streamServiceListServer{stream})
}
```
- 自定义List方法的实现，如下为实现调用stream.Send进行传递一系列信息。
```go
func (s *StreamService) List(r *pb.StreamRequest, stream pb.StreamService_ListServer)error{        
    for n := 0; n <= 6; n++ {               
        err := stream.Send(&pb.StreamResponse{                       
            Pt: &pb.StreamPoint{                              
                Name:  r.Pt.Name,                              
                Value: r.Pt.Value + int32(n),                      
            },              
    })               
        if err != nil {                       
            return err              
        }        
    }        
    return nil
}
```
#### 使用Recv读取Stream中gRPC消息体

`RecvMsg`会从流中读取完整的 gRPC 消息体
（1）RecvMsg 是阻塞等待的
（2）RecvMsg 当流成功/结束（调用了 Close）时，会返回 io.EOF
（3）RecvMsg 当流出现任何错误时，流会被中止，错误信息会包含 RPC 错误码。而在 RecvMsg 中可能出现如下错误：io.EOFio.ErrUnexpectedEOFtransport.ConnectionErrorgoogle.golang.org/grpc/codes

### Server-side streaming RPC：服务器端流式 RPC

#### 特性
![f451b9aa7a44bcb9a9f0977f33d0a855.png](en-resource://database/474:1)

服务器端流式 RPC，为单向流，并代指 **Server 为 Stream** 而 **Client 为普通 RPC** 请求

> 客户端发起一次普通的 RPC 请求，**服务端通过流式响应多次发送数据集**，**客户端 Recv 接收数据**集。

在写.proto文件时就需要定义server端的response为stream:

```go
service StreamService {
    rpc List(StreamRequest) returns (stream StreamResponse) {};
    ....
}
```

### Server端
1. 调用grpc包中的NewServer()创建Server对象
2. 完成SeverHandler函数
2. 定义一个 Server handler函数，并与第一步中创建的Server对象注册到proto中
3. 监听tcp&port


#### Client端

1. 连接Server监听的port
2. 使用第一步获得的连接创建Client对象
3. 发送RPC请求
```go
client := pb.NewStreamServiceClient(conn)
err = printLists(client, &pb.StreamRequest{Pt: &pb.StreamPoint{Name: "gRPC Stream Client: List", Value: 2018}})
if err != nil {        
    log.Fatalf("printLists.err: %v", err)
}
```
4. 由于要接收来自Server端的Stream数据，需要在for循环中使用stream.Recv()读取资料，直到收到EOF停止
```go
func printLists(client pb.StreamServiceClient, r *pb.StreamRequest) error {       
    stream, err := client.List(context.Background(), r)        
    if err != nil {               
        return err        
    }        
    for {               
            resp, err := stream.Recv()               
            if err == io.EOF {                      
                break            
            }               
            if err != nil {                       
                return err               
            }               
            log.Printf("resp: pj.name: %s, pt.value: %d", resp.Pt.Name, resp.Pt.Value)        
    }        
    return nil
}
```

### Client-side streaming RPC：客户端流式 RPC

#### 特性

![c3b501baa1211aac2602069cc2909b6b.png](en-resource://database/476:1)


#### Server端

在接收客户端Stream消息体需要对每一个 Recv 都进行处理。
当发现 io.EOF (流关闭) 后，通过`stream.SendAndClose`将最终的响应结果发送给客户端，同时关闭客户端正在另外一侧等待的 Recv。
```go
func (s *StreamService) Record(stream pb.StreamService_RecordServer) error {
    for {
        r, err := stream.Recv()
        if err == io.EOF {
            return stream.SendAndClose(&pb.StreamResponse{Pt: &pb.StreamPoint{Name: "gRPC Stream Server: Record", Value: 1}})
        }
        if err != nil {
            return err
        }

        log.Printf("stream.Recv pt.name: %s, pt.value: %d", r.Pt.Name, r.Pt.Value)
    }

    return nil
}
```

#### Client端

在Server端处调用了`stream.SendAndClose`发送最终响应后，在Client端处调用stream.CloseAndRecv().
```go
func printRecord(client pb.StreamServiceClient, r *pb.StreamRequest) error {
    stream, err := client.Record(context.Background())
    if err != nil {
        return err
    }

    for n := 0; n < 6; n++ {
        err := stream.Send(r)
        if err != nil {
            return err
        }
    }

    resp, err := stream.CloseAndRecv()
    if err != nil {
        return err
    }

    log.Printf("resp: pj.name: %s, pt.value: %d", resp.Pt.Name, resp.Pt.Value)

    return nil
}
```

### Bidirectional streaming RPC：双向流式 RPC

#### 特征

双向流式 RPC，顾名思义是双向流。由客户端以流式的方式发起请求，服务端同样以流式的方式响应请求首个请求一定是 Client 发起，但具体交互方式（谁先谁后、一次发多少、响应多少、什么时候关闭）根据程序编写的方式来确定（可以结合协程）

![20788db9944768139777032eef4f8fd2.png](en-resource://database/478:1)


#### Server端

```go

func (s *StreamService) Route(stream pb.StreamService_RouteServer) error {
    n := 0
    for {
        err := stream.Send(&pb.StreamResponse{
            Pt: &pb.StreamPoint{
                Name:  "gPRC Stream Client: Route",
                Value: int32(n),
            },
        })
        if err != nil {
            return err
        }

        r, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }

        n++

        log.Printf("stream.Recv pt.name: %s, pt.value: %d", r.Pt.Name, r.Pt.Value)
    }

    return nil
}
```
#### Client端
```go

func printRoute(client pb.StreamServiceClient, r *pb.StreamRequest) error {
    stream, err := client.Route(context.Background())
    if err != nil {
        return err
    }

    for n := 0; n <= 6; n++ {
        err = stream.Send(r)
        if err != nil {
            return err
        }

        resp, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            return err
        }

        log.Printf("resp: pj.name: %s, pt.value: %d", resp.Pt.Name, resp.Pt.Value)
    }

    stream.CloseSend()

    return nil
}
```

### TLS for gRPC

#### 生成TLS证书
私钥：
`openssl ecparam -genkey -name secp384r1 -out server.key`

公钥：
`openssl req -new -x509 -sha256 -key server.key -out server.pem -days 3650`

#### Server端

通过方法`NewServerTLSFromFile`输入：
1. 证书文件
2. 密钥构造 TLS 凭证
```go
func main() {
    c, err := credentials.NewServerTLSFromFile("../../conf/server.pem", "../../conf/server.key")
    if err != nil {
        log.Fatalf("credentials.NewServerTLSFromFile err: %v", err)
    }

    server := grpc.NewServer(grpc.Creds(c))
    pb.RegisterSearchServiceServer(server, &SearchService{})

    lis, err := net.Listen("tcp", ":"+PORT)
    if err != nil {
        log.Fatalf("net.Listen err: %v", err)
    }

    server.Serve(lis)
}
```

#### Client端

通过方法`NewClientTLSFromFile`输入
1. 证书文件

```go
func main() {
    c, err := credentials.NewClientTLSFromFile("../../conf/server.pem", "go-grpc-example")
    if err != nil {
        log.Fatalf("credentials.NewClientTLSFromFile err: %v", err)
    }

    conn, err := grpc.Dial(":"+PORT, grpc.WithTransportCredentials(c))
    if err != nil {
        log.Fatalf("grpc.Dial err: %v", err)
    }
    defer conn.Close()

    client := pb.NewSearchServiceClient(conn)
    resp, err := client.Search(context.Background(), &pb.SearchRequest{
        Request: "gRPC",
    })
    if err != nil {
        log.Fatalf("client.Search err: %v", err)
    }

    log.Printf("resp: %s", resp.GetResponse())
}
```

### CA for gRPC

#### 生成CA

- 引入遵守 X.509 标准的 CA 颁发的根证书保证证书的可靠性和有效性。

根证书（root certificate）是属于根证书颁发机构（CA）的公钥证书。我们可以通过验证 CA 的签名从而信任 CA ，任何人都可以得到 CA 的证书（含公钥），用以验证它所签发的证书（客户端、服务端）
它包含的文件如下：
1. 公钥
2. 密钥

##### 生成ca.key
`openssl genrsa -out ca.key 2048`
使用genras生成的ca.key同时包含公钥与私钥。

```go
//Output：
Generating RSA private key, 2048 bit long modulus
```

##### 生成自签CA证书ca.pem
`openssl req -new -x509 -days 7200 -key ca.key -out ca.pem`
```go
//Output
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:GD
Locality Name (eg, city) []:GZ
Organization Name (eg, company) [Internet Widgits Pty Ltd]:PIS
Organizational Unit Name (eg, section) []:ME
Common Name (e.g. server FQDN or YOUR name) []:Streaming-gRPC
Email Address []:Henate@126.com
PS D:\_Work\_Doing\gRPC-go\src\github.com\Henate\Streaming-gRPC\conf>
```

#### Server端证书

##### 生成Server端ECC密钥
命令：`openssl ecparam -genkey -name secp384r1 -out server.key`

##### 生成CSR
解释：CSR 是 Cerificate Signing Request 的英文缩写，为证书签名请求文件。主要作用是 CA 会利用 CSR 文件进行签名使得攻击者无法伪装或篡改原有证书
关键命令：`openssl req`
命令：`openssl req -new -key server.key -out server.csr`
    > 此步是使用server生成的key中公钥向CA申请证书签名请求。

##### 使用CA签名CSR
- 使用根CA证书对"请求签发证书"进行签发，生成x509格式证书
关键命令：`openssl x509`
命令：`openssl x509 -req -sha256 -CA ca.pem -CAkey ca.key -CAcreateserial -days 3650 -in server.csr -outserver.pem`
    > CA证书+CA私钥签名server的CSR生成x509CA签名证书

输出：
```go
Signature ok
subject=C = CN, ST = GD, L = GZ, O = PIS, OU = ME, CN = Streaming-gRPC, emailAddress = Henate@126.com
Getting CA Private Key
```

#### Client端证书

##### 生成Client端ECC密钥
`openssl ecparam -genkey -name secp384r1 -out client.key`
##### 生成 CSR
关键命令：`openssl req`
`openssl req -new -key client.key -out client.csr`
##### 基于 CA 签发
关键命令：`openssl x509`
- 使用根CA证书对"请求签发证书"进行签发，生成x509格式证书
`openssl x509 -req -sha256 -CA ca.pem -CAkey ca.key -CAcreateserial -days 3650 -in client.csr -out client.pem`

#### Server端实现

1. 使用`tls.LoadX509KeyPair()`读取server端的私钥+CA签名server证书，返回一个密钥对`Certificate`。
2. 使用`certPool`的方法`AppendCertsFromPEM()`校验密钥对是否能通过CA证书的验证，若通过，加入到`certPool`中
3. 使用`credentials.NewTLS()`配置tls，参数有密钥对`Certificate`+校验方式+`certPool`

#### Client端实现
1. Client 通过请求得到 Server 端的证书
2. 使用 CA 认证的根证书对 Server 端的证书进行可靠性、有效性等校验
3. 校验 ServerName 是否可用、有效 




