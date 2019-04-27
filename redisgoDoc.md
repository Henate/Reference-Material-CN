# Redis学习笔记

## 一、简介
### 1.参考链接
[Redis Github地址](https://github.com/antirez/redis)
[Redis-GO Github地址](https://github.com/gomodule/redigo)

## 二、redigo
### 1. 获取依赖

使用go get获取托管在github的redis包
```go
go get github.com/garyburd/redigo/redis
```

### 2. hello world
1. 使用Dial,DialWithTimeout或者NewConn函数来创建连接；
2. 使用defer对Conn进行关闭

```go
package main
import (        
    "fmt"       
    "github.com/gomodule/redigo/redis"
)
func main() {       
    Conn, err := redis.Dial("tcp", "localhost:6379")        
    if err != nil {               
        fmt.Println("Connect to redis error",err)               
        return      
    }       
    defer Conn.Close()
}
```

### 3. 命令执行

#### 使用Conn接口调用Do执行命令
当连接Redis后，一般使用Do函数执行后续命令：
```go
// Do sends a command to the server and returns the received reply.
Do(commandName string, args ...interface{}) (reply interface{}, err error)
```

- Do函数会根据参数的输入类型进行转化

转化关系如下表：

| Go Type | Conversion |
| --- | --- |
| []byte | Sent as is |
| string | Sent as is |
| int, int64 | strconv.FormatInt(v) |
| float64 | strconv.FormatFloat(v, 'g', -1, 64) |
| bool | true -> "1", false -> "0" |
| nil | "" |
| all other types | fmt.Print(v) |

- Redis 命令响应会用以下Go类型表示：

| Redis type | Go type |
| --- | --- |
| error | redis.Error |
| integer | int64 |
| simple string | string |
| bulk string | []byte or nil if value not present. |
| array | []interface{} or nil if value not present. |


可以使用GO的类型断言或者reply辅助函数将返回的interface{}转换为对应类型

#### SET命令
使用方法：
```go
reply, err := Conn.Do("SET", "key", "value")
```
- 使用小结
1. 若Redis中无此键值，则新建一个键值对；
2. 若Redis中存在此键值，新value覆盖旧value；
3. 使用附加参数：`"EX", "10"` 指定键值过期时间，单位为秒。过期后键值被删除；

- 附加参数
1. EX seconds -- 指定过期时间，单位为秒.
2. PX milliseconds -- 指定过期时间，单位为毫秒.
3. NX -- 仅当键值不存在时设置键值.
4. XX -- 仅当键值存在时设置键值. 

#### APPEND命令
使用方法：
```go
reply, err := Conn.Do("APPEND", "key", "value")
```
直接往redis追加一个键值

#### GET命令
使用方法：
```go
value, err := redis.String(Conn.Do("GET", "key"))
```
- 使用小结
GET取对应键值，如果键值不存在则nil会返回

### 4. 管道化(Pipelining)
#### 什么是管道化
请求/响应服务可以实现**持续处理新请求，即使客户端没有准备好读取旧响应**这样客户端可以发送多个命令到服务器而无需等待响应，最后在一次读取多个响应。这就是管道化(pipelining)，这个技术在多年就被广泛使用了。距离，很多POP3协议实现已经支持此特性，显著加速了从服务器下载新邮件的过程。 

#### 管道化方法
- 使用Send()，Flush()，Receive()方法支持管道化操作
```go
Send(commandName string, args ...interface{}) error
Flush() error
Receive() (reply interface{}, err error)
```
- 方法简介：
1. Send向连接的输出缓冲中写入命令。
2. Flush将连接的输出缓冲清空并写入服务器端。
3. Recevie按照FIFO顺序依次读取服务器的响应。

```go
c.Send("SET", "foo", "bar")
c.Send("GET", "foo")
c.Flush()c.Receive() // reply from SET
v, err = c.Receive() // reply from GET
```

#### 并发

连接并不支持并发调用写入方法(Send,Flush)或者读取方法（Receive）。但是连接支持并发的读写。 
因为Do方法组合了Send，Flush和Receive，Do不可以与其他方法并发执行。为了能够完全并发访问Redis，在gorouotine中使用线程安全的连接池来获取和释放连接。