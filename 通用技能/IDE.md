# VScode

ssh远程开发。搜索remote develop插件，安装

本地打开`~/.ssh/config`

```
# VM ssh
Host ubuntuVM
    HostName 192.168.163.130 # 地址
    User linuxUserName # 好像可以不要
```

密码登录连接，右下角弹出小心，点进去第一次会配置一些环境，然后输入密码`Setting up SSH Host ubuntuVM: ([details](command:opensshremotes.showDetails)) Initializing`

接着设置免密登录：

```
# 本地生成密钥对
ssh-keygen -t rsa -C "linuxUserName"
# ssh-keygen -t rsa -P '' -f ~/.ssh/vm_id_rsa

# 发送公钥
ssh-copy-id -i vm_id_rsa.pub linuxUserName@192.168.163.130
# 在VM查看，是否成功收到
$HOME/.ssh/authorized_keys

# 修改本地ssh配置config，添加密码信息
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/vm_id_rsa
    
最终本地config文件：
# 其他host
# ......

# VM settings
Host ubuntuVM
	HostName 192.168.163.130
	IdentityFile ~/.ssh/vm_id_rsa
    PreferredAuthentications publickey
    User linuxUserName
```



# Idea

清单：

代码格式化

快捷键

mvn



# Excel

数字转文本，科学表达式烦人，【数据】-【分列】

单元格查找合并，`=VLOOKUP(M4,Sheet1!A:B,2,FALSE)`，变的单元格加数字M4，范围是不变的，不要带数字，写成A:B形式



对比两个表格异同：【开始】-【条件格式】-【新建规则】。。。

