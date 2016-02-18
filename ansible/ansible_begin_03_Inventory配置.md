ansible学习笔记 - Inventory配置
===

1 配置文件查找顺序
---

ansible的某些配置是在配置文件中设置的。默认的配置对一般用户来说足够了，但由于一些原因你可能需要修改它。

配置文件的处理优先级顺序如下：

```
* ANSIBLE_CONFIG (an environment variable)
* ansible.cfg (in the current directory)
* .ansible.cfg (in the home directory)
* /etc/ansible/ansible.cfg
```

ansible 1.5之前的处理优先级顺序是：

```
* ansible.cfg (in the current directory)
* ANSIBLE_CONFIG (an environment variable)
* .ansible.cfg (in the home directory)
* /etc/ansible/ansible.cfg
```

这一点了解即可，本文测试使用的是ansible 2.0.0.2。

按照优先级顺序，找到的第一个配置文件会被ansible加载，其他的配置文件不会被合并处理。

2 最新配置
---

如果你的ansible是使用包管理器安装的，缺省的配置文件在`/etc/ansible`中，名为”.rpmnew” 的文件作为修改的参考。

如果是通过 pip 或 源码编译安装，你需要自己创建这个文件以覆盖缺省的配置。

最新的配置，可以参考 [基于版本控制的ansible.cfg文件](https://raw.github.com/ansible/ansible/devel/examples/ansible.cfg)。

3 环境变量配置
---

通过环境变量修改配置，将会覆盖所有从配置文件中加载的配置信息。可以设置的环境变量限于篇幅不做罗列，可参考：https://github.com/ansible/ansible/blob/devel/lib/ansible/constants.py

如：

```bash
export YAML_FILENAME_EXTENSIONS=yaml
```

4 配置说明
---

配置文件类似INI文件格式，被分为若干片段，大部分的配置都配置在[defaults]内。

### 4.1 通用配置 - defaults

在 `ansible.cfg` 的`defaults`片段内，有如下的可选配置：

host_key_checking

详见 2.5

inventory

缺省的被控节点清单文件位置，在ansible 1.9之前，该配置项是 `hostfile`。

```
inventory = /etc/ansible/hosts
```

log_path

日志目录

```
log_path=/var/log/ansible.log
```

module_lang

设置模块与系统通信的缺省的语言环境，缺省是 `C`.

```
module_lang = en_US.UTF-8
```

可配置的项目很多，具体可参考 [ansible官方配置文档](http://docs.ansible.com/ansible/intro_configuration.html)。

3 Inventory被控节点配置
---

创建或编辑 `/etc/ansible/hosts` ，作为被控节点服务器的主清单文件。

#### 3.1 基本配置
```
192.168.33.10

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

方括号中的字符串是组名，它用来对系统进行分类，以决定你要什么时间控制哪些系统系统，以及控制的目的。

你可以把一个实例分配到一个以上的组中，比如一个服务器实例既在webservers中，又在dbservers中。

#### 3.2 配置特殊端口

如果server实例的ssh端口不是标准端口22，那么可以在定义时配置端口号，与ip地址或主机名用冒号分隔即可，如：

```
badwolf.example.com:5309
```

#### 3.3 配置别名

如果你有静态IP地址，并且想给它设置一个别名，如下设置：

```
jumper ansible_port=5555 ansible_host=192.168.1.50
```

那么使用ansible连接jumper，就可以连接到192.168.1.50:5555

#### 3.4 主机的行为参数

你还可以设置每一个主机的连接类型：

```
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=mpdehaan
other2.example.com     ansible_connection=ssh        ansible_user=mdehaan
```

设置主机连接行为的参数有：

```
Host connection: ansible_connection
SSH connection: ansible_host,ansible_port,ansible_user,ansible_ssh_pass,ansible_ssh_private_key_file,ansible_ssh_common_args,ansible_sftp_extra_args,ansible_scp_extra_args,ansible_ssh_extra_args,ansible_ssh_pipelining
Privilege escalation: ansible_become,ansible_become_method,ansible_become_user,ansible_become_pass
Remote host environment parameters: ansible_shell_type,ansible_python_interpreter,ansible\_\*\_interpreter
```

限于篇幅，更多信息请参阅：http://docs.ansible.com/ansible/intro_inventory.html#id10

#### 3.5 配置一组服务器

批量配置一组服务器,可以使用近似正则表达式来描述：

```
[webservers]
www[01:50].example.com
```

#### 3.6 配置Host变量

先大概了解，这些暂时无用，以后会在playbooks中使用。

```
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

#### 3.7 配置Group变量

组变量会一次性把变量赋给组中的所有成员。

```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

#### 3.8 配置组的组

组的组的名字，用 `:children` 后缀标注。

```
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

#### 3.9 拆分主机、组配置和特殊数据

官方推荐，不要把变量保存在被控节点的主配置文件中，而是保存在与主清单文件相对独立的变量文件中。这个变量文件使用YAML格式，扩展名可以是`yml`,`yaml`,`json`,或者没有扩展名。yaml的语法参见：http://docs.ansible.com/ansible/YAMLSyntax.html

假设 被控主机的主清单文件为：`/etc/ansible/hosts`, 如果主机被命名为 `foosball`,它归属于 `raleigh` 和 `webservers` 中,那么yaml文件中的变量，像下面这样是有效的。

```
/etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

`raleigh` 组的组变量文件 `/etc/ansible/group_vars/raleigh` 中的内容，类似于下面的形式：

```
---
ntp_server: acme.example.org
database_server: storage.example.org
```

这个文件不一定存在，因为这是一个可选的特性。

随着配置的越来越复杂，你可以在以主机或组的名称命名的文件夹中创建不同的目录，ansible会读取这些文件夹中的所有文件。仍以`raleigh`组为例：

```
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
```

	Tips: 这样设置，可以针对不同类型的变量，配置到不同配置文件中，避免了配置在唯一的文件中使文件变得很大。注意这个特性只适用于ansible 1.4+
	Tips: ansible 1.2+ 版本中`group_vars/` 和 `host_vars/`可能同时存在于playbook和inventory主配置文件目录中，如果这样，那么playbook中的配置优先生效。
	Tips: 推荐将inventory主配置文件和变量文件用git进行管理，以便于跟踪配置的变化。

更多请参考 [ansible inventory 官方文档](http://docs.ansible.com/ansible/intro_inventory.html)。

