# go-assets中文文档

此文档翻译于go-assets官方README.md，如需查看最新的更新文档以及源代码，请移步[官方Github](https://github.com/jessevdk/go-assets)。
go-assets是go语言的一个简单的嵌入式资源生成器和使用者库。本库主要用途是生成与嵌入到将被整合于web服务器或其他服务中的小型内存文件系统中，这些服务器中有着小批量的资源需要在运行中被使用到。使用资源进行单二进制部署是非常棒的体验。

[Generator](http://godoc.org/github.com/jessevdk/go-assets#Generator)类型可以被用于生成一个go文件，文件中包含着一个来自文件和目录中，在硬盘上的内存文件。文件数据可以被选择使用gzip压缩以达到减小体积。然后，生成的文件可以被包含在你的应用程序用，且这些资源可以被直接地访问无需读取到硬盘上。被生成的资源变量来自于[FileSystem](http://godoc.org/github.com/jessevdk/go-assets#FileSystem)类型，且被执行在os.FileInfo 与 http.FileSystem接口中，所以可以被http.FileHandler
直接地访问。

同样地可以参考[go-assets-builder](https://github.com/jessevdk/go-assets-builder)，一个简单的生成器程序，使用 go-assets和以命令行应用程序方式展露出生成器。

参考 <http://godoc.org/github.com/jessevdk/go-assets>可获得更多信息。

