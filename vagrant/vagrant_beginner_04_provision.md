从字面上来看，provision是准备，实现的功能是在原生镜像的基础上，进行一些附加的操作，以改变虚拟机的环境，比如安装应用，发布程序等。

一 helloword
---

在vagrant的 Vagrant.configure(2) do |config| 节点内，加入如下代码：

```
  config.vm.provision "shell", inline: "echo hello provisio."
```

还有一种格式：

```
  config.vm.provision "shell" do |s|
    s.inline = "echo hello provision."
  end
```

测试一下：

如果vm已经启动，直接运行

```
vagrant provision
```

就可以看到控制台显示的信息了。或者：

```
vagrant reload --provision
```

重启vm，并自动执行provision操作。

Tips: 运行后可能会提示：default: stdin: is not a tty 错误，不影响执行效果，想要去除，在配置文件增加一行即可。

```
config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
```

二 shell
---

### 2.1 基本使用

helloword只是一个开始，对于inline模式，如果脚本较多，可以在Vagrantfile中指定内联脚本，在Vagrant.configure节点外面，写入命名内联脚本：

```
$script = <<SCRIPT
echo I am provisioning...
echo hello provision.
SCRIPT
```

然后，inline调用如下：

```
  config.vm.provision "shell", inline: $script
```

也可以把代码保存在一个shell里，进行调用：

```
  config.vm.provision "shell", path: "script.sh"
```

script.sh的内容：

```
echo hello provision.
```

效果是一样的。

Tips：path可以使用http/https来访问远程脚本，这个在部署时访问一个脚本仓库来做一些通用的操作，比较方便。如：

```
  config.vm.provision "shell", path: "https://example.com/provisioner.sh"
```

Tips: 脚本实际上是在vm里运行的，做个测试验证一下，在Vagrant.configure节点外面，写入命名内联脚本：

```
$script = <<SCRIPT
echo I am provisioning...
date > /etc/vagrant_provisioned_at
SCRIPT
```

检查结果， /etc/vagrant_provisioned_at 这个文件不在host主机里，而是在vm虚机里。

### 2.2 脚本参数

如果要执行的脚本需要参数，那么使用args属性进行传递：

```
  config.vm.provision "shell" do |s|
    s.inline = "echo $1"
    s.args   = "'hello, world!'"
  end
```

Tips: 这两args有两层引号，如果去掉一层，字符串中的逗号会让shell以为是两个参数，那么后面的world会被丢掉。
你也可以使用方括号作为外层分隔符，而内层分隔符使用单引号或双引号都可以，只要前后匹配即可，如：

```
s.args   = ["hello, world!"]
```

Tips: 多个参数，你可以把args理解为一个json 数组，只要在inline里用$x进行引用即可。

### 2.3 环境变量

为命令行指定环境变量，env的格式为hash，是一个hash对象的列表，多个环境变量，多次配置env。
如：

```
  config.vm.provision "shell" do |s|
    s.inline = "echo $1 $JAVA_HOME"
    s.env = {JAVA_HOME:"/opt/java"}
    s.args   = ["java_home is "]
  end
```

执行结果：

```
==> default: java_home is /opt/java
```

多个环境变量的例子：

```
  config.vm.provision "shell" do |s|
    s.inline = "echo $1 $JAVA_HOME $2 $PATH"
    s.env = {PATH:"$JAVA_HOME/bin:$PATH",JAVA_HOME:"/opt/java"}
    s.args   = ["java_home is ",";PATH ="]
  end
```

执行结果：

```
==> default: java_home is /opt/java ;PATH = /bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Tips: 环境变量可以引用已经存在的环境变量，如PATH:"/opt/java/bin:$PATH",结果是在原有的PATH环境变量前面增加了一个路径。

Tips: env新增的环境变量，如JAVA_HOME，系统中原来是没有的，在同时设置的环境变量中也是可以引用的，而且与顺序无关。

三 文件操作
---

file 操作有两个参数：

- source : 源文件
- destination : 目标文件

```
  config.vm.provision "file", source: "./Vagrantfile", destination: "Vagrantfile"
```

将host主机的 "./Vagrantfile" 上传到 vm虚拟机的目标文件 "./Vagrantfile" 。

四 集群管理，自动化配置等系统
---

ansible,cfengine,Chef,puppet
每一套系统都可以写本书了，所以这里不详细说明，另外立题，先占位。

简单来说 Ansible 是一个极简化的应用和系统部署工具，类似 Puppet、Chef、SaltStack。由于默认使用 ssh 管理服务器（集群），配置文件采用 yaml 而不是某一种特定语言制定。
cfengine是一个Linux的自动化配置系统。
Chef 是一套Linux的配置管理系统。

五 Docker 面向容器的虚拟解决方案
---

六 Salt
---

Salt 是一个强大的远程执行管理器，用于快速和高效的服务器管理。

更多，请参考：https://www.vagrantup.com/docs/provisioning/