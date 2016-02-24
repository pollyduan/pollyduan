Playbooks 是ansible的配置、开发和编排语言，它可以描述希望远程系统执行的规则或者一系列IT处理步骤。如果把ansible比喻为车间，那么playbooks就是设计方案。

官方推荐在学习的时候，边看[官方文档](http://docs.ansible.com/ansible/playbooks.html)，边阅读[官方的实例](https://github.com/ansible/ansible-examples)。

playbooks遵循yaml语法，它不是一种程序语言，只是一个配置和处理的模型。每一个playbook由一个或更多的`plays`的列表组成的集合。

1 yaml文档
===

列表(list)和字典(dictionary)是yaml文档的基本组成元素。

对于ansible而言，每一个yaml文档都是以一个列表开始的，每一个列表的项目，都是一个字典的列表。

每一个ansible yaml文档都是以 `---` 开始，以 `...` 结束的。

每一个列表都是以`"- "` (**一个减号+一个空格**) 一行的开始。

如:

```
---
# A list of tasty fruits
- fruits:
  Apple
  Orange
```

一个字典数据是一个 `key: value` 键值对，**冒号后面必须跟一个空格**。如定义一个员工的列表：

```
---
# An employee record
- martin:
  name: Martin D'vloper
  job: Developer
  skill: Elite
```

2 文件格式
===

每个yaml文件都是以 `---` 为首行，表示这是一个yaml文档。

每一个yaml文件都是一个play的list列表。

每一个play是一个hash的list列表。hash包括hosts、name、remote_user以及tasks list列表。

每一个tasks也是一个hash的list列表，包括:name、<actions>、notify、handler、roles、remote_user等。


3 基本配置
===

3.1 主机和用户名
---

playbooks中的每一个play，都需要选择要操控哪些主机，以及以哪个用户身份来运行这些任务。

```
---
- hosts: webservers
  remote_user: root
```

除了针对主机指定用户，也可以为单个任务指定不同的用户。

```
---
- hosts: webservers
  remote_user: root
  tasks:
    - name: test connection
      ping:
      remote_user: yourname
```

ansible也支持一个用户以另一个用户身份来运行一个play。

```
---
- hosts: webservers
  remote_user: yourname
  sudo: yes
```

你还可以针对特定的任务，使用sudo:

```
---
- hosts: webservers
  remote_user: yourname
  tasks:
    - service: name=nginx state=started
      become: yes
      become_method: sudo
```

如果切换到非root用户，使用：

```
---
- hosts: webservers
  remote_user: yourname
  become: yes
  become_user: another_user
```

提权方法，除了sudo之外，还有其他的诸如：su

假设你需要指定sudo密码，请在使用`ansible-playbook`命令行时指定：`--ask-become-pass`开关，或者使用旧版本的参数：`--ask-sudo-pass (-K)`。

如果你在运行一段become playbook时，看起来挂起了，很有可能因为你的sudo操作需要输入密码，play挂在了输入密码的提示行。这时候，你只需要按下`Ctrl+C` 指定要求sudo密码开关并重新运行即可。

3.2 任务列表
---

每一个play是一个tasks的列表。

任务(tasks)是有序运行的，同一时刻只运行一个，在进入下一个任务之前会遍历`host pattern`匹配到的所有机器。

当运行playbook的时候是自上而下运行的，任务执行失败的主机会从循环中移除。如果某些指令执行失败，只需要简单地修改playbook文件重新运行即可。

每一个任务的目标就是执行一个制定了参数的模块，前面提到的变量可以在模块参数中使用。

模块是幂等的，这意味着如果你再次运行，他只会处理改变了状态的主机。

每一个tasks应该有一个名字，用name参数指定。如果没有指定，则会把`action`指定为name。

模块可以声明为`action: module options`格式，但官方推荐更常见的方式`module: options`。

一个基本的模块式用看起来这样：

```
tasks:
  - name: make sure apache is running
    service: name=httpd state=running
```

像大部分模块一样，`service`模块带有`key=value`形式的参数。

`command`和`shell`是唯一提供参数列表，而不是`key=value`形式参数的模块。

```
tasks:
  - name: disable selinux
    command: /sbin/setenforce 0
```

`command`和`shell`模块关心返回值，也就是说如果返回值非0，那么tasks会失败。如果你的命令行运行成功时的返回值为非`0`,那么你可能需要这样处理：

```
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand || /bin/true
```

或者指定忽略错误：

```
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand
    ignore_errors: True
```

如果action命令行太长，为了更易读，你可以在空格处换行，并在新行缩进。

```
tasks:
  - name: Copy ansible inventory file to client
    copy: src=/etc/ansible/hosts dest=/etc/ansible/hosts
            owner=root group=root mode=0644
```

Variables变量可以在action命令行中引用，格式为双层大括号包裹关键字，如：

```
tasks:
  - name: create a virtual host file for {{ vhost }}
    template: src=somefile.j2 dest=/etc/httpd/conf.d/{{ vhost }}
```

3.3 action简化
---

前面3.2格式说明的时候提到了，模块有两种声明方式。ansible在`0.8`或更高版本中提供了列表样式模块声明：

```
template: src=templates/foo.j2 dest=/etc/foo.conf
```

而在更早的版本中，你只能如下的样式进行声明：

```
action: template src=templates/foo.j2 dest=/etc/foo.conf
```

Tips: 旧的格式在新版本中会继续使用，官方没有计划废弃。

3.4 Handlers: 在change时运行的操作
---

前面提到过，模块是幂等的，当远程主机发生变化时可获知这个变化。playbooks可以识别它，通过基本的时间机制响应这个变化。

`nofity`行为在playbooks中的每一个任务块末尾被触发，如果一个`notify`被多个不同的任务触发，它只会运行一次。

举例说明，多个资源表明apache需要重启，因为他们修改了apache的配置文件，但playbooks控制apache只被重启一次，以避免不必要的重启操作。

一个notify通知，可以出发两个action行为。

```
- name: template configuration file
  template: src=template.j2 dest=/etc/foo.conf
  notify:
     - restart memcached
     - restart apache
```

`notify`捕获到事件以后，会调用实际的handler事件处理器,他是tasks的列表，与普通的tasks任务没有区别：

```
handlers:
    - name: restart memcached
      service: name=memcached state=restarted
    - name: restart apache
      service: name=apache state=restarted
```

事件处理器最好是用来处理重启服务或重启主机。

4 include引入文件
---

理论上是可以编写一个很长的playbooks文件的，事实上为了让文件更易读，或者某些模块被重用的考虑，我们可以把一个大文件切割为小文件，并使用`include`模块将多个文件合并为一个playbooks文件。

使用任务引用，你可以把tasks文件打碎为小文件，并在`tasks`配置项中进行引入。既然hanlder也是一个tasks，那么在`handlers`项中使用 `include`也是没有问题的。

ansible也可以从其他playbooks文件中引入plays。

当你开始综合思考关于tasks、handlers、variables等等这些内容，你就开始构建一个庞大的概念了。你开始思考模型化某些东西是什么，而不是怎样去做某些事情。或者说，它不再是“对这些主机做些什么”，而是“这些主机属于dbservers”，"另一些主机属于dbservers"。而dbservers或webservers需要部署哪些行为不关心，在程序开发的角度来说，这叫封装行为。就好比，你可以开汽车，而无需关心发动机是如何工作的。

ansible中的roles以包含文件和打包各种功能为基础，以构建干净的，可重用的抽闲。这将允许你关注大的场景，只在必要的时候才深入细节。

我们开始理解包含功能，这样roles更有意义，我们的终极目标是理解roles。roles是很强大的功能，你应该在每次编写playbooks时都使用它。

