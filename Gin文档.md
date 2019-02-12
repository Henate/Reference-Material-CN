# Gin 官方文档 (中文)

此文档翻译于Gin官方README.md，如需查看最新的更新文档以及源代码，请移步[官方Github](https://github.com/gin-gonic/gin)

<img align="right" width="159px" src="https://raw.githubusercontent.com/gin-gonic/logo/master/color.png">

[![Build Status](https://travis-ci.org/gin-gonic/gin.svg)](https://travis-ci.org/gin-gonic/gin)
[![codecov](https://codecov.io/gh/gin-gonic/gin/branch/master/graph/badge.svg)](https://codecov.io/gh/gin-gonic/gin)
[![Go Report Card](https://goreportcard.com/badge/github.com/gin-gonic/gin)](https://goreportcard.com/report/github.com/gin-gonic/gin)
[![GoDoc](https://godoc.org/github.com/gin-gonic/gin?status.svg)](https://godoc.org/github.com/gin-gonic/gin)
[![Join the chat at https://gitter.im/gin-gonic/gin](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/gin-gonic/gin?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Sourcegraph](https://sourcegraph.com/github.com/gin-gonic/gin/-/badge.svg)](https://sourcegraph.com/github.com/gin-gonic/gin?badge)
[![Open Source Helpers](https://www.codetriage.com/gin-gonic/gin/badges/users.svg)](https://www.codetriage.com/gin-gonic/gin)
[![Release](https://img.shields.io/github/release/gin-gonic/gin.svg?style=flat-square)](https://github.com/gin-gonic/gin/releases)

Gin是一个用Go(Golang)编写的网络架构。有着与Martini架构相似但性能更佳的API的特性，而且在[httprouter](https://github.com/julienschmidt/httprouter)的支持下性能有40倍的提高。如果你需要高性能和高生产力，你将喜欢上Gin。

![Gin console logger](https://gin-gonic.github.io/gin/other/console.png)

## 内容

- [Installation](#installation)
- [Prerequisite](#prerequisite)
- [Quick start](#quick-start)
- [Benchmarks](#benchmarks)
- [Gin v1.stable](#gin-v1-stable)
- [Build with jsoniter](#build-with-jsoniter)
- [API Examples](#api-examples)
    - [Using GET,POST,PUT,PATCH,DELETE and OPTIONS](#using-get-post-put-patch-delete-and-options)
    - [Parameters in path](#parameters-in-path)
    - [Querystring parameters](#querystring-parameters)
    - [Multipart/Urlencoded Form](#multiparturlencoded-form)
    - [Another example: query + post form](#another-example-query--post-form)
    - [Map as querystring or postform parameters](#map-as-querystring-or-postform-parameters)
    - [Upload files](#upload-files)
    - [Grouping routes](#grouping-routes)
    - [Blank Gin without middleware by default](#blank-gin-without-middleware-by-default)
    - [Using middleware](#using-middleware)
    - [How to write log file](#how-to-write-log-file)
    - [Custom Log Format](#custom-log-format)
    - [Model binding and validation](#model-binding-and-validation)
    - [Custom Validators](#custom-validators)
    - [Only Bind Query String](#only-bind-query-string)
    - [Bind Query String or Post Data](#bind-query-string-or-post-data)
    - [Bind Uri](#bind-uri)
    - [Bind HTML checkboxes](#bind-html-checkboxes)
    - [Multipart/Urlencoded binding](#multiparturlencoded-binding)
    - [XML, JSON, YAML and ProtoBuf rendering](#xml-json-yaml-and-protobuf-rendering)
    - [JSONP rendering](#jsonp)
    - [Serving static files](#serving-static-files)
    - [Serving data from reader](#serving-data-from-reader)
    - [HTML rendering](#html-rendering)
    - [Multitemplate](#multitemplate)
    - [Redirects](#redirects)
    - [Custom Middleware](#custom-middleware)
    - [Using BasicAuth() middleware](#using-basicauth-middleware)
    - [Goroutines inside a middleware](#goroutines-inside-a-middleware)
    - [Custom HTTP configuration](#custom-http-configuration)
    - [Support Let's Encrypt](#support-lets-encrypt)
    - [Run multiple service using Gin](#run-multiple-service-using-gin)
    - [Graceful restart or stop](#graceful-restart-or-stop)
    - [Build a single binary with templates](#build-a-single-binary-with-templates)
    - [Bind form-data request with custom struct](#bind-form-data-request-with-custom-struct)
    - [Try to bind body into different structs](#try-to-bind-body-into-different-structs)
    - [http2 server push](#http2-server-push)
    - [Define format for the log of routes](#define-format-for-the-log-of-routes)
    - [Set and get a cookie](#set-and-get-a-cookie)
- [Testing](#testing)
- [Users](#users)

## 安装

如需安装Gin包，首先你需要安装Go和设置你的GO工作空间。

1. 下载以及安装：

```sh
$ go get -u github.com/gin-gonic/gin
```

2. 导入到代码中：

```go
import "github.com/gin-gonic/gin"
```

3. （可选）如果需要使用到`http.StatusOK`等变量，则需要导入`net/http`。

```go
import "net/http"
```

### 使用依赖包管理工具[Govendor](https://github.com/kardianos/govendor)

1. `go get` govendor

```sh
$ go get github.com/kardianos/govendor
```
2. 创建工程目录并使用`cd`进入文件夹中

```sh
$ mkdir -p $GOPATH/src/github.com/myusername/project && cd "$_"
```

3. 使用Vendor初始化工程以及添加gin

```sh
$ govendor init
$ govendor fetch github.com/gin-gonic/gin@v1.3
```

4. 复制一个开始模板到工程中

```sh
$ curl https://raw.githubusercontent.com/gin-gonic/gin/master/examples/basic/main.go > main.go
```

5. 运行工程

```sh
$ go run main.go
```

## 依赖

现在运行Gin需要在Go 1.6以及更后的版本。且不久将会被要求运行在Go 1.7中。

## 快速启动
 
```sh
#假设以下代码在文件example.go中
$ cat example.go
```

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}
```

```
# run example.go and visit 0.0.0.0:8080/ping on browser
$ go run example.go
```

## 性能基准测试（Benchmarks）

Gin 所使用的是 [HttpRouter](https://github.com/julienschmidt/httprouter)的一个自定义版本

[查看所有 benchmarks](/BENCHMARKS.md)

Benchmark name                              | (1)        | (2)         | (3) 		    | (4)
--------------------------------------------|-----------:|------------:|-----------:|---------:
**BenchmarkGin_GithubAll**                  | **30000**  |  **48375**  |     **0**  |   **0**
BenchmarkAce_GithubAll                      |   10000    |   134059    |   13792    |   167
BenchmarkBear_GithubAll                     |    5000    |   534445    |   86448    |   943
BenchmarkBeego_GithubAll                    |    3000    |   592444    |   74705    |   812
BenchmarkBone_GithubAll                     |     200    |  6957308    |  698784    |  8453
BenchmarkDenco_GithubAll                    |   10000    |   158819    |   20224    |   167
BenchmarkEcho_GithubAll                     |   10000    |   154700    |    6496    |   203
BenchmarkGocraftWeb_GithubAll               |    3000    |   570806    |  131656    |  1686
BenchmarkGoji_GithubAll                     |    2000    |   818034    |   56112    |   334
BenchmarkGojiv2_GithubAll                   |    2000    |  1213973    |  274768    |  3712
BenchmarkGoJsonRest_GithubAll               |    2000    |   785796    |  134371    |  2737
BenchmarkGoRestful_GithubAll                |     300    |  5238188    |  689672    |  4519
BenchmarkGorillaMux_GithubAll               |     100    | 10257726    |  211840    |  2272
BenchmarkHttpRouter_GithubAll               |   20000    |   105414    |   13792    |   167
BenchmarkHttpTreeMux_GithubAll              |   10000    |   319934    |   65856    |   671
BenchmarkKocha_GithubAll                    |   10000    |   209442    |   23304    |   843
BenchmarkLARS_GithubAll                     |   20000    |    62565    |       0    |     0
BenchmarkMacaron_GithubAll                  |    2000    |  1161270    |  204194    |  2000
BenchmarkMartini_GithubAll                  |     200    |  9991713    |  226549    |  2325
BenchmarkPat_GithubAll                      |     200    |  5590793    | 1499568    | 27435
BenchmarkPossum_GithubAll                   |   10000    |   319768    |   84448    |   609
BenchmarkR2router_GithubAll                 |   10000    |   305134    |   77328    |   979
BenchmarkRivet_GithubAll                    |   10000    |   132134    |   16272    |   167
BenchmarkTango_GithubAll                    |    3000    |   552754    |   63826    |  1618
BenchmarkTigerTonic_GithubAll               |    1000    |  1439483    |  239104    |  5374
BenchmarkTraffic_GithubAll                  |     100    | 11383067    | 2659329    | 21848
BenchmarkVulcan_GithubAll                   |    5000    |   394253    |   19894    |   609

- (1): 固定时间内重复地测试并达到恒定值，数据值越高则代表越确切
- (2):单次重复时间 (ns/op)，数值越低越好
- (3): 堆内存 (B/op)，数值越低越好
- (4): 每次重复的平均内存分配时间(allocs/op)，数值越低越好

## Gin v1. 稳定版

- [x] 零分配路由器.
- [x] 最快的分配路由及编写代码http路由架构。
- [x] 拥有成套的单元测试
- [x] 通过Battle tested
- [x] 使用API冻结，新的版本更新后将不会影响到你原本的代码

## 使用 [jsoniter](https://github.com/json-iterator/go)

Gin使用的默认json包为 `encoding/json` ，但你可以通过构建其他tags参数将默认值更改为使用 [jsoniter](https://github.com/json-iterator/go)。

```sh
$ go build -tags=jsoniter .
```

## API 例子

### 使用 GET, POST, PUT, PATCH, DELETE 以及 OPTIONS

```go
func main() {
	// 禁止控制台颜色
	// gin.DisableConsoleColor()

	// 创建带有默认中间件的路由
	// 中间件：日志和恢复（无崩溃）
	router := gin.Default()

	router.GET("/someGet", getting)
	router.POST("/somePost", posting)
	router.PUT("/somePut", putting)
	router.DELETE("/someDelete", deleting)
	router.PATCH("/somePatch", patching)
	router.HEAD("/someHead", head)
	router.OPTIONS("/someOptions", options)

	// By default it serves on :8080 unless a
	// PORT environment variable was defined.
	router.Run()
	// router.Run(":3000") for a hard coded port
}
```

### 路径中的参数

```go
func main() {
	router := gin.Default()

        //这一个处理程序匹配如/user/john，并不会匹配/user/ 或者 /user
	router.GET("/user/:name", func(c *gin.Context) {
		name := c.Param("name")
		c.String(http.StatusOK, "Hello %s", name)
	})

	// However, this one will match /user/john/ and also /user/john/send
	// If no other routers match /user/john, it will redirect to /user/john/
	router.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
		message := name + " is " + action
		c.String(http.StatusOK, message)
	})

	router.Run(":8080")
}
```

### 查询字符串参数

```go
func main() {
	router := gin.Default()

    
        //查询字符串参数使用请求对象作解释
        //请求匹配以下格式的url将会获得响应： /welcome?firstname=Jane&lastname=Doe
	router.GET("/welcome", func(c *gin.Context) {
		firstname := c.DefaultQuery("firstname", "Guest")
		lastname := c.Query("lastname") // c.Request.URL.Query().Get("lastname")的快捷使用方式

		c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
	})
	router.Run(":8080")
}
```

### Multipart/Urlencoded Form

```go
func main() {
	router := gin.Default()

	router.POST("/form_post", func(c *gin.Context) {
		message := c.PostForm("message")
		nick := c.DefaultPostForm("nick", "anonymous")

		c.JSON(200, gin.H{
			"status":  "posted",
			"message": message,
			"nick":    nick,
		})
	})
	router.Run(":8080")
}
```

### 其他例子: query + post form

```
POST /post?id=1234&page=1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=manu&message=this_is_great
```

```go
func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {

		id := c.Query("id")
		page := c.DefaultQuery("page", "0")
		name := c.PostForm("name")
		message := c.PostForm("message")

		fmt.Printf("id: %s; page: %s; name: %s; message: %s", id, page, name, message)
	})
	router.Run(":8080")
}
```

```
id: 1234; page: 1; name: manu; message: this_is_great
```

### Map 作为查询字符串或者 postform 的参数

```
POST /post?ids[a]=1234&ids[b]=hello HTTP/1.1
Content-Type: application/x-www-form-urlencoded

names[first]=thinkerou&names[second]=tianou
```

```go
func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {

		ids := c.QueryMap("ids")
		names := c.PostFormMap("names")

		fmt.Printf("ids: %v; names: %v", ids, names)
	})
	router.Run(":8080")
}
```

```
ids: map[b:hello a:1234], names: map[second:tianou first:thinkerou]
```

### 上传文件

#### 单一文件

参考 issue [#774](https://github.com/gin-gonic/gin/issues/774) 以及 detail [example code](examples/upload-file/single).

`file.Filename` **不能** 相信有效性. 参考 [`Content-Disposition` on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition#Directives) and [#1693](https://github.com/gin-gonic/gin/issues/1693)

> 因为 filename 总是随意输入的因此不能盲目地使用到应用上：路径信息应当先被剥离出来，转化成为符合服务器文件系统规则后才能使用

```go
func main() {
	router := gin.Default()
	// 为multipart forms 设定最小的容量限制 (默认值为 32 MiB)
	// router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// 单一文件
		file, _ := c.FormFile("file")
		log.Println(file.Filename)

		// 上传文件到指定的 dst.
		// c.SaveUploadedFile(file, dst)

		c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
	})
	router.Run(":8080")
}
```

使用 `curl`发送命名:

```bash
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"
```

#### 多文件

查看参考 detail [example code](examples/upload-file/multiple).

```go
func main() {
	router := gin.Default()
	// 为multipart forms 设定最小的容量限制 (默认值为 32 MiB)
	// router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["upload[]"]

		for _, file := range files {
			log.Println(file.Filename)

			// 上传文件到指定的 dst.
			// c.SaveUploadedFile(file, dst)
		}
		c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
	})
	router.Run(":8080")
}
```

使用 `curl`发送命名:

```bash
curl -X POST http://localhost:8080/upload \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

### 组合路由

```go
func main() {
	router := gin.Default()

	// Simple group: v1
	v1 := router.Group("/v1")
	{
		v1.POST("/login", loginEndpoint)
		v1.POST("/submit", submitEndpoint)
		v1.POST("/read", readEndpoint)
	}

	// Simple group: v2
	v2 := router.Group("/v2")
	{
		v2.POST("/login", loginEndpoint)
		v2.POST("/submit", submitEndpoint)
		v2.POST("/read", readEndpoint)
	}

	router.Run(":8080")
}
```

### 默认无中间件的空白Gin

使用以下方法

```go
r := gin.New()
```

代替

```go
// Default With the Logger and Recovery middleware already attached
r := gin.Default()
```


### 使用中间件
```go
func main() {
	// 创建默认无中间件的的路由
	r := gin.New()

	// 全局中间件
	// Logger中间件将会把log写到gin.DefaultWriter中，即使你设置了GIN_MODE=release。
	// 默认gin.DefaultWriter = os.Stdout
	r.Use(gin.Logger())

	// Recovery中间件能从panic中恢复并丢出500。
	r.Use(gin.Recovery())

	// 你可以添加任意数量的路由中间件
	r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

	// 授权验证Authorization组：
	// authorized := r.Group("/", AuthRequired())
	// 另一种使用方法:
	authorized := r.Group("/")
	// per group middleware! in this case we use the custom created
	// 中间件AuthRequired()就在"authorized"组中。
	authorized.Use(AuthRequired())
	{
		authorized.POST("/login", loginEndpoint)
		authorized.POST("/submit", submitEndpoint)
		authorized.POST("/read", readEndpoint)

		// 嵌入组
		testing := authorized.Group("testing")
		testing.GET("/analytics", analyticsEndpoint)
	}

	// 监听及服务于 0.0.0.0:8080
	r.Run(":8080")
}
```

### 如何写下log文件
```go
func main() {
    // 禁用控制台颜色，你在写log到文件过程中不需要使用控制台颜色。
    gin.DisableConsoleColor()

    // 写log到文件中
    f, _ := os.Create("gin.log")
    gin.DefaultWriter = io.MultiWriter(f)

    // 如果你需要把log写到文件以及控制台中，使用下行代码。
    // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)

    router := gin.Default()
    router.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })

    router.Run(":8080")
}
```

### 自定义日志格式
```go
func main() {
	router := gin.New()

	// 中间件LoggerWithFormatter将会记录日志到gin.DefaultWriter中。
	// 默认gin.DefaultWriter = os.Stdout
	router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {

		// 你的自定义格式日志
		return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
				param.ClientIP,
				param.TimeStamp.Format(time.RFC1123),
				param.Method,
				param.Path,
				param.Request.Proto,
				param.StatusCode,
				param.Latency,
				param.Request.UserAgent(),
				param.ErrorMessage,
		)
	}))
	router.Use(gin.Recovery())

	router.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	router.Run(":8080")
}
```

**输出示例**
```
::1 - [Fri, 07 Dec 2018 17:04:38 JST] "GET /ping HTTP/1.1 200 122.767µs "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36" "
```

### 模型绑定及验证

为了绑定一个请求主体到一个类型中，会使用到模型绑定。我们现在已经支持绑定的有JSON,XML,YAML以及标准表单值(foo=bar&boo=baz)。

Gin使用[**go-playground/validator.v8**](https://github.com/go-playground/validator)作验证工作。查看标签使用方法的完整文档可以[点击这里](http://godoc.org/gopkg.in/go-playground/validator.v8#hdr-Baked_In_Validators_and_Tags)。

需要记住的是，你需要在所有你想要绑定的字段上设置正响应的绑定标签。例如当从JSON绑定时，设置`json:"fieldname"`。

同样的是，Gin提供两套绑定的方法：
- **Type** - Must bind
  - **Methods** - `Bind`, `BindJSON`, `BindXML`, `BindQuery`, `BindYAML`
  - **Behavior** - 这些方法在hood下使用 `MustBindWith` 。如果出现一个绑定错误，请求将会被`c.AbortWithError(400, err).SetType(ErrorTypeBind)`终止。这使得响应状态码设置为400，以及`Content-Type` 头设置成`text/plain; charset=utf-8`。需要注意的是，如果你想在这之后设置响应代码，将会发出一个警告`[GIN-debug] [WARNING] Headers were already written. Wanted to override status code 400 with 422`。如果你想要有一个更佳的控制行为，请考虑使用有相同效果的方法 `ShouldBind` 。
 
- **Type** - Should bind
  - **Methods** - `ShouldBind`, `ShouldBindJSON`, `ShouldBindXML`, `ShouldBindQuery`, `ShouldBindYAML`
  - **Behavior** - 这些方法在hood下使用 `ShouldBindWith` 。如果出现绑定错误，将会返回错误值。开发者需要合理地处理请求与错误。

当使用绑定方法时，Gin会尝试通过Content-Type 头信息推断出绑定器的依赖，如果你能确定需要绑定的对象，可以使用`MustBindWith` 或者 `ShouldBindWith`。

你也可以详细说明需要的特定字段。如果字段使用`binding:"required"` 修饰但绑定时值为空，就会返回一个错误。

```go
// 从JSON绑定
type Login struct {
	User     string `form:"user" json:"user" xml:"user"  binding:"required"`
	Password string `form:"password" json:"password" xml:"password" binding:"required"`
}

func main() {
	router := gin.Default()

	// Example for binding JSON ({"user": "manu", "password": "123"})
	router.POST("/loginJSON", func(c *gin.Context) {
		var json Login
		if err := c.ShouldBindJSON(&json); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		
		if json.User != "manu" || json.Password != "123" {
			c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
			return
		} 
		
		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})

	// Example for binding XML (
	//	<?xml version="1.0" encoding="UTF-8"?>
	//	<root>
	//		<user>user</user>
	//		<password>123</password>
	//	</root>)
	router.POST("/loginXML", func(c *gin.Context) {
		var xml Login
		if err := c.ShouldBindXML(&xml); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		
		if xml.User != "manu" || xml.Password != "123" {
			c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
			return
		} 
		
		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})

	// Example for binding a HTML form (user=manu&password=123)
	router.POST("/loginForm", func(c *gin.Context) {
		var form Login
		// This will infer what binder to use depending on the content-type header.
		if err := c.ShouldBind(&form); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		
		if form.User != "manu" || form.Password != "123" {
			c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
			return
		} 
		
		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})

	// Listen and serve on 0.0.0.0:8080
	router.Run(":8080")
}
```

**请求示例**
```shell
$ curl -v POST \
  http://localhost:8080/loginJSON \
  -H 'content-type: application/json' \
  -d '{ "user": "manu" }'
> POST /loginJSON HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.51.0
> Accept: */*
> content-type: application/json
> Content-Length: 18
>
* upload completely sent off: 18 out of 18 bytes
< HTTP/1.1 400 Bad Request
< Content-Type: application/json; charset=utf-8
< Date: Fri, 04 Aug 2017 03:51:31 GMT
< Content-Length: 100
<
{"error":"Key: 'Login.Password' Error:Field validation for 'Password' failed on the 'required' tag"}
```

**跳过验证**

当运行上述的例子使用`curl` 命令时，将会返回错误。因为示例中`Password`.字段使用了`binding:"required"` 。如果`Password`.字段使用`binding:"-"` ，上述例子将不会返回错误。

### 自定义验证器

可以注册使用自定义验证器，参考[示例代码](%28examples/custom-validation/server.go%29.)。

[embedmd]:# (examples/custom-validation/server.go go)
```go
package main

import (
	"net/http"
	"reflect"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"gopkg.in/go-playground/validator.v8"
)

// Booking contains binded and validated data.
type Booking struct {
	CheckIn  time.Time `form:"check_in" binding:"required,bookabledate" time_format:"2006-01-02"`
	CheckOut time.Time `form:"check_out" binding:"required,gtfield=CheckIn" time_format:"2006-01-02"`
}

func bookableDate(
	v *validator.Validate, topStruct reflect.Value, currentStructOrField reflect.Value,
	field reflect.Value, fieldType reflect.Type, fieldKind reflect.Kind, param string,
) bool {
	if date, ok := field.Interface().(time.Time); ok {
		today := time.Now()
		if today.Year() > date.Year() || today.YearDay() > date.YearDay() {
			return false
		}
	}
	return true
}

func main() {
	route := gin.Default()

	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		v.RegisterValidation("bookabledate", bookableDate)
	}

	route.GET("/bookable", getBookable)
	route.Run(":8085")
}

func getBookable(c *gin.Context) {
	var b Booking
	if err := c.ShouldBindWith(&b, binding.Query); err == nil {
		c.JSON(http.StatusOK, gin.H{"message": "Booking dates are valid!"})
	} else {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
	}
}
```

```console
$ curl "localhost:8085/bookable?check_in=2018-04-16&check_out=2018-04-17"
{"message":"Booking dates are valid!"}

$ curl "localhost:8085/bookable?check_in=2018-03-08&check_out=2018-03-09"
{"error":"Key: 'Booking.CheckIn' Error:Field validation for 'CheckIn' failed on the 'bookabledate' tag"}
```

[结构体级别验证](https://github.com/go-playground/validator/releases/tag/v8.7)也可以使用这种方法注册。
查看 [struct-lvl-validation example](examples/struct-lvl-validations) 学到更多。

### 只绑定查询字符串

`ShouldBindQuery` 函数只绑定查询参数而没有post数据，参考 [详细信息](https://github.com/gin-gonic/gin/issues/742#issuecomment-315953017).

```go
package main

import (
	"log"

	"github.com/gin-gonic/gin"
)

type Person struct {
	Name    string `form:"name"`
	Address string `form:"address"`
}

func main() {
	route := gin.Default()
	route.Any("/testing", startPage)
	route.Run(":8085")
}

func startPage(c *gin.Context) {
	var person Person
	if c.ShouldBindQuery(&person) == nil {
		log.Println("====== Only Bind By Query String ======")
		log.Println(person.Name)
		log.Println(person.Address)
	}
	c.String(200, "Success")
}

```

### 绑定字符串或Post数据

参考 [详细信息](https://github.com/gin-gonic/gin/issues/742#issuecomment-264681292).

```go
package main

import (
	"log"
	"time"

	"github.com/gin-gonic/gin"
)

type Person struct {
	Name     string    `form:"name"`
	Address  string    `form:"address"`
	Birthday time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
}

func main() {
	route := gin.Default()
	route.GET("/testing", startPage)
	route.Run(":8085")
}

func startPage(c *gin.Context) {
	var person Person
	// If `GET`, only `Form` binding engine (`query`) used.
	// If `POST`, first checks the `content-type` for `JSON` or `XML`, then uses `Form` (`form-data`).
	// See more at https://github.com/gin-gonic/gin/blob/master/binding/binding.go#L48
	if c.ShouldBind(&person) == nil {
		log.Println(person.Name)
		log.Println(person.Address)
		log.Println(person.Birthday)
	}

	c.String(200, "Success")
}
```

Test it with:
```sh
$ curl -X GET "localhost:8085/testing?name=appleboy&address=xyz&birthday=1992-03-15"
```

### 绑定Uri

参考 [详细信息](https://github.com/gin-gonic/gin/issues/846).

```go
package main

import "github.com/gin-gonic/gin"

type Person struct {
	ID string `uri:"id" binding:"required,uuid"`
	Name string `uri:"name" binding:"required"`
}

func main() {
	route := gin.Default()
	route.GET("/:name/:id", func(c *gin.Context) {
		var person Person
		if err := c.ShouldBindUri(&person); err != nil {
			c.JSON(400, gin.H{"msg": err})
			return
		}
		c.JSON(200, gin.H{"name": person.Name, "uuid": person.ID})
	})
	route.Run(":8088")
}
```

Test it with:
```sh
$ curl -v localhost:8088/thinkerou/987fbc97-4bed-5078-9f07-9141ba07c9f3
$ curl -v localhost:8088/thinkerou/not-uuid
```

### 绑定 HTML 复选框

参考 [详细信息](https://github.com/gin-gonic/gin/issues/129#issuecomment-124260092)

main.go

```go
...

type myForm struct {
    Colors []string `form:"colors[]"`
}

...

func formHandler(c *gin.Context) {
    var fakeForm myForm
    c.ShouldBind(&fakeForm)
    c.JSON(200, gin.H{"color": fakeForm.Colors})
}

...

```

form.html

```html
<form action="/" method="POST">
    <p>Check some colors</p>
    <label for="red">Red</label>
    <input type="checkbox" name="colors[]" value="red" id="red">
    <label for="green">Green</label>
    <input type="checkbox" name="colors[]" value="green" id="green">
    <label for="blue">Blue</label>
    <input type="checkbox" name="colors[]" value="blue" id="blue">
    <input type="submit">
</form>
```

结果:

```
{"color":["red","green","blue"]}
```

### Multipart/Urlencoded 绑定

```go
package main

import (
	"github.com/gin-gonic/gin"
)

type LoginForm struct {
	User     string `form:"user" binding:"required"`
	Password string `form:"password" binding:"required"`
}

func main() {
	router := gin.Default()
	router.POST("/login", func(c *gin.Context) {
		// 你可以使用明确的绑定声明绑定一个multipart表单
		// c.ShouldBindWith(&form, binding.Form)
		// 又或者你可以简单地使用自动绑定方法ShouldBind：
		var form LoginForm
		// 在这个例子中会自动选择一个合适的绑定
		if c.ShouldBind(&form) == nil {
			if form.User == "user" && form.Password == "password" {
				c.JSON(200, gin.H{"status": "you are logged in"})
			} else {
				c.JSON(401, gin.H{"status": "unauthorized"})
			}
		}
	})
	router.Run(":8080")
}
```

使用以下测试:
```sh
$ curl -v --form user=user --form password=password http://localhost:8080/login
```

### XML, JSON, YAML 以及 ProtoBuf 渲染

```go
func main() {
	r := gin.Default()

	//  gin.H为map[string]interface{}的快捷方式 
	r.GET("/someJSON", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})

	r.GET("/moreJSON", func(c *gin.Context) {
		// 你也可以使用一个结构体
		var msg struct {
			Name    string `json:"user"`
			Message string
			Number  int
		}
		msg.Name = "Lena"
		msg.Message = "hey"
		msg.Number = 123
		// 需要注意是在JSON中msg.Name变成了字段"user" 
		// 将会输出  :   {"user": "Lena", "Message": "hey", "Number": 123}
		c.JSON(http.StatusOK, msg)
	})

	r.GET("/someXML", func(c *gin.Context) {
		c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})

	r.GET("/someYAML", func(c *gin.Context) {
		c.YAML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})

	r.GET("/someProtoBuf", func(c *gin.Context) {
		reps := []int64{int64(1), int64(2)}
		label := "test"
		// 这一个protobuf特殊类型被定义在testdata/protoexample文件中
		data := &protoexample.Test{
			Label: &label,
			Reps:  reps,
		}
		// 需要注意的是数据在响应中变成了二进制数据
		// 将会输出一个proto例子，测试protobuf序列化数据
		c.ProtoBuf(http.StatusOK, data)
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

#### 安全JSON

使用安全JSON用于防止JSON劫持。如果给定的是数组值，将会设定一个预设值`"while(1),"` 


```go
func main() {
	r := gin.Default()

	// 你也可以使用你的自己的安全JSON前缀
	// r.SecureJsonPrefix(")]}',\n")

	r.GET("/someJSON", func(c *gin.Context) {
		names := []string{"lena", "austin", "foo"}

		// 将会输出  :   while(1);["lena","austin","foo"]
		c.SecureJSON(http.StatusOK, names)
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```
#### JSONP

使用JSONP向不同域的服务器请求数据.如果查询参数中存在回调函数,则需要增加回调函数到响应主体中.

```go
func main() {
	r := gin.Default()

	r.GET("/JSONP?callback=x", func(c *gin.Context) {
		data := map[string]interface{}{
			"foo": "bar",
		}
		
		//callback is x
		// Will output  :   x({\"foo\":\"bar\"})
		c.JSONP(http.StatusOK, data)
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

#### AsciiJSON

使用 非 ASCII 字符组成的AsciiJSON 生成仅有 ASCII 字符的 JSON.

```go
func main() {
	r := gin.Default()

	r.GET("/someJSON", func(c *gin.Context) {
		data := map[string]interface{}{
			"lang": "GO语言",
			"tag":  "<br>",
		}

		// 将输出: {"lang":"GO\u8bed\u8a00","tag":"\u003cbr\u003e"}
		c.AsciiJSON(http.StatusOK, data)
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

#### PureJSON
通常,JSON使用unicode字符去取代特殊的HTML字符,比如:`<` 会变成`\u003c`.如果你想编码成这种字符文字,你可以使用PureJSON, 该特性在Go 1.6及以下版本是不支持的.

```go
func main() {
	r := gin.Default()
	
	// 服务于unicode字符
	r.GET("/json", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"html": "<b>Hello, world!</b>",
		})
	})
	
	// 服务于文字字符
	r.GET("/purejson", func(c *gin.Context) {
		c.PureJSON(200, gin.H{
			"html": "<b>Hello, world!</b>",
		})
	})
	
	// listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

### 静态文件服务

```go
func main() {
	router := gin.Default()
	router.Static("/assets", "./assets")
	router.StaticFS("/more_static", http.Dir("my_file_system"))
	router.StaticFile("/favicon.ico", "./resources/favicon.ico")

	// Listen and serve on 0.0.0.0:8080
	router.Run(":8080")
}
```

### 在阅读程序中提供数据服务

```go
func main() {
	router := gin.Default()
	router.GET("/someDataFromReader", func(c *gin.Context) {
		response, err := http.Get("https://raw.githubusercontent.com/gin-gonic/logo/master/color.png")
		if err != nil || response.StatusCode != http.StatusOK {
			c.Status(http.StatusServiceUnavailable)
			return
		}

		reader := response.Body
		contentLength := response.ContentLength
		contentType := response.Header.Get("Content-Type")

		extraHeaders := map[string]string{
			"Content-Disposition": `attachment; filename="gopher.png"`,
		}

		c.DataFromReader(http.StatusOK, contentLength, contentType, reader, extraHeaders)
	})
	router.Run(":8080")
}
```

### HTML 渲染

可使用 LoadHTMLGlob() 或者 LoadHTMLFiles()

```go
func main() {
	router := gin.Default()
	router.LoadHTMLGlob("templates/*")
	//router.LoadHTMLFiles("templates/template1.html", "templates/template2.html")
	router.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.tmpl", gin.H{
			"title": "Main website",
		})
	})
	router.Run(":8080")
}
```

templates/index.tmpl

```html
<html>
	<h1>
		{{ .title }}
	</h1>
</html>
```

在不同路径下使用同名的 templates

```go
func main() {
	router := gin.Default()
	router.LoadHTMLGlob("templates/**/*")
	router.GET("/posts/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "posts/index.tmpl", gin.H{
			"title": "Posts",
		})
	})
	router.GET("/users/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "users/index.tmpl", gin.H{
			"title": "Users",
		})
	})
	router.Run(":8080")
}
```

templates/posts/index.tmpl

```html
{{ define "posts/index.tmpl" }}
<html><h1>
	{{ .title }}
</h1>
<p>Using posts/index.tmpl</p>
</html>
{{ end }}
```

templates/users/index.tmpl

```html
{{ define "users/index.tmpl" }}
<html><h1>
	{{ .title }}
</h1>
<p>Using users/index.tmpl</p>
</html>
{{ end }}
```

#### 自定义模板渲染

你可以使用自定义的html模板渲染

```go
import "html/template"

func main() {
	router := gin.Default()
	html := template.Must(template.ParseFiles("file1", "file2"))
	router.SetHTMLTemplate(html)
	router.Run(":8080")
}
```

#### 自定义定界符

你可以使用自定义定界符

```go
	r := gin.Default()
	r.Delims("{[{", "}]}")
	r.LoadHTMLGlob("/path/to/templates")
```

#### 自定义模板函数

查看详细 [示例代码](examples/template).

main.go

```go
import (
    "fmt"
    "html/template"
    "net/http"
    "time"

    "github.com/gin-gonic/gin"
)

func formatAsDate(t time.Time) string {
    year, month, day := t.Date()
    return fmt.Sprintf("%d%02d/%02d", year, month, day)
}

func main() {
    router := gin.Default()
    router.Delims("{[{", "}]}")
    router.SetFuncMap(template.FuncMap{
        "formatAsDate": formatAsDate,
    })
    router.LoadHTMLFiles("./testdata/template/raw.tmpl")

    router.GET("/raw", func(c *gin.Context) {
        c.HTML(http.StatusOK, "raw.tmpl", map[string]interface{}{
            "now": time.Date(2017, 07, 01, 0, 0, 0, 0, time.UTC),
        })
    })

    router.Run(":8080")
}

```

raw.tmpl

```html
Date: {[{.now | formatAsDate}]}
```

Result:
```
Date: 2017/07/01
```

### 多模板

Gin默认允许使用一个HTML模板.参考[多模板渲染](https://github.com/gin-contrib/multitemplate) 来使用多模板特性如go 1.6 `block template`.

### 重定向

发起一个HTTP重定向非常地简单。同时支持内置与外部地址。

```go
r.GET("/test", func(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
})
```


如发起一个路由器重定向，如下例中使用 `HandleContext` 。

``` go
r.GET("/test", func(c *gin.Context) {
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
})
r.GET("/test2", func(c *gin.Context) {
    c.JSON(200, gin.H{"hello": "world"})
})
```


### 自定义中间件

```go
func Logger() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()

		// 设定参数example
		c.Set("example", "12345")

		// 请求前

		c.Next()

		// 请求后
		latency := time.Since(t)
		log.Print(latency)

		// 访问我们发送的状态status
		status := c.Writer.Status()
		log.Println(status)
	}
}

func main() {
	r := gin.New()
	r.Use(Logger())

	r.GET("/test", func(c *gin.Context) {
		example := c.MustGet("example").(string)

		// 将会打印： "12345"
		log.Println(example)
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

### 使用 BasicAuth() 中间件

```go
// 使用仿真隐私数据
var secrets = gin.H{
	"foo":    gin.H{"email": "foo@bar.com", "phone": "123433"},
	"austin": gin.H{"email": "austin@example.com", "phone": "666"},
	"lena":   gin.H{"email": "lena@guapa.com", "phone": "523443"},
}

func main() {
	r := gin.Default()

	// Group中使用gin.BasicAuth()作中间件
	// gin.Accounts 是 map[string]string 的快捷方式
	authorized := r.Group("/admin", gin.BasicAuth(gin.Accounts{
		"foo":    "bar",
		"austin": "1234",
		"lena":   "hello2",
		"manu":   "4321",
	}))

	// /admin/secrets endpoint
	// hit "localhost:8080/admin/secrets
	authorized.GET("/secrets", func(c *gin.Context) {
		// get user, it was set by the BasicAuth middleware
		user := c.MustGet(gin.AuthUserKey).(string)
		if secret, ok := secrets[user]; ok {
			c.JSON(http.StatusOK, gin.H{"user": user, "secret": secret})
		} else {
			c.JSON(http.StatusOK, gin.H{"user": user, "secret": "NO SECRET :("})
		}
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

### 中间件中的Goroutines

当在中间件或者处理函数中使用Goroutines时，你**不能**在其内部使用原始的上下文，而应该使用一个只读的副本。

```go
func main() {
	r := gin.Default()

	r.GET("/long_async", func(c *gin.Context) {
		// 创建一个在goroutine中使用的副本
		cCp := c.Copy()
		go func() {
			// 使用time.Sleep()仿真一个5秒种的长时间任务
			time.Sleep(5 * time.Second)

			// 需要注意的是你需要使用上下文的副本"cCp"，非常重要！
			log.Println("Done! in path " + cCp.Request.URL.Path)
		}()
	})

	r.GET("/long_sync", func(c *gin.Context) {
		// 使用time.Sleep()仿真一个5秒种的长时间任务
		time.Sleep(5 * time.Second)

		// 因为没有使用goroutine，我们不需要上下文的副本。
		log.Println("Done! in path " + c.Request.URL.Path)
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

### 自定义HTTP配置

如下，直接使用`http.ListenAndServe()` ：

```go
func main() {
	router := gin.Default()
	http.ListenAndServe(":8080", router)
}
```
or

```go
func main() {
	router := gin.Default()

	s := &http.Server{
		Addr:           ":8080",
		Handler:        router,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	s.ListenAndServe()
}
```

### 支持 Let's Encrypt (小绿锁)

使用一键 LetsEncrypt 搭建 HTTPS 服务器的例子。

```
package main

import (
	"log"

	"github.com/gin-gonic/autotls"
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	// Ping handler
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	log.Fatal(autotls.Run(r, "example1.com", "example2.com"))
}
```

自定义autocert管理器的例子。

[embedmd]:# (examples/auto-tls/example2/main.go go)
```go
package main

import (
	"log"

	"github.com/gin-gonic/autotls"
	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/acme/autocert"
)

func main() {
	r := gin.Default()

	// Ping handler
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})

	m := autocert.Manager{
		Prompt:     autocert.AcceptTOS,
		HostPolicy: autocert.HostWhitelist("example1.com", "example2.com"),
		Cache:      autocert.DirCache("/var/www/.cache"),
	}

	log.Fatal(autotls.RunWithManager(r, &m))
}
```

### 使用Gin运行多个服务

可以参考这一个[问题](https://github.com/gin-gonic/gin/issues/346)以及实验以下例子：

[embedmd]:# (examples/multiple-service/main.go go)
```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	"golang.org/x/sync/errgroup"
)

var (
	g errgroup.Group
)

func router01() http.Handler {
	e := gin.New()
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 01",
			},
		)
	})

	return e
}

func router02() http.Handler {
	e := gin.New()
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 02",
			},
		)
	})

	return e
}

func main() {
	server01 := &http.Server{
		Addr:         ":8080",
		Handler:      router01(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}

	server02 := &http.Server{
		Addr:         ":8081",
		Handler:      router02(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}

	g.Go(func() error {
		return server01.ListenAndServe()
	})

	g.Go(func() error {
		return server02.ListenAndServe()
	})

	if err := g.Wait(); err != nil {
		log.Fatal(err)
	}
}
```

### 优雅地重启或停止

你希望能够优雅地重启或停止你的web服务器吗？
有一些方法可以做到这一点。

我们可以使用 [fvbock/endless](https://github.com/fvbock/endless) 取代默认的`ListenAndServe`。参考 issue [#296](https://github.com/gin-gonic/gin/issues/296) 可以获得更多细节。

```go
router := gin.Default()
router.GET("/", handler)
// [...]
endless.ListenAndServe(":4242", router)
```

使用endless的多种可供方法：

* [manners](https://github.com/braintree/manners): 一个优雅的GO HTTP服务器提供优雅地关闭方法。
* [graceful](https://github.com/tylerb/graceful): Graceful是一个Go包能使http.Handler服务器优雅地关闭。
* [grace](https://github.com/facebookgo/grace): 优雅地重启以及零停机部署Go服务器。

如果你使用Go 1.8，你可能不需要用到这个库！参考使用http.Server的内建 [Shutdown()](https://golang.org/pkg/net/http/#Server.Shutdown) 方法提供优雅的关机方式。查看Gin上全部的[graceful-shutdown](./examples/graceful-shutdown) 例子。

[embedmd]:# (examples/graceful-shutdown/graceful-shutdown/server.go go)
```go
// +build go1.8

package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"time"

	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.GET("/", func(c *gin.Context) {
		time.Sleep(5 * time.Second)
		c.String(http.StatusOK, "Welcome Gin Server")
	})

	srv := &http.Server{
		Addr:    ":8080",
		Handler: router,
	}

	go func() {
		// service connections
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("listen: %s\n", err)
		}
	}()

	// Wait for interrupt signal to gracefully shutdown the server with
	// a timeout of 5 seconds.
	quit := make(chan os.Signal)
	signal.Notify(quit, os.Interrupt)
	<-quit
	log.Println("Shutdown Server ...")

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	if err := srv.Shutdown(ctx); err != nil {
		log.Fatal("Server Shutdown:", err)
	}
	log.Println("Server exiting")
}
```

### Build a single binary with templates

You can build a server into a single binary containing templates by using [go-assets][].

[go-assets]: https://github.com/jessevdk/go-assets

```go
func main() {
	r := gin.New()

	t, err := loadTemplate()
	if err != nil {
		panic(err)
	}
	r.SetHTMLTemplate(t)

	r.GET("/", func(c *gin.Context) {
		c.HTML(http.StatusOK, "/html/index.tmpl",nil)
	})
	r.Run(":8080")
}

// loadTemplate loads templates embedded by go-assets-builder
func loadTemplate() (*template.Template, error) {
	t := template.New("")
	for name, file := range Assets.Files {
		if file.IsDir() || !strings.HasSuffix(name, ".tmpl") {
			continue
		}
		h, err := ioutil.ReadAll(file)
		if err != nil {
			return nil, err
		}
		t, err = t.New(name).Parse(string(h))
		if err != nil {
			return nil, err
		}
	}
	return t, nil
}
```

See a complete example in the `examples/assets-in-binary` directory.

### Bind form-data request with custom struct

The follow example using custom struct:

```go
type StructA struct {
    FieldA string `form:"field_a"`
}

type StructB struct {
    NestedStruct StructA
    FieldB string `form:"field_b"`
}

type StructC struct {
    NestedStructPointer *StructA
    FieldC string `form:"field_c"`
}

type StructD struct {
    NestedAnonyStruct struct {
        FieldX string `form:"field_x"`
    }
    FieldD string `form:"field_d"`
}

func GetDataB(c *gin.Context) {
    var b StructB
    c.Bind(&b)
    c.JSON(200, gin.H{
        "a": b.NestedStruct,
        "b": b.FieldB,
    })
}

func GetDataC(c *gin.Context) {
    var b StructC
    c.Bind(&b)
    c.JSON(200, gin.H{
        "a": b.NestedStructPointer,
        "c": b.FieldC,
    })
}

func GetDataD(c *gin.Context) {
    var b StructD
    c.Bind(&b)
    c.JSON(200, gin.H{
        "x": b.NestedAnonyStruct,
        "d": b.FieldD,
    })
}

func main() {
    r := gin.Default()
    r.GET("/getb", GetDataB)
    r.GET("/getc", GetDataC)
    r.GET("/getd", GetDataD)

    r.Run()
}
```

Using the command `curl` command result:

```
$ curl "http://localhost:8080/getb?field_a=hello&field_b=world"
{"a":{"FieldA":"hello"},"b":"world"}
$ curl "http://localhost:8080/getc?field_a=hello&field_c=world"
{"a":{"FieldA":"hello"},"c":"world"}
$ curl "http://localhost:8080/getd?field_x=hello&field_d=world"
{"d":"world","x":{"FieldX":"hello"}}
```

**NOTE**: NOT support the follow style struct:

```go
type StructX struct {
    X struct {} `form:"name_x"` // HERE have form
}

type StructY struct {
    Y StructX `form:"name_y"` // HERE have form
}

type StructZ struct {
    Z *StructZ `form:"name_z"` // HERE have form
}
```

In a word, only support nested custom struct which have no `form` now.

### Try to bind body into different structs

The normal methods for binding request body consumes `c.Request.Body` and they
cannot be called multiple times.

```go
type formA struct {
  Foo string `json:"foo" xml:"foo" binding:"required"`
}

type formB struct {
  Bar string `json:"bar" xml:"bar" binding:"required"`
}

func SomeHandler(c *gin.Context) {
  objA := formA{}
  objB := formB{}
  // This c.ShouldBind consumes c.Request.Body and it cannot be reused.
  if errA := c.ShouldBind(&objA); errA == nil {
    c.String(http.StatusOK, `the body should be formA`)
  // Always an error is occurred by this because c.Request.Body is EOF now.
  } else if errB := c.ShouldBind(&objB); errB == nil {
    c.String(http.StatusOK, `the body should be formB`)
  } else {
    ...
  }
}
```

For this, you can use `c.ShouldBindBodyWith`.

```go
func SomeHandler(c *gin.Context) {
  objA := formA{}
  objB := formB{}
  // This reads c.Request.Body and stores the result into the context.
  if errA := c.ShouldBindBodyWith(&objA, binding.JSON); errA == nil {
    c.String(http.StatusOK, `the body should be formA`)
  // At this time, it reuses body stored in the context.
  } else if errB := c.ShouldBindBodyWith(&objB, binding.JSON); errB == nil {
    c.String(http.StatusOK, `the body should be formB JSON`)
  // And it can accepts other formats
  } else if errB2 := c.ShouldBindBodyWith(&objB, binding.XML); errB2 == nil {
    c.String(http.StatusOK, `the body should be formB XML`)
  } else {
    ...
  }
}
```

* `c.ShouldBindBodyWith` stores body into the context before binding. This has
a slight impact to performance, so you should not use this method if you are
enough to call binding at once.
* This feature is only needed for some formats -- `JSON`, `XML`, `MsgPack`,
`ProtoBuf`. For other formats, `Query`, `Form`, `FormPost`, `FormMultipart`,
can be called by `c.ShouldBind()` multiple times without any damage to
performance (See [#1341](https://github.com/gin-gonic/gin/pull/1341)).

### http2 server push

http.Pusher is supported only **go1.8+**. See the [golang blog](https://blog.golang.org/h2push) for detail information.

[embedmd]:# (examples/http-pusher/main.go go)
```go
package main

import (
	"html/template"
	"log"

	"github.com/gin-gonic/gin"
)

var html = template.Must(template.New("https").Parse(`
<html>
<head>
  <title>Https Test</title>
  <script src="/assets/app.js"></script>
</head>
<body>
  <h1 style="color:red;">Welcome, Ginner!</h1>
</body>
</html>
`))

func main() {
	r := gin.Default()
	r.Static("/assets", "./assets")
	r.SetHTMLTemplate(html)

	r.GET("/", func(c *gin.Context) {
		if pusher := c.Writer.Pusher(); pusher != nil {
			// use pusher.Push() to do server push
			if err := pusher.Push("/assets/app.js", nil); err != nil {
				log.Printf("Failed to push: %v", err)
			}
		}
		c.HTML(200, "https", gin.H{
			"status": "success",
		})
	})

	// Listen and Server in https://127.0.0.1:8080
	r.RunTLS(":8080", "./testdata/server.pem", "./testdata/server.key")
}
```

### Define format for the log of routes

The default log of routes is:
```
[GIN-debug] POST   /foo                      --> main.main.func1 (3 handlers)
[GIN-debug] GET    /bar                      --> main.main.func2 (3 handlers)
[GIN-debug] GET    /status                   --> main.main.func3 (3 handlers)
```

If you want to log this information in given format (e.g. JSON, key values or something else), then you can define this format with `gin.DebugPrintRouteFunc`.
In the example below, we log all routes with standard log package but you can use another log tools that suits of your needs.
```go
import (
	"log"
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	gin.DebugPrintRouteFunc = func(httpMethod, absolutePath, handlerName string, nuHandlers int) {
		log.Printf("endpoint %v %v %v %v\n", httpMethod, absolutePath, handlerName, nuHandlers)
	}

	r.POST("/foo", func(c *gin.Context) {
		c.JSON(http.StatusOK, "foo")
	})

	r.GET("/bar", func(c *gin.Context) {
		c.JSON(http.StatusOK, "bar")
	})

	r.GET("/status", func(c *gin.Context) {
		c.JSON(http.StatusOK, "ok")
	})

	// Listen and Server in http://0.0.0.0:8080
	r.Run()
}
```

### Set and get a cookie

```go
import (
    "fmt"

    "github.com/gin-gonic/gin"
)

func main() {

    router := gin.Default()

    router.GET("/cookie", func(c *gin.Context) {

        cookie, err := c.Cookie("gin_cookie")

        if err != nil {
            cookie = "NotSet"
            c.SetCookie("gin_cookie", "test", 3600, "/", "localhost", false, true)
        }

        fmt.Printf("Cookie value: %s \n", cookie)
    })

    router.Run()
}
```


## Testing

The `net/http/httptest` package is preferable way for HTTP testing.

```go
package main

func setupRouter() *gin.Engine {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong")
	})
	return r
}

func main() {
	r := setupRouter()
	r.Run(":8080")
}
```

Test for code example above:

```go
package main

import (
	"net/http"
	"net/http/httptest"
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestPingRoute(t *testing.T) {
	router := setupRouter()

	w := httptest.NewRecorder()
	req, _ := http.NewRequest("GET", "/ping", nil)
	router.ServeHTTP(w, req)

	assert.Equal(t, 200, w.Code)
	assert.Equal(t, "pong", w.Body.String())
}
```

## Users

Awesome project lists using [Gin](https://github.com/gin-gonic/gin) web framework.

* [drone](https://github.com/drone/drone): Drone is a Continuous Delivery platform built on Docker, written in Go.
* [gorush](https://github.com/appleboy/gorush): A push notification server written in Go.
* [fnproject](https://github.com/fnproject/fn): The container native, cloud agnostic serverless platform.
* [photoprism](https://github.com/photoprism/photoprism): Personal photo management powered by Go and Google TensorFlow.
* [krakend](https://github.com/devopsfaith/krakend): Ultra performant API Gateway with middlewares.
* [picfit](https://github.com/thoas/picfit): An image resizing server written in Go.
