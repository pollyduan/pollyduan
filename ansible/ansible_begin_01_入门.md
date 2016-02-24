ansible学习笔记 - 入门
===

1 安装ansible
---

### 1.1 pip安装

```
sudo pip install ansible
sudo mkdir /etc/ansible
sudo touch /etc/ansible/hosts
```
OS X Mavericks里:

```
$ sudo CFLAGS=-Qunused-arguments CPPFLAGS=-Qunused-arguments pip install ansible
```

### 1.2 yum安装

```
yum install -y epel-release
yum install -y ansible
```

2 初始化配置
---

### 2.1 关键词
由于一些关键词直接翻译并不容易理解，首先约定几个关键词：

被控节点清单：Inventory
组：group
主机：host
剧本：playbooks
模块：module
参数：arguments

### 2.2 配置被控节点

创建或编辑 `/etc/ansible/hosts` ，作为被控节点服务器的主清单文件。

```
192.168.33.10
```

### 2.3 测试节点连接：

主清单文件``中只配置一个主机：

```
#/etc/ansible/hosts
192.168.33.10
```

测试一下：

```
$ ansible all -m ping
192.168.33.10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 2.4 在节点主机上运行实时命令

执行一个helloword:

```
$ ansible all -a "/bin/echo hello"
192.168.33.10 | SUCCESS | rc=0 >>
hello
```

### 2.5 被控节点key检查

如果节点没有在被控机的~/.ssh/know_hosts中初始化，ansible在执行时会提示确认，需要人工交互输入yes。

如果被控机重装系统导致key与登记的key不同了，在ansible访问被控节点的时候，会抛出错误提示，直到修复。如何修复？通常情况下需要手工删除`know-hosts` 的 `key`,重新登录被控机。

如果你不想接受这种处理，可以禁用这种行为，通过编辑 `/etc/ansible/ansible.cfg` 或 `~/.ansible.cfg`，填入如下内容：

```
[defaults]
host_key_checking = False
```

还有另外一种方式——设置一个环境变量：

```
$ export ANSIBLE_HOST_KEY_CHECKING=False
```


3 使用ansible命令行
---

特定命令(Ad-Hoc Commands)是临时写的实时运行的命令，不想保存以后重用的命令。不知道官网为毛非要用这个拉丁语的泊来词，刚接触还以为是个什么特别的词组缩写。

ansible命令的格式：

```
ansible <host-pattern> -m <module_name> -a <arguments>
# module_name 缺省为command

#其他开关,如果操作需要提升权限，请增加参数： --sudo -K
```

### 3.1 host-pattern

该命令字中有些用法会有特殊字符，如果控制机使用的是zsh，类似于*，!，[等这些特殊符号需要用单引号包裹。为了保持兼容，建议都使用单引号包裹。

测试之前在hosts中增加一个组：

```
#/etc/ansible/hosts
192.168.33.10

[g1]
192.168.33.10
192.168.33.20

[g2]
192.168.33.10

```

#### 3.1.1 指定一个主机或组名

```
ansible g1 -a "echo hello"
```

多个主机或组名，用冒号分隔。
隐含的组，all 代表所有主机。

#### 3.1.2 用!剔除主机

```
$ ansible 'g1:!g2' -a "echo hello"
192.168.33.20 | SUCCESS | rc=0 >>
hello
```

表示忽略隶属于g1，并且隶属于g2中的机器。只控制隶属于g1，但不隶属于g2中的机器。

#### 3.1.3 用&获取交集

```
$ ansible 'g1:&g2' -a "echo hello"
192.168.33.10 | SUCCESS | rc=0 >>
hello
```

只控制隶属于g1，且同时隶属于g2中的机器。

#### 3.1.4 用下标提取组中的某些机器

```
$ ansible 'g1[1]' -a "echo hello"
192.168.33.20 | SUCCESS | rc=0 >>
hello

```

提取g1中的第二台机器。

```
$ ansible 'g1[0:1]' -a "echo hello"
192.168.33.10 | SUCCESS | rc=0 >>
hello

192.168.33.20 | SUCCESS | rc=0 >>
hello
```

提取g1组中的前两台主机。

#### 3.1.5 用~开头来使用正则表达式匹配主机

```
$ ansible '~192.*20' -a "echo hello"
192.168.33.20 | SUCCESS | rc=0 >>
hello
```

### 3.2 module模块

模块是ansible操作主机的主要插件。

查看安装了哪些模块？

```
ansible-doc -l
```

查看一个模块的文档：

```
ansible-doc yum
```

andible官网提供的模块有16类，上百个,详见：[http://docs.ansible.com/ansible/modules_by_category.html]()。这里列举一些常用的。

#### 3.2.1 执行命令

command和shell类似，区别在于：command不是通过shell运行的，不支持环境变量和重定向>|<、管道|、后台&等符号。

script是把本地脚本上传到远程主机，并在远程主机运行。

```
# 打印hello
ansible all -a "/bin/echo hello"

# 重启
ansible all -a "/sbin/reboot" -f 10

# 使用shell模块 注意当前shell的引号问题
ansible all -m shell -a 'ls -al'

# 在远程节点执行本地脚本
ansible 192.168.33.20 -m script -a './script.sh'
```

#### 3.2.2 文件操作

```
# 传输文件
ansible 192.168.33.20 -m copy -a "src=/etc/hosts dest=~/tmp/hosts"
```

测试时遇到提示缺少libselinux-python组件:

```
Aborting, target uses selinux but python bindings (libselinux-python) aren't installed!
```
用 `yum` 安装即可。

```
# 修改文件权限，所有者，分组（这些参数可以用在copy模块中）
ansible 192.168.33.20 -m file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
```

创建文件夹

ansible会自动级联创建，类似于`mkdir -p`。

```
ansible 192.168.33.20 -m file -a "dest=~/tmp mode=755 owner=mdehaan group=mdehaan state=directory"
```

删除文件(夹)

```
ansible 192.168.33.20 -m file -a "dest=~/tmp state=absent"
```

#### 3.2.3 用包管理器管理软件

模块名为yum或apt

确保软件安装，但不更新它。软件名可带版本号，如wget-1.1.2

```bash
ansible 192.168.33.20 -m yum -a "name=wget state=present"
```

确保安装最新版本

```bash
ansible 192.168.33.20 -m yum -a "name=wget state=latest"
```

确保软件不被安装

```bash
ansible 192.168.33.20 -m yum -a "name=wget state=absent"
```

#### 3.2.4 用户管理

创建用户

```bash
ansible 192.168.33.20 -m user -a "name=foo password=foo"
```

删除用户

```bash
ansible 192.168.33.20 -m user -a "name=foo state=absent"
```

```
192.168.33.20 | SUCCESS => {
    "changed": true,
    "force": false,
    "name": "foo",
    "remove": false,
    "state": "absent"
}
```

#### 3.2.5 Git

```bash
ansible v1 -m git -a "repo=https://github.com/axiaoxin/json_line_format.vim.git dest=~/project-dir version=HEAD"
```

#### 3.2.6 服务管理

```
ansible v1 -m service -a "name=mysql state=started"
ansible v1 -m service -a "name=mysql state=restarted"
ansible v1 -m service -a "name=mysql state=stopped"
```

#### 3.2.7 收集信息(facts)

显示系统的变量信息。

```
ansible 192.168.33.20 -m setup
```

