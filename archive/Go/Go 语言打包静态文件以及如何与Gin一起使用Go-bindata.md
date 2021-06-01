# Go| Go 语言打包静态文件以及如何与Gin一起使用Go-bindata


# 前言

前几天，开始学习用 Go 语言开发一个内部项目来帮助解决测试环境中的一些不便利的问题。因为开发的小项目中存在一些静态文件和配置文件，第一打包的时候发现并没有将静态文件打包进入可执行文件，这样在发布的时候又需要手动拷贝一下静态文件，这样就很麻烦了。在一些网上资料上发现了 `go-bindata`。这样我还是只分发一个文件就可以了，不过打包出来的文件会大一点。

# 一、go-bindata是什么？
his package converts any file into managable Go source code. Useful for embedding binary data into a go program. The file data is optionally gzip compressed before being converted to a raw byte slice.

It comes with a command line tool in the go-bindata sub directory. This tool offers a set of command line options, used to customize the output being generated.

go-bindata 将任何文件封装在一个 Go 语言的 Source Code 里面，文件数据在转换为原始字节时可以选择使用 gzip 压缩，同时提供了统一的接口，帮助获取原始的文件数据

# 二、使用步骤
## 1. 安装
```go
go get -u github.com/jteeuwen/go-bindata/...
```

## 2. 使用
使用 `go-bindata --help` 可以查看具体的使用方式

```shell
go-bindata --help
Usage: go-bindata [options] <input directories>

  -debug
        Do not embed the assets, but provide the embedding API. Contents will still be loaded from disk.
  -dev
        Similar to debug, but does not emit absolute paths. Expects a rootDir variable to already exist in the generated code's package.
  -fs
        Whether generate instance http.FileSystem interface code.
  -ignore value
        Regex pattern to ignore
  -mode uint
        Optional file mode override for all files.
  -modtime int
        Optional modification unix timestamp override for all files.
  -nocompress
        Assets will *not* be GZIP compressed when this flag is specified.
  -nomemcopy
        Use a .rodata hack to get rid of unnecessary memcopies. Refer to the documentation to see what implications this carries.
  -nometadata
        Assets will not preserve size, mode, and modtime info.
  -o string
        Optional name of the output file to be generated. (default "./bindata.go")
  -pkg string
        Package name to use in the generated code. (default "main")
  -prefix string
        Optional path prefix to strip off asset names.
  -tags string
        Optional set of build tags to include.
  -version
        Displays version information.

```

**这里项目的大概结构如下**


```
├── conf
│   └── app.ini
├── main.go
├── routers
│   └──  router.go
├── setting
│   └── setting.go
└── template
    ├── css
    │   └── chunk-vendors.c6d02872.css
    ├── favicon.ico
    ├── fonts
    │   └──  element-icons.535877f5.woff
    ├── img
    │   └──  Avatar.41ba4b7a.jpg
    ├── index.html
    └── js
        ├── app.40872d4f.js
```

> 为了查看方便和限于篇幅，删除了一些无关的文件

在打包的时候想将 `/conf` 和 `/template`，打包进生成的文件中

最基本的使用方式是 `go-bindata <input directories>` 这里我们参数全部使用默认的，这样将创建 `bindata.go` 且生成的文件放在根目录的 main 包下

```go
go-bindata template/... conf/...
```

如果要修改最终生成文件名和包名可以使用 `-o` 和 `-pkg` 参数，这样就在 asset 下生成 asset.go 文件且包为 asset

```go
go-bindata -o=asset/asset.go -pkg=asset template/... conf/...
```

## 3. 读取文件
通过上述方式生成的 asset.go 文件中包含 3 个获取文件信息的方法

- func Asset(name string) ([]byte, error) 根据文件名加载文件返回 byte
- func AssetNames() []string 返回所有的文件列表
- func AssetInfo(name string) (os.FileInfo, error) 返回文件信息

如果你查看源文件，可以查看 `_bindata` 中维护了文件的信息

```go
var _bindata = map[string]func() (*asset, error){
	"template/css/app.ee8ee5dd.css":              templateCssAppEe8ee5ddCss,
	"template/favicon.ico":                       templateFaviconIco,
	"template/fonts/element-icons.535877f5.woff": templateFontsElementIcons535877f5Woff,
	"template/img/Avatar.41ba4b7a.jpg":           templateImgAvatar41ba4b7aJpg,
	"template/index.html":                        templateIndexHtml,
	"template/js/app.40872d4f.js":                templateJsApp40872d4fJs,
	"conf/app.ini":                               confAppIni,
}
```

如果我们想读取 conf/app.ini 的文件，可以用使用

```go
conf_ini, _ := asset.Asset("conf/app.ini")
```

这样简单的操作就完成了

# 三、和 Gin 一起使用

在正常使用 Gin 时，我们一般这样配置静态资源的使用

```go
r := gin.Default()

r.LoadHTMLGlob("template/index.html")
r.Static("/css", "./template/css")
r.Static("/fonts", "./template/fonts")
r.Static("/img", "./template/img")
r.Static("/js", "./template/js")
r.StaticFile("/favicon.ico", "./template/favicon.ico")

// 首页
r.GET("/", func(c *gin.Context) {
	c.HTML(http.StatusOK, "index.html", nil)
})
```

为了将使用打包好的静态资源，我们需要换种方式进行打包和配置

## 1. 使用 go-bindata-assetfs 进行打包
Serve embedded files from `go-bindata` with net/http.

go-bindata-assetfs 实现了 http.FileSystem 帮助我们更好的在 http 服务上使用生成的文件

## 2. 安装 go-bindata-assetfs
这个需要和 `go-bindata` 一起安装，如果已经安装了  `go-bindata`  则不需要再次安装

```go
go get github.com/go-bindata/go-bindata/...
go get github.com/elazarl/go-bindata-assetfs/...
```

## 3. 打包文件

这里我们只需将 `go-bindata` 换成 ` go-bindata-assetfs` 即可，两者的使用方式相同

```go
go-bindata-assetfs -o=asset/asset.go -pkg=asset template/... conf/...
```

## 4. 重新配置
```go
r := gin.Default()

fsCss := assetfs.AssetFS{Asset: asset.Asset, AssetDir: asset.AssetDir, AssetInfo: asset.AssetInfo, Prefix: "dist/css", Fallback: "index.html"}
fsFonts := assetfs.AssetFS{Asset: asset.Asset, AssetDir: asset.AssetDir, AssetInfo: asset.AssetInfo, Prefix: "dist/fonts", Fallback: "index.html"}
fsImg := assetfs.AssetFS{Asset: asset.Asset, AssetDir: asset.AssetDir, AssetInfo: asset.AssetInfo, Prefix: "dist/img", Fallback: "index.html"}
fsJs := assetfs.AssetFS{Asset: asset.Asset, AssetDir: asset.AssetDir, AssetInfo: asset.AssetInfo, Prefix: "dist/js", Fallback: "index.html"}
fs := assetfs.AssetFS{Asset: asset.Asset, AssetDir: asset.AssetDir, AssetInfo: asset.AssetInfo, Prefix: "dist", Fallback: "index.html"}

r.StaticFS("/css", &fsCss)
r.StaticFS("/fonts", &fsFonts)
r.StaticFS("/img", &fsImg)
r.StaticFS("/js", &fsJs)
r.StaticFS("/favicon.ico", &fs)

r.GET("/", func(c *gin.Context) {
    c.Writer.WriteHeader(200)
    indexHtml, _ := asset.Asset("dist/index.html")
    _, _ = c.Writer.Write(indexHtml)
    c.Writer.Header().Add("Accept", "text/html")
    c.Writer.Flush()
})
```

关于 
```
fs := assetfs.AssetFS{Asset: asset.Asset, AssetDir: asset.AssetDir, AssetInfo: asset.AssetInfo, Prefix: "template", Fallback: "index.html"}
```

其中： 

`Prefix: "template"` 是为了在查询文件时去除 `template` 前缀，例如原文件路径为 `"template/css/app.ee8ee5dd.css"` => `/css/app.ee8ee5dd.css` 方便和前端请求对应

`Fallback: "index.html"` 意思为如何查询不到则默认返回 `index.html` 文件，因为配置了前缀，这里返回的应该是 `template/index.html`

## 5. 日常开发
在日常开发中，没有必要没改动一下静态文件就要重新生成 asset.go，此时我们可以使用 `-debug` 模式生成 asset.go 文件，这样访问文件还是使用的真实文件

```go
go-bindata-assetfs -debug -o=asset/asset.go -pkg=asset template/... conf/...
```

# 总结
通过 `go-bindata` 和 `go-bindata-assetfs` 的使用，我们可以将静态文件进行打包，最终提供单个分发文件，简化部署和使用。

# 题外

提一下 go-bindata 项目之前的一些周折。

如果你搜索 go-bindata 的文章，会发现早期的文章指向的项目地址往往是：https://github.com/jteeuwen/go-bindata 。那是最早的项目地址，jteeuwen 是原作者 Jim Teeuwen 的账号。

但不知道什么时候，因为什么原因，原作者把项目关闭了，连 jteeuwen 这个账号都删除了。（从现存线索推断，大约是 2018 年的事）

现在原地址也有一个项目，但已经 不是原项目 ，也 不再维护 了。那是有人发现 go-bindata 删除后，为了让依赖它的项目不会报错，重新注册了 jteeuwen 这个账号，重新 fork 了这个项目 (真正原项目已删，是从一个 fork 那里 fork 的) 。因为初衷是让某个项目能够继续工作（据说是已经没法修改的私人项目，所以也不能指向新的地址），并没有打算继续维护，也不想冒充原项目，所以这个项目设为了 archived (read only)。详情可以参考以下讨论：

https://github.com/jteeuwen/go-bindata/issues/5

https://github.com/jteeuwen/discussions/issues

现在给出的项目地址，不确定跟原作者有没有关系——估计是没有的。那它不过是众多 fork 的其中一个。选它仅仅因为它最活跃、关注人数最多。这可能跟它挂在了同名 organization 下有一定关系，也可能里面有某个大牛。

理由并不重要，只需要知道它最活跃是一个共识，就够了。

> 题外这段引用 https://jaycechant.info/2020/go-bindata-golang-static-resources-embedding/

# 参考

- https://github.com/jteeuwen/go-bindata
- https://github.com/elazarl/go-bindata-assetfs
- https://github.com/go-bindata/go-bindata