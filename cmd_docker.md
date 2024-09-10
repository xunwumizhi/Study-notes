# docker

## 容器

```bash
# -e env1=v1 -e env2=v2
# 从 .env 文件加载环境变量 --env-file
docker run --entrypoint /bin/bash <image> <entry-param>

docker run -p host_port:container_port

docker run --name csql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql

docker run --name cmaria -p 3307:3307 -e MYSQL_ROOT_PASSWORD=root -d mariadb

# docker run --name credis -p 6379:6379 -d redis
docker run --name credis -d redis
docker exec -it credis bash
# redis-cli
```

## 启动命令

RUN <command> (shell 模式)
RUN ["executable", "param1", "param2"] (exec 模式)

CMD ["executable","param1","param2"] (exec 模式, 推荐使用)
CMD command param1 param2 (shell 模式)
CMD ["param1","param2"] (作为 ENTRYPOINT 指令的参数)

使用 ENTRYPOINT 时 CMD 指令会被作为其默认参数, 而用户也可以在启动容器时通过覆盖 CMD 指令来输入参数; 

此外, 这也意味着 ENTRYPOINT 指令的内容不会被 docker run 最后参数覆盖，除非手动指定 --entrypoint /bin/bash -it

## image

```bash
# 清理none镜像
docker images | grep none | awk '{print $3}' | xargs docker rmi
```

## 文件

```bash
docker cp <container_id>:/path/to/file/on/container /path/on/host
```

## 常用镜像

- mongo
```
docker pull mongo:4.4.18

docker run -d --name cmongo_4 mongo:4.4.18
# docker run -d --name cmongo_4 -p 27017:27017 mongo:4.4.18

docker exec -it cmongo_4 bash
```
