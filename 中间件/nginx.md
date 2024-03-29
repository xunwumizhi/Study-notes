# 运维cmd
```bash
nginx -g 'daemon off;'
nginx -t -c conf/nginx.conf # 检测配置文件是否正常  
nginx -s reload
nginx -s quit
nginx -s stop # 强制终止Nginx，不管当前是否有任务在执行  
```

# 系统了解

Nginx一网打尽：动静分离、压缩、缓存、黑白名单、跨域、高可用、性能优化...想要的这都有！ - 掘金
https://juejin.cn/post/7112826654291918855

Nginx一网打尽：动静分离、压缩、缓存、黑白名单、跨域、高可用、性能优化...
https://mp.weixin.qq.com/s/yYkGDN4jlWw6rj3pGAbDJg

# 概念

## 正向代理与反向代理

正向代理，client的替身向server发起请求；

反向代理，可以理解为服务器网关，是server的替身，client向替身——反向代理服务器(如Nginx)发起请求


# 配置 Nginx

nginx.conf 配置文件

四个重要的设置：全局main、主机server、上游服务器upstream、location

## 下载访问本机文件
```conf
http {
  ...
  server: {
    # 配置下载
        location /download {
            alias D:\\download;
            autoindex on;
            autoindex_exact_size off;
        }
  }
  ...
}
```


## proxy_pass：解决跨域

```bash
location /api {
    add_header Access-Control-Allow-Origin '*';
    add_header Access-Control-Allow-Headers '*';

    proxy_pass https://.com;
}
```

location中proxy_pass 后面host命名加不加`/`的区别

nginx 之 proxy_pass详解 - Skyline - CSDN博客  https://blog.csdn.net/zhongzh86/article/details/70173174



## proxy_set_header：设置请求头

允许重新定义或添加字段传递给代理服务器的请求头。

用以权限认证，检查等
