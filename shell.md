```bash
# 执行字符串命令
cmd="date"
$cmd
echo ${cmd}|awk '{run=$0;system(run)}'
```

# if else

```bash
# sh 文件
if [[ "$name" = "Tom" ]]; then
    echo ${name}" = Tom"
else
    echo ${name}
fi

# 在 shell 直接执行
if [[ "$name" != "Tom" ]]; then echo $name" != Tom"; else echo "Tom"; fi
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
