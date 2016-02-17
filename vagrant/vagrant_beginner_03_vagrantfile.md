《[vagrant学习笔记 - 入门](http://blog.csdn.net/54powerman/article/details/50663054)》中的hello vagrant配置文件，只是最基本的配置，它使用缺省的box配置初始化了一个虚拟机。有时候，我希望对vm做更详尽的配置，比如配置一次创建一组vm，搭建一个mfs的测试环境，他需要一台服务器做mfsmaster，两台服务器做mfs chunk server，一台服务器做metalogger，还有一台服务器做mfs client进行测试。

下面是一组服务测试mfs的vagrant file范例：

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

app_servers = {
    :mfschunk1 => '192.168.33.20',
    :mfschunk2 => '192.168.33.21'
}

Vagrant.configure("2") do |config|
    config.vm.box = "centos64"

    config.vm.define :mfsmaster do |mfsmaster_config|
        mfsmaster_config.vm.network :private_network, ip: "192.168.33.10"
        config.vm.provider :virtualbox do |vb|
            vb.name = "mfsmaster"
            vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
            vb.customize ["modifyvm", :id, "--memory", "1024"]
            vb.customize ["modifyvm", :id, "--cpus", "2"]
            vb.cpus = 2
        end
    end

    app_servers.each do |app_server_name, app_server_ip|
        config.vm.define app_server_name do |app_config|
            app_config.vm.hostname = "#{app_server_name.to_s}.vagrant.internal"
            app_config.vm.network :private_network, ip: app_server_ip
            app_config.vm.provider "virtualbox" do |vb|
                vb.name = app_server_name.to_s
            end
        end
    end

    config.vm.define :metalogger do |metalogger_config|
        metalogger_config.vm.hostname = "metalogger.vagrant.internal"
        metalogger_config.vm.network :private_network, ip: "192.168.33.30"
        metalogger_config.vm.provider "virtualbox" do |vb|
            vb.name = "metalogger"
            vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
            vb.customize ["modifyvm", :id, "--memory", "1024"]
            vb.customize ["modifyvm", :id, "--cpus", "2"]
            vb.cpus = 2
        end
    end

    config.vm.define :mfsclient do |mfsclient_config|
        mfsclient_config.vm.hostname = "mfsclient.vagrant.internal"
        mfsclient_config.vm.network :private_network, ip: "192.168.33.40"
        mfsclient_config.vm.provider "virtualbox" do |vb|
            vb.name = "mfsclient"
        end
    end
end
```

保存Vagrantfile文件以后，执行如下命令查看虚机配置：

```
$ vagrant status
Current machine states:

mfsmaster                 not created (virtualbox)
mfschunk1                 not created (virtualbox)
mfschunk2                 not created (virtualbox)
metalogger                not created (virtualbox)
mfsclient                 not created (virtualbox)
```

执行up命令，批量创建虚机并启动。

```
$ vagrant up
```

如果只想启动一台，执行：

```
$ vagrant up mfsmaster
```

vagrant基础配置
Vagrantfile 配置文件的格式简单介绍一下。

1 文件头
说明文件的格式信息，处理方式等。

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
```

指定使用ruby格式，vi进行编辑的，所有文件都采用这个文件头即可。

2 通用数据
设置一些基础数据，供配置信息中调用。

```
app_servers = {
    :mfschunk1 => '192.168.33.20',
    :mfschunk2 => '192.168.33.21'
}
```

这里是定义一个hashmap，以key-value方式来存储vm主机名和ip地址。

3 配置信息

```
ENV["LC_ALL"] = "en_US.UTF-8"
#指定vm的语言环境，缺省地，会继承host的locale配置
Vagrant.configure("2") do |config|
    # ...
end
```

参数2，表示的是当前配置文件使用的vagrant configure版本号为Vagrant 1.1+,如果取值为1，表示为Vagrant 1.0.x Vagrantfiles，旧版本暂不考虑，记住就写2即可。
本文只对configure 2作说明，旧版本不多解释了。

do … end 为配置的开始结束符，所有配置信息都写在这两段代码之间。
config是为当前配置命名，你可以指定任意名称，如myvmconfig，在后面引用的时候，改为自己的名字即可。

3.1 基本配置信息

3.1.1 镜像

```
config.vm.box = "ubuntu/precise64"
#指定当前vm使用的box镜像名称，值为本地仓库的镜像名。
```

3.1.2 定义vm的configure配置节点

```
config.vm.define :mfsmaster do |mfsmaster_config|
# ...
end
```

表示在config配置中，定义一个名为mfsmaster的vm配置，该节点下的配置信息命名为mfsmaster_config；
如果该Vagrantfile配置文件只定义了一个vm，这个配置节点层次可忽略。

3.1.2.1 vm网络环境配置

vagrant的网络连接方式有三种：

- NAT : 缺省创建，用于让vm可以通过host转发访问局域网甚至互联网。
- host-only : 只有主机可以访问vm，其他机器无法访问它。
- bridge : 此模式下vm就像局域网中的一台独立的机器，可以被其他机器访问。

```
config.vm.network :private_network, ip: "192.168.33.10"
#配置当前vm的host-only网络的IP地址为192.168.33.10
```

host-only 模式的IP可以不指定，而是采用dhcp自动生成的方式，如 :

```
  config.vm.network "private_network", type: "dhcp”
```

```
#config.vm.network "public_network", ip: "192.168.0.17"
#创建一个bridge桥接网络，指定IP
#config.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)"
#创建一个bridge桥接网络，指定桥接适配器
config.vm.network "public_network"
#创建一个bridge桥接网络，不指定桥接适配器
```

配置当前项以后，如果host有多个网络适配器，第一次启动会询问桥接到哪个网络，如：

```
==> mfsmaster: Available bridged network interfaces:
1) en1: Wi-Fi (AirPort)
2) en0: 以太网
3) en2: Thunderbolt 1
4) p2p0
==> mfsmaster: When choosing an interface, it is usually the one that is
==> mfsmaster: being used to connect to the internet.
    mfsmaster: Which interface should the network bridge to?
```

我使用的是wifi，选择1，继续。

3.1.2.2 同步文件夹配置
用来让host与vm二者进行文件同步。

```
config.vm.synced_folder "../data", "/vagrant_data"
#设置同步文件夹，让主机与vm中的一个文件夹内容保持一致。
```

缺省地，vagrant会把工作目录映射到vm的/vagrant目录，如果需要增加更多同步文件夹，使用上面的配置，第一个文件夹为host主机的目录，第二个文件夹为vm中的目录。

3.1.2.3 设置主机名

指定vm的hostname，会覆盖vm中/etc/hostsname中的设置。

```
config.vm.hostname = “mfsmaster.vagrant.internal"
```

3.1.2.4 端口转发

指定将host的8080端口请求，转发到vm的80端口，这样访问http://host:8080 就相当于访问http://vm:80了。结合《[vagrant 学习笔记 - 基本命令使用](http://blog.csdn.net/54powerman/article/details/50669807)》中的share 共享http功能，我们就可以做到让internet每个角落的用户访问vm里的http服务了。

```
config.vm.network: forwarded_port, guest: 80, host: 8080
```

guest和host是必须的，还有几个可选属性：

- guest_ip：字符串，vm指定绑定的Ip，缺省为0.0.0.0
- host_ip：字符串，host指定绑定的Ip，缺省为0.0.0.0
- protocol：字符串，可选TCP或UDP，缺省为TCP

3.1.2.5 vm提供者配置

```
config.vm.provider :virtualbox do |vb|
     # ...
end
```

3.1.2.2.1 vm provider通用配置

虚机容器提供者配置，对于不同的provider，特有的一些配置，此处配置信息是针对virtualbox定义一个提供者，命名为vb，跟前面一样，这个名字随意取，只要节点内部调用一致即可。

配置信息又分为通用配置和个性化配置，通用配置对于不同provider是通用的，常用的通用配置如下：

```
vb.name = "centos6"
#指定vm-name，也就是virtualbox管理控制台中的虚机名称，或者说是virtual box工作目录的名字，即~/VirtualBox\ VMs/centos6。如果不指定该选项会生成一个随机的名字，不容易区分。
vb.gui = true
# vagrant up启动时，是否自动打开virtual box的窗口，缺省为false
vb.memory = "1024"
#指定vm内存，单位为MB
vb.cpus = 2
#设置CPU个数
```

3.1.2.2.2 vm provider个性化配置（virtualbox）

上面的provider配置是通用的配置，针对不同的虚拟机，还有一些的个性的配置，通过vb.customize配置来定制。

对virtual box的个性化配置，可以参考：`VBoxManage modifyvm` 命令的使用方法。其他虚机的provider，暂时未做测试。

详细的功能接口和使用说明，可以参考[virtualbox官方文档](http://www.virtualbox.org/manual/)。

```
#修改vb.name的值
v.customize ["modifyvm", :id, "--name", "mfsmaster2"]

#如修改显存，缺省为8M，如果启动桌面，至少需要10M，如下修改为16M：
vb.customize ["modifyvm", :id, "--vram", "16"]

#调整虚拟机的内存
 vb.customize ["modifyvm", :id, "--memory", "1024"]

#指定虚拟CPU个数
 vb.customize ["modifyvm", :id, "--cpus", "2"]

#增加光驱：
vb.customize ["storageattach",:id,"--storagectl", "IDE Controller","--port","0","--device","0","--type","dvddrive","--medium","/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso"]
#注：meduim参数不可以为空，如果只挂载驱动器不挂在iso，指定为“emptydrive”。如果要卸载光驱，medium传入none即可。
#从这个指令可以看出，customize方法传入一个json数组，按照顺序传入参数即可。

#json数组传入多个参数，是否意味着VBoxManage命令一样，同时操作多个属性呢？由此，我们可以做一个测试来验证一下：
v.customize ["modifyvm", :id, "--name", “mfsserver3", "--memory", “2048"]
```

扩展一下，如果创建的虚机很多，vm都混杂在一起，我们都知道virtualbox支持对vm进行分组。要在vagrant使用分组，可以在mfs的vagrantfile中如下自定义：

```
vb.customize ["modifyvm",:id,"--groups",”/mfs"]
```

参数说明：

- 分组名是路径格式，/开始，表示第一级目录，可以指定多级目录，如/mfs/chunk；
- 可以指定多个分组，用逗号分开，如:“/dev,/mfs"

每一个vm创建以后，都会放到mfs分组里。可以在virtualbox管理界面查看。

3.1.3 一组相同配置的vm

前面配置了一组vm的hash map，定义一组vm时，使用如下节点遍历。

```
#遍历app_servers map，将key和value分别赋值给app_server_name和app_server_ip
app_servers.each do |app_server_name, app_server_ip|
     #针对每一个app_server_name，来配置config.vm.define配置节点，命名为app_config
     config.vm.define app_server_name do |app_config|
          # 此处配置，参考3.1.2 config.vm.define
     end
end
```

如果不想定义app_servers，下面也是一种方案:

```
(1..3).each do |i|
        config.vm.define "app-#{i}" do |node|
        app_config.vm.hostname = "app-#{i}.vagrant.internal"
        app_config.vm.provider "virtualbox" do |vb|
            vb.name = app-#{i}
        end
  end
end
```

3.2 中央仓库配置

指定box镜像push发布的地址，供box镜像管理者使用。普通使用者不需关心。

```
config.push.define "atlas" do |push|
     push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
end
```

3.3 vm部署

用来加载box以后，对vm的环境进行一些定制，比如设置环境变量，安装软件，部署程序等。如：

```
config.vm.provision "shell", inline: <<-SHELL
   sudo apt-get update
   sudo apt-get install -y apache2
 SHELL
```

这部分内容不少，待续。

3.4 其他

还有很多很多配置，暂时没用到，待续。

详细的文档可参考：
http://informatica.uv.cl/~gabriel/docs/manual_Virtual_Box_html/ch08.html#idp21992144