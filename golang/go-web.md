## HTTP server client

HTTP客户端使用了连接池，避免频繁建立带来的大开销，

HTTP服务端的路由只是一个静态索引匹配，对于动态路由匹配支持的不好，并且每一个请求都会创建一个gouroutine进行处理，海量请求到来时需要考虑这块的性能瓶颈；

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

## Json Patch

`type. JSONPatchType`  PatchType = “application/json-patch+json”

带上操作json，适合删除改变现有json格式：`[{"op":"remove", "path":"/metadata/annotations/key1" }]`

是一组操作。

`type.MergePatchType`  PatchType = “application/merge-patch+json”

直接覆盖替换，没有则创建，json要从`"metadata": {`开始