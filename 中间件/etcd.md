# etcd

```go
s, err := concurrency.NewSession(c, concurrency.WithTTL(15))
e := concurrency.NewElection(s, keyPrifex)

ctx, ctxCancel := context.WithCancel(context.TODO())
defer ctxCancel()
if err = e.Campaign(ctx, keyValue); err != nil {
    fmt.Println(err)
    return
}

select {
case <-stop:
case <-s.Done(): // 等待lease过期，但由于session为lease自动续约，过期时意味着ctx关闭，session退出，到期后没续约
    log.Println(w.name, "elect: expired")
}
// 辞职
e.Resign(context.TODO()) // 传递一个新ctx
```
