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
# 命名私钥，默认是id_rsa，最好保持默认吧，不然要额外进行配置

ssh-copy-id -i container_rsa.pub <linuxUserName>@<host-ip>
# 手动复制

clip < ~/.ssh/id_rsa.pub  //将key复制到剪切板
# 登录主机，查看是否成功收到
cat ~/.ssh/authorized_keys

# 非默认命名密钥对，需要配置 ssh 管理秘钥
ps -ef |grep ssh-agent
ssh-add ~/.ssh/git_rsa
# 或者配置`~/.ssh/config`
```bash
vim ~/.ssh/config

Host my.host		# 给远程主机起一个容易阅读的名字
  HostName <address>    # 远程主机地址,ip或者域名
  Port 22				# 远程主机SSH服务监听端口
  User <my>				# 登录用户
  PreferredAuthentications publickey	# 可选，指定鉴权方式
  IdentityFile ~/.ssh/git_rsa		# 私钥所在路径，前面已经将公钥上传给主机了

# 先安装插件
sudo yum install bash-completion
apt-get install bash-completion
# 补全
source /usr/share/bash-completion/completions/git
source /etc/bash_completion.d/git

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
man top
# 检索正则
# /^\s*wa CPU总览中 io-wait
# /running

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


## ping | telnet | netstat | tcpdump | sar
```bash
ping <ip>

telnet <ip> <port>

netstat -lntp # ss -ltp

# 确定网卡
# -n DEV 表示显示网络收发的报告，间隔1秒输出一组数据
sar -n DEV 1

tcpdump -i eth0 -nn tcp port 80

tcpdump -i eth0 -nn -s0 -v tcp port 80
# -i : 选择要捕获的接口，通常是以太网卡或无线网卡，也可以是 vlan 或其他特殊接口。如果该系统上只有一个网络接口，则无需指定。
# -nn : 单个 n 表示不解析域名，直接显示 IP；两个 n 表示不解析域名和端口。这样不仅方便查看 IP 和端口号，而且在抓取大量数据时非常高效，因为域名解析会降低抓取速度。
# -s0 : tcpdump 默认只会截取前 96 字节的内容，要想截取所有的报文内容，可以使用 -s number， number 就是你要截取的报文字节数，如果是 0 的话，表示截取报文全部内容。
# -v : 使用 -v，-vv 和 -vvv 来显示更多的详细信息，通常会显示更多与特定协议相关的信息。
# port 80 : 这是一个常见的端口过滤器，表示仅抓取 80 端口上的流量，通常是 HTTP
# tcp 指定抓取协议

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
# -P, --perl-regexp
# -E, --extended-regexp
echo office365 | grep -P '\d+' -o
find . -name "*.txt" | xargs grep -P 'regex' -o

# atime	access time	访问时间	文件中的数据库最后被访问的时间
# mtime	modify time	修改时间	文件内容被修改的最后时间
# ctime	change time	变化时间	文件的元数据发生变化。比如权限，所有者等
for i in $(find /data/log -type f -mtime +7);do > $i;done;
for i in $(find /data/log -size +20M);do > $i;done;

ls -rlth /data/services/ | grep -v "总用量" | awk '{print $NF}' | while read f; do echo $f; done

find /data/log -type f -mtime +90 |xargs -d"\n" ls -alh
find . -type f -print0 |xargs -0 rm -rf

# awk
grep "key" <file> |awk -F '=' '{print $2}'  
lsof |grep delete |awk -F' ' '{print $10}' |sort |uniq

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

## date | uname | lscpu
```bash
# 显示当前时间的时间戳
date +%s
date -d @978278400

# 查看系统信息
uname -a
```

## stress | ab
```
# 压力测试工具
stress --cpu 1 --timeout 60

# 并发10个请求测试Nginx性能，总共测试100个请求
$ ab -c 10 -n 100 <http-url>
# 或者 -n 请求次数 换成 -t 请求时间
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

# docker

```bash
# -e env list
docker run --name csql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql

docker run --name credis -p 6379:6379 -d redis
```

# Linux 性能优化实战

## CPU 性能
### 平均负载(Load Average)

压测
```bash
uptime
# 02:34:03 # 当前时间
# up 2 days, 20:14,  1 user,  # 系统运行时长
# load average: 0.63, 0.83, 0.88 # 过去 1 分钟、5 分钟、15 分钟的平均负载
man uptime
# top 第一行信息包含了 uptime
top 

# 查看 CPU 个数（核数）
grep "processor" /proc/cpuinfo

# stress 提高压力
stress --cpu 1 --timeout 600
watch -d update

# 多核 CPU 状态工具 -P指定CPU 5统计间隔 1输出一次
mpstat -P ALL 5 1
# 多核进程状态分析工具 pidstat -u 5 1
pidstat -u 5 1
# -d 分析 io
# -w 分析 switch
```

### 上下文切换

多任务管理，切换任务时，需要保存 CPU 寄存器和程序计数器等上下文

根据任务的不同，CPU 的上下文切换就可以分为几个不同的场景，也就是进程上下文切换、线程上下文切换以及中断上下文切换。

- 进程 CPU 上下文切换

进程上下文切换，是指从一个进程切换到另一个进程运行。

而系统调用过程中一直是同一个进程在运行。所以，系统调用过程通常称为特权模式切换，而不是上下文切换。但实际上，系统调用过程中，CPU 的上下文切换还是无法避免的。
CPU 寄存器里原来用户态的指令位置，需要先保存起来。接着，为了执行内核态代码，CPU 寄存器需要更新为内核态指令的新位置。最后才是跳转到内核态运行内核任务。而系统调用结束后，CPU 寄存器需要恢复原来保存的用户态，然后再切换到用户空间，继续运行进程。所以，一次系统调用的过程，其实是发生了两次 CPU 上下文切换。

- 线程 CPU 上下文切换

同进程内的线程切换，要比多进程间的切换消耗更少的资源，而这，也正是多线程代替多进程的一个优势。

- 中断 CPU 上下文切换

跟进程上下文不同，中断上下文切换并不涉及到进程的用户态。对同一个 CPU 来说，中断处理比进程拥有更高的优先级，所以中断上下文切换并不会与进程上下文切换同时发生。同样道理，由于中断会打断正常进程的调度和执行，所以大部分中断处理程序都短小精悍，以便尽可能快的执行结束。

vmstat 是一个常用的系统性能分析工具，主要用来分析系统的内存使用情况，也常用来分析 CPU 上下文切换和中断的次数。

每隔5秒输出1组数据
$ vmstat 5

cs（context switch）是每秒上下文切换的次数。

in（interrupt）则是每秒中断的次数。

r（Running or Runnable）是就绪队列的长度，也就是正在运行和等待 CPU 的进程数。b（Blocked）则是处于不可中断睡眠状态的进程数。

- 查看具体进程上下文切换 

每隔5秒输出1组数据 上下文切换信息
$ pidstat -w 5

一个是  cswch  ，表示每秒自愿上下文切换（voluntary context switches）的次数，

另一个则是  nvcswch  ，表示每秒非自愿上下文切换（non voluntary context switches）的次数。

所谓自愿上下文切换，是指进程无法获取所需资源，导致的上下文切换。比如说， I/O、内存等系统资源不足时，就会发生自愿上下文切换。

而非自愿上下文切换，则是指进程由于时间片已到等原因，被系统强制调度，进而发生的上下文切换。比如说，大量进程都在争抢 CPU 时，就容易发生非自愿上下文切换。

- 查看中断

/proc/interrupts 查看硬中断
/proc/softirqs 软中断

Linux 通过 /proc 虚拟文件系统，向用户空间提供了系统内部状态的信息

Linux 中的中断处理程序分为上半部和下半部：上半部对应硬件中断，用来快速处理中断。下半部对应软中断，用来异步处理上半部未完成的工作。

网卡接收到数据包后，会通过硬件中断的方式，通知内核有新的数据到了。这时，内核就应该调用中断处理程序来响应它。你可以自己先想一下，这种情况下的上半部和下半部分别负责什么工作呢？对上半部来说，既然是快速处理，其实就是要把网卡的数据读到内存中，然后更新一下硬件寄存器的状态（表示数据已经读好了），

最后再发送一个软中断信号，通知下半部做进一步的处理。而下半部被软中断信号唤醒后，需要从内存中找到网络数据，再按照网络协议栈，对数据进行逐层解析和处理，直到把它送给应用程序。

> list
```bash
# 系统切换总览
vmstat 5
# 进程切换
pidstat -w

# 中断
cat /proc/interrupts
cat /proc/stat |grep ^cpu
```

### 进程 CPU 分析

通过 top、ps、pidstat 等工具，你能够轻松找到 CPU 使用率较高的进程

占用 CPU 的到底是代码里的哪个函数呢？找到它，你才能更高效、更针对性地进行优化。除了 GDB 还可以用 perf 来分析

第一种常见用法是 perf top，类似于 top，它能够实时显示占用 CPU 时钟最多的函数或者指令，因此可以用来查找热点函数


命令list
```bash
# 分析某个进程函数分析 -g 显示调用
# 实时
sudo perf top -g -p <pid>
# 采集后再看
perf record -g
perf report

# 查看进程调用，找到父进程
pstree | grep stress
```

