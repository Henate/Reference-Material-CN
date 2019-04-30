# gRPC笔记

## 安装依赖
1. 安装gPRC
`go get -u google.golang.org/grpc`
在gRPC的[github release](https://github.com/protocolbuffers/protobuf/releases)处下载gRPC最新版本。并把$LOCAL\protoc-3.7.1-win64\bin加到环境变量中以使用gPRC

2. 安装Protoc Plugin
`go get -u github.com/golang/protobuf/{proto,protoc-gen-go}`
此步成功后将会在$GOPATH/bin目录下出现protoc-gen-go.exe，需要把$GOPATH/bin加入到环境变量才能使用protoc-gen-go对proto文件进行编译。

## IDL

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
- 在当前文件夹处执行：
`protoc --go_out=plugins=grpc:. *.proto`
或者
`Get-ChildItem *.proto |Resolve-Path -Relative | %{protoc $_ --go_out=.}`

- P.S：注意要在全部依赖注入后才生成proto，以保证生成函数的完整性。


## 最小程序

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