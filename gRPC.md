# gRPC笔记

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

### 为什么用 Streaming RPC
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