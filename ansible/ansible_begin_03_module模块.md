ansible是以模块为基础的，所有的功能都是用模块实现。

一 了解模块
===

modules模块(指的是`task plugins`或`library plugins`)是ansible实际要处理的工作。

1 查看已安装的模块
---

```
ansible-doc -l
```

2 查看模块帮助信息
---

格式：

```
ansibled-doc [模块名]
```

如下，查看yum模块如何使用：

```
ansible-doc -s yum
```

列出的yum的参数，其中有些参数是必选的，它的参数名后面会有一个`=`号，其他参数为可选参数。

显示详细的帮助文档：

```
ansible-doc yum
```

3 模块分类
---

### 3.1 核心模块(Core Modules)

这些模块是ansible核心工作组维护的，会一直包含在ansible的后续版本中。核心模块会比非核心模块接收更高优先级的pull request。

核心模块源码托管在github的[ansible-modules-core](http://github.com/ansible/ansible-modules-core)仓库中。

### 3.2 扩展模块(Extras Modules)

这些模块是ansible附带的，可能在将来被分离，这些模块是由ansible社区维护的。非核心组件会一只可用，但会接收低优先级的响应率和pull request。

模块分类的归属也不是一成不变的，随着时间的推进，流行的扩展模块会被提升到核心模块中。

扩展模块源码托管在github的[ansible-modules-extra](https://github.com/ansible/ansible-modules-extras/tree/devel)仓库中。

### 3.3 官方模块索引

所有插件都可在[官方模块目录](http://docs.ansible.com/ansible/modules_by_category.html)中检索到。

4 模块的参数
---

每个模块可以携带参数(arguments)，格式为一组用空格分隔的`key=value`形式的键值对。特殊的，`ping`不携带参数；`command`和`shell` 带有一个shell命令行的字符串作为参数。

```
- name: restart webserver
  service: name=httpd state=restarted
```

在playbooks中还有一种成为#符合参数#传递参数的方式，它传递的是一个hash，如：

```
- name: restart webserver
  service:
    name: httpd
    state: restarted
```

5 模块的返回值
---

当使用ansible程序作为输出的时候，通常模块会返回一个数据结构，它们可以被注册为variables变量或者直接阅读。

### 5.1 facts

一些模块会返回`facts`对象，这是通过json hash对象中一个`ansible_facts`key来实现的。里边的所有东西对当前主机来说都可以作为一个变量来使用。

### 5.2 status

每个模块一定返回一个状态值status，指明该模块是否执行成功，某些东西是否发生了改变。

### 5.3 其他通用返回值

通常在模块运行失败或成功时都会返回一个消息，用来解释失败的原因或者对执行结果做一个注解。

二 常用模块
===

1 setup
---

用来收集远程主机的一些基本信息。

2 ping
---

用来检查主机是否存活。

3 Files文件模块
---

### 3.1 file

此处的file为广义的文件，包括文件、软链和目录等。该模块用于设置文件的属性，或者删除文件。

其中`path`参数是必选的，其他参数是可选的。需要注意的是，`path`有两个别名，分别为`dest`和`name`,也就是说这三个有其一即可。

`state`参数是可选的，缺省为file，常用的可选值为：

参数值|说明
---|---
file|设置文件属性，如果文件不存在不会创建
link|处理软链
directory|创建目录，如果目录不存在它会自动递归创建子目录
hard|处理硬链接
touch|创建文件
absent|删除文件，如果是目录会递归删除

其他常用参数：

参数|说明
---|---
group|设置文件的组
mode|设置文件的属性
owner|设置文件的属主
path|要操作的目标文件名，别名dest,name
recurse|是否递归调用，只对目录有效(when state= directory)，可选值为yes或no，缺省为no
src|要操作的源文件名，如果软链的源文件

范例：

```
#创建文件
ansible 192.168.33.20 -m file -a 'path=/tmp/newfile state=touch'
#创建目录
ansible 192.168.33.20 -m file -a 'path=/tmp/dir1 state=directory owner=root mode=666'
#创建软链
ansible 192.168.33.20 -m file -a 'src=/etc/ansible/hosts dest=/tmp/inventory state=link'
```

### 3.2 copy

用于从控制机上传文件到远程主机。

参数：

dest 是保存的目标文件；这个参数是必须的。

src和content二者必须提供一个。其中content是一个字符串，用来写入目标文件。而src是源文件路径，如果src是一个目录，则递归复制。如果src以`/`结尾，则复制这个目录下的所有文件；如果不是以`/`结尾，则复制整个目录。

backup参数，如果目标文件存在，且内容发生了变化时，是否备份目标文件。该选项对文件夹无效。

mode、group、owner可以设置目标文件属性，同file模块类似。

```
#向远程主机的目标文件写入一个字符串
ansible 192.168.33.20 -m copy -a 'content="abcd efg" dest=~/copyfile.txt'

#复制文件
ansible 192.168.33.20 -m copy -a 'src=~/1.txt dest=~/'

#复制tmp文件夹,tmp文件夹就保存在~/中
ansible 192.168.33.20 -m copy -a 'src=~/tmp dest=~/'

#复制tmp文件夹下的所有文件，tmp文件夹里的文件直接保存到~/中
ansible 192.168.33.20 -m copy -a 'src=~/tmp/ dest=~/'
```

4 commands命令模块
---

### 4.1 shell

在远程节点执行一个命令。执行的命令字符串是必须的，他没有key名。

命令格式为字符串，带有空格分隔的多个参数。如`some_command arg1 arg2 ...`

模块参数：

chdir 在运行命令之前进入工作目录
creates 传入一个文件名，如果文件存在则不运行命令，比如pid文件存在不运行服务。
removes 与creates相反，如果文件不存在则不运行。

```
#下面命令会在远程主机执行显示abcd
ansible 192.168.33.20 -m shell -a "echo abcd"
#/home/tmp文件不存在，这个命令会被运行
ansible 192.168.33.20 -m shell -a "creates=/home/tmp echo abcd"
#/home/tmp文件不存在，这个命令不会被运行
ansible 192.168.33.20 -m shell -a "removes=/home/tmp echo abcd"
#进入工作目录后处理当前目录的文件
ansible 192.168.33.20 -m shell -a "chdir=/opt/logs/myapp tar cvfz log201602.tar.gz *201602*.log'
```

### 4.2 command

参数与shell模块完全一样，区别在于command不是使用shell运行的，所以系统环境变量以及管道符等特殊符号`>,|,<,&`不可使用。


