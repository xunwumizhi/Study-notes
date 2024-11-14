# log

```shell
printf "Number: %d\n" 42

echo
```

# basic

```bash
# 清理文件
for i in $(find ./ -name '*.log'); do > $i; done

# 查找根目录下大于500M的文件，排除/proc目录
find / ! -path "/proc/*" -type f -size +500M | sort -rh|xargs ls -lh | awk '{ print $9 ": " $5 }'

# 如果排除俩个目录
find / ! -path "/proc/*" ! -path "/home/*" -type f -size +500M | sort -rh|xargs ls -lh | awk '{ print $9 ": " $5 }'

# [a|c|m]min    [最后访问|最后状态修改|最后内容修改]min 单位分钟
# [a|c|m]time    [最后访问|最后状态修改|最后内容修改]time 单位小时
find ./ -type f -mtime 0  #查找一天内修改的文件
find ./ -type f -mtime -2 #查找2天内修改的文件，多了一个减号
find ./ -type f -mtime +2 #查找2天前修改的文件，多了一个减号

# 执行字符串命令
cmd="date"
$cmd
echo ${cmd}|awk '{run=$0;system(run)}'
```

# if else

Shell test 命令 | 菜鸟教程
https://www.runoob.com/linux/linux-shell-test.html

```bash
# sh 文件
if [[ "$name" = "Tom" ]]; then
    echo ${name}" = Tom"
elif [[ "$name" = "Jerry" ]]; then
    echo "elif"
else
    echo ${name}
fi

# 在 shell 直接执行
if [[ "$name" != "Tom" ]]; then echo $name" != Tom"; else echo "Tom"; fi
```

# switch

```shell
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac

case "$site" in
   "runoob") echo "菜鸟教程"
   ;;
   "google") echo "Google 搜索"
   ;;
   "taobao") echo "淘宝网"
   ;;
esac
```


# for range

```bash
# 管道接循环
cat lines.txt |while read f; do echo $f; done
ls -rlth /data/services/ | grep -v "总用量" | awk '{print $NF}' | while read f; do echo $f; done

# for in
for i in $(find /data/log -type f -mtime +7); do > $i; done
for i in $(cat lines.txt); do bash run.sh $i; done
```


# mysql

```bash
# 执行SQL获取结果
dataTime=$(echo "<sql>" | mysql --skip-column-names --default-character-set=utf8 <...>)

# 读SQL文件执行
cat <create.sql> | mysql ...
```

# curl

```bash
# json参数替换
template='{"bucketname":"%s","objectname":"%s","targetlocation":"%s"}'
json_string=$(printf "$template" "$BUCKET_NAME" "$OBJECT_NAME" "$TARGET_LOCATION")

# curl
response=$(curl -i \
            --user ${username}:${api_token} \
            -X POST \
            -H 'Content-Type: application/json' \
            -H "${head2}"
            -d "${json}" \
            "https://api.github.com/repos/${username}/${repository}/releases" \
            --output /dev/null \
            --write-out "%{http_code}" \
            --silent
          )
if [[ "$response" = "200" ]]; then
    echo "ok"
else
    echo "bad code"${response}
fi
```

```bash
# 引用本地文件JSON文件
addr="120.0.0.1:8080"
curl -i -X POST -d "@req.json" -H "Content-Type: application/json" http://$addr/api
```

# 综合例子

192. 统计词频 - 力扣（LeetCode）: https://leetcode.cn/problems/word-frequency/

```bash
cat words.txt |tr -s ' ' '\n' |sort |uniq -c |sort -nr |awk '{print $2" "$1}'
```
