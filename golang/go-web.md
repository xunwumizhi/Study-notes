
## web 标准库

-   net/hhtp
主需要两步：1. 创建路由器并注册路由；2. 创建服务器server，使用路由器，启动服务
```go
// 创建路由器ServerMux
mux := http.NewServeMux()
// 注册路由
mux.HandleFunc(pattern, myHandleFunc) // 也可以用mux的Handle方法来注册实现了ServeHttp方法的handler
// 创建服务器
server := &Server{Addr: addr, Handler: mux}
// 启动服务
server.ListenAndServe()

```

使用net/http包DefaultServeMux快速创建

```go
func main() {
	handler := func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "hello")
	}
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe(":8895", nil)
}

```

- 包装
```go
func  HandleWrapper(h func(http.ResponseWriter, *http.Request)) func(http.ResponseWriter, *http.Request) {
	return  func(w http.ResponseWriter, r *http.Request) {
		h(w, r)

		// before return
		w.Header().Set("Content-Tpye", "application/json")
	}
}
```


-   context

上下文，升级版的goroutine消息传递channel，方便父routine控制子routine

## fasthttp

```go
ctx *fasthttp.RequestCtx
ctx.QueryArgs().Peek()
ctx.UserValue("urlArg")
ctx.PostBody()

```

## beego

```bash
bee version
bee api apiproject
bee run  # 热发布，修改后无需再次启动程序
bee pack

# 自动化生成文档
bee run -gendoc=true -downdoc=true

```

注解路由，`[options,get]`里面逗号分隔，不能带空格！！

```go
// @router /staticblock/:key [options,get]
func (this *CMSController) StaticBlock() {

}

// 并在 router.go 中通过如下方式注册路由：
beego.Include(&CMSController{})

```

过滤，重定向

```go
beego.InsertFilter("/*", beego.BeforeRouter, FilterUser)

if ctx.Request.Method == "OPTIONS" {
    // ctx.Abort(200, "Network is ok")
    ctx.Output.Body([]byte("Network is ok"))
    return
}

```

## Json Patch

`type. JSONPatchType`  PatchType = “application/json-patch+json”

带上操作json，适合删除改变现有json格式：`[{"op":"remove", "path":"/metadata/annotations/key1" }]`

是一组操作。

`type.MergePatchType`  PatchType = “application/merge-patch+json”

直接覆盖替换，没有则创建，json要从`"metadata": {`开始