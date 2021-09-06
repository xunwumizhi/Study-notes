# vim

```bash
# 命令模式
:wq # 保存并退出
:q! # 强制退出并忽略所有更改
:e! # 放弃所有修改，并打开原来文件。

:set number # 显示行号
:set nonumber
:set paste  # 粘贴

# 光标移动
j # 下一行
k # 上一行
w # 向前移动一个单词（光标停在单词首部）
b # 向后移动一个单词
gg # 光标移动到首行
G  # 尾行


# 插入
i # 光标位置前
a # 光标位置后

I # 行首
A # 行尾
o # 插入下一行
O # 插入上一行

# 剪切
dd    # 行
dgg   # 删除当前行至文档首部
dG    # 

d$    # 删除当前字符至行尾
d^    # 
D     # 等价于d$
dw    # 删除字符到单词尾
daw   # 删除整个单词

# 撤销
u      # 撤销（Undo）
Ctrl+r # 反撤销 redo


# 复制粘贴
yy  # 复制整行
yG  # 复制至文末
p   # 粘贴至下一行

yyp # 复制到下一行
ddp # 剪切当前行，复制到下一行的下一行

```

```bash
# 查找
/text              # 正则 ^ $
:set ignorecase　　 # 忽略大小写的查找
:set noignorecase　 # 不忽略大小写的查找
*                   # 查找光标所在单词  
```


# git

## settings
ssh配置
```bash
ssh-keygen -t rsa -C "你的邮箱"
# Enter file in which to save the key (/Users/kingboy/.ssh/id_rsa): 
# 命名私钥，默认是id_rsa

clip < ~/.ssh/id_rsa.pub  //将key复制到剪切板
```

本地新建仓库关联
```bash
git init
git add -A
git commit -m "init"

git remote add origin <address>
git push -u origin master

# after fork
git remote -v
git remote add upstream <origin-repo>
# git remote set-url upstream <origin-repo> # 添加并修改
git remote -v
```

## repository/branch
```bash
git push -u origin feature/new-func
git checkout -b <new_branch> # 基于当前分支构建分支
git checkout <commitId> -b   # 基于某个commit/tag新建分支
git fetch origin remoteBranch:localBranch

git pull --rebase origin master

git remote show origin
git remote prune origin
```

## tag
```bash
git tag -a v1.0 -m "v1.0版本发布"
```

## commit
```bash
git commit --amend --no-edit
git commit --amend -m "add new file"

git stash
git stash list
git stash pop

git push -f
git rebase <commit>
# rebase交互式界面编辑
git rebase -i <commit>
git diff [options] <commit> <commit> [--] [<path>...]
```

## undo
```bash
# 针对merge的撤销，指定回滚到merge前的哪个提交上
git revert <commit> -m 1  

git reflog
git reset <commit>
git reset --hard <commit>
```

# Linux

## comprehensive
```bash
top # https://juejin.cn/post/6844903919588491278


# 一次移动多个
mv SOURCE... -t DIRECTORY


tar -xzvf <input.tar.gz>
tar -czvf <output.tar.gz> <inputfile>...
# -x --extract, -c --create, -z --gzip... , -v --verbose, -f --file

uniq | wc -l
tail -n 5 # tail 默认展示后10行
head -n 20 text.txt |tail -n 10 # 第11~20行
cat text.txt |wc -l
tailf <log-file> # tail -f <log-file>

lsof |grep delete
which python
whereis python

```

## memory-storage

```bash
# 内存
free -h

# 磁盘
df -h
# 查看某个路径下所有文件
du -hs --exclude=/proc /* | sort -hr
# 当前目录
du -hs * 
# 指定目录深度								
du -h -d 1
# 所有文件，不只是目录
du -ha									

```

## file-system
```bash
sudo ls -al /proc/22686/fd

```

## process


## network
```bash
curl -i -H "Content-Type: application/json" -X POST -d '{""}' "<URL>"

ping <ip>

telnet <ip> <port>

netstat -lntp # ss -ltp
```


## user
```bash
# 查看用户
cat /etc/passwd  
# [login_name:passwd:UID:GID:uname:login_path:shell]  
# harbor:x:10000:10000::/home/harbor:/bin/bash

# 查看用户组
cat /etc/group
# [组名:口令:组标识号:组内用户列表]
# segroup:x:1001:zhangsan,lisi,wangwu
```

## device

```bash
mount
```

## systemctl
```bash
init 3

nc -l 9191 < kubectl
nc <server-ip> 9191 > kubectl

systemctl status [service]
supervisorctl status [service]

nohup <cmd> >new.log 2>&1 &
nohup <cmd> >>add.log 2>&1 &
nohup <cmd> >/dev/null 2>&1

/var/log/syslog
journalctl -u kubelet

while read -r line; do echo $line; done < /tmp/.kubetmp
```

## grep | awk | sed

```bash
grep "key" <file> |awk -F '=' '{print $2}'  # sed -E "s#key=\"(.*)\"#\1#g"

cat all_pods_list | grep -wf pods.txt | awk '{print $1,$2}' | while read x;do kubectl get pods -n $x; done
cat all_pods_list | grep -wf pods.txt | awk '{print "kubectl get pods -n",$1,$2}' | xargs -P 5 -I {} bash -c {}
```


# nginx

```bash
nginx -g 'daemon off;'
nginx -s reload
nginx -s quit
```


# supervisor
```bash
vim /etc/supervisor/conf.d/xxx.conf
supervisord -c /etc/supervisor/conf.d/xxx.conf

supervisorctl -h
```



# Mac
PATH路径，全局设置在`/etc/paths.d`下面新建文件，直接写路径：
```
# /etc/paths.d/go
/usr/local/go/bin
```
