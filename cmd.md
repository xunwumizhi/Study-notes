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
w # 向尾部移动一个单词（光标停在单词首部）
b # 向头部移动一个单词
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
ggyG # 全部复制，先按gg，然后ggyG
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
git stash drop stash@{0}
git stash clear # 清除所有


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
git reset master -- ./    # 当前目录下文件回滚到master
git reset --hard <commit>

git checkout master -- ./  # checkout当前目录文件至master，但不会删除新增文件

# U untracked file, 新增文件为Untracked状态，这时git clean等于 Linux rm 文件效果
git clean --help
git clean -ndf    # 手动删除文件

# tracked file, 对已有文件操作，等于 Linux rm 后再 git add
git rm -rf ${file}
```

## submodule

git clone下来的工程中带有submodule时，初始的时候，submodule的内容并不会自动下载下来的，此时，只需执行如下命令：
```bash
git submodule init
git submodule update
# 等价于
git submodule update --init --recursive
```

# Linux

## tar |uniq |whereis |mv |ls
```bash
# tar
tar -xzvf <input.tar.gz>
tar -czvf <output.tar.gz> <inputfile>...
# -x --extract, -c --create, -z --gzip... , -v --verbose, -f --file

# uniq
uniq | wc -l

tail -n 5 # tail 默认展示后10行

head -n 20 text.txt |tail -n 10 # 第11~20行

cat text.txt |wc -l
tailf <log-file> # tail -f <log-file>

# lsof
lsof |grep delete |awk -F' ' '{print $10}' |sort |uniq


which python
whereis python

# 一次移动多个
mv SOURCE... -t DIRECTORY

sudo ls -al /proc/22686/fd
ls -alhr ./
```


## top | free | df | du

```bash
# 10分钟教会你看懂top - 掘金: https://juejin.cn/post/6844903919588491278

# Linux top命令的用法详细详解 - 莫水千流 - 博客园: https://www.cnblogs.com/zhoug2020/p/6336453.html
top

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


## ping | telnet | netstat | tcpdump
```bash
ping <ip>

telnet <ip> <port>

netstat -lntp # ss -ltp

tcpdump -i eth0 -nn -s0 -v port 80
# -i : 选择要捕获的接口，通常是以太网卡或无线网卡，也可以是 vlan 或其他特殊接口。如果该系统上只有一个网络接口，则无需指定。
# -nn : 单个 n 表示不解析域名，直接显示 IP；两个 n 表示不解析域名和端口。这样不仅方便查看 IP 和端口号，而且在抓取大量数据时非常高效，因为域名解析会降低抓取速度。
# -s0 : tcpdump 默认只会截取前 96 字节的内容，要想截取所有的报文内容，可以使用 -s number， number 就是你要截取的报文字节数，如果是 0 的话，表示截取报文全部内容。
# -v : 使用 -v，-vv 和 -vvv 来显示更多的详细信息，通常会显示更多与特定协议相关的信息。
# port 80 : 这是一个常见的端口过滤器，表示仅抓取 80 端口上的流量，通常是 HTTP。
```

## curl | wget

```bash
# curl 网络请求
curl -i -H "Content-Type: application/json" -X POST -d '{""}' "<URL>"

# 下载并重命名
wget -O <localPath> <remoteUrl>
curl -o <localPath> <remoteUrl>
```


## passwd | group
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

## mount

```bash
mount
```

## nc |nohup |journalctl
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

## grep | awk | sed | find

```bash
# grep 提取字符串
echo office365 | grep -P '\d+' -o
find . -name "*.txt" | xargs grep -P 'regex' -o




# awk
grep "key" <file> |awk -F '=' '{print $2}'  

# 获取key的值
# sed 正则提取字符串，用单引号'可用转义字符
echo here365test | sed 's/.*ere\([0-9]*\).*/\1/g'
# 使用双引号，要显示 -E 使用正则
sed -E "s#key=\"(.*)\"#\1#g"
echo '{"code":46015,"msg":"bad request"}' |sed -E "s/.*code\":([0-9]*).*/\1/g"
# 单引号写法
echo '{"code":46015,"msg":"bad request"}' |sed 's/.*code":\([0-9]*\).*/\1/g'
# sed 使用变量、替换变量
pattern1=XXX
pattern2=XXX 
sed -i "s/$pattern1/$pattern2/g" inputfile


grep -C 3 <content> # 上下几行
grep -r "" /data

cat all_pods_list | grep -wf pods.txt | awk '{print $1,$2}' | while read x;do kubectl get pods -n $x; done
cat all_pods_list | grep -wf pods.txt | awk '{print "kubectl get pods -n",$1,$2}' | xargs -P 5 -I {} bash -c {}
```

## scp
```
scp "/data/karlkang/kg_contest_tool/download/top100_$actid.csv" "user_00@9.146.222.202:/cfs/cm3/upload/kg_contest_data_tool"
```

## chmod | chwon
```
chwon -R root /var/run/httpd.pid
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
