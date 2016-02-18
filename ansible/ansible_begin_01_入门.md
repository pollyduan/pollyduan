ansible学习笔记 - 入门
===

1 安装ansible
---

2 初始化配置
---

### 2.1 关键词
由于一些关键词直接翻译并不容易理解，首先约定几个关键词：

被控节点：Inventory
组：group
主机：host

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

### 2.4 向节点发送消息

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

