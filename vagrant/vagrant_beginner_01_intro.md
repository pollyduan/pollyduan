官网http://www.vagrantup.com/

官方下载地址：https://www.vagrantup.com/downloads.html

旧版本下载：https://releases.hashicorp.com/vagrant/

box下载：

官方仓库：https://atlas.hashicorp.com/boxes/search

官方镜像：https://vagrantcloud.com/boxes/search

第三方仓库：http://www.vagrantbox.es/

1 什么是vagrant？
---

用官方的定义，vagrant——让搭建开发环境如此简单。它可以为我们搭建和配置轻量级的、可重用的、可移植的开发环境。通俗地讲，它是一个用来简化虚拟机配置和管理的命令行工具。

vagrant支持VirtualBox, VMware, AWS以及其他任何虚拟机（这话说的够大，不过也八九不离十）。

先来约定几个关键词：

host——宿主，主机，也就是你安装虚拟机软件和vagrant的操作系统；

guest/vm——虚拟机，客户机，也就是我们要制作的虚拟机环境。

2 安装vagrant
---

根据你的操作系统下载vagrant安装包。

目前支持的host有Mac osx、windows、debian、centos，当然也支持ubuntu、redhat、fedora了，你懂的。

截至本文完成，最新版本是1.8.1。

自行安装虚拟机管理软件，我使用的是mac osx + virtualbox（http://www.virtualbox.org）

仓库里的box镜像下载有点慢，如果下载不了，我放了个centos的box文件在度娘云里，http://pan.baidu.com/s/1XkmEM

3 增加镜像
---

将镜像添加到本地仓库，有三种方式：

3.1 使用http绝对地址

```
vagrant box add precises64 http://files.vagrantup.com/precise64.box
```

3.2 使用本地文件（其实从协议来说，和上面一样，只是相当于file:///协议的地址)

```
vagrant box add precises64 ./precise64.box
```

3.3 使用仓库名称

```
vagrant box add precises64 ubuntu/precise64
```

这种方式，vagrant会自动在中央仓库查找并下载到本地镜像库中。

`vagrant box add ubuntu/precise64`

这样省略本地镜像名称，则直接用中央仓库中的镜像名作为本地镜像名，这样做的好处是可以跟仓库中的镜像对应。

最后，执行下面的命令看一下：

```
$ vagrant box list
ubuntu/precise64                     (virtualbox, 20160120.0.0)
```

4 创建虚拟机
---

类似于增加本地仓库镜像的操作，该操作使用镜像创建一个vm虚拟机。

4.1 使用http绝对地址

```
$ mkdir -p ~/vm/precise64
$ vagrant init http://files.vagrantup.com/precise64.box
```

该操作只是使用该仓库名创建Vagrantfile文件，并不拉取镜像，在vagrant up的时候才会拉取镜像。

4.2 使用仓库名称

```
vagrant init ubuntu/precise64
```

4.3 使用Vagrantfile文件
直接编写，或通过http/git从网络上拉取到Vagrantfile文件后，作为虚拟机配置文件。如：

```
$ mkdir -p ~/vm/coreos;cd !$
$ git clone https://github.com/coreos/coreos-vagrant.git
```


5 Helloword
---

```
$ mkdir -p /Users/pollyduan/vm/ubuntu
$ cd /Users/pollyduan/vm/ubuntu
$ vagrant init ubuntu/precise64
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/precise64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/precise64' is up to date...
==> default: Setting the name of the VM: ubuntu_default_1455425367263_71317
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 4.1.44
    default: VirtualBox Version: 5.0
==> default: Mounting shared folders...
    default: /vagrant => /Users/pollyduan/vm/ubuntu

$ vagrant ssh
Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 3.2.0-97-virtual x86_64)

vagrant@vagrant-ubuntu-precise-64:~$

```

现在，你已经在vm虚拟里面了。你可以使用软件管理工具进行安装环境。如ubuntu里的apt-get、centos里的yum或者新版fedora里的dnf

还有一种方式，就是自己编写Vagrantfile文件，前面的例子，去掉注释，就是一个最简单的vagrant file的例子 - hello vagrant，vagrant init命令的作用就在于此：

```
$ vi /Users/pollyduan/vm/ubuntu/Vagrantfile

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/precise64"
end
```

启动试试看：

```
$ vagrant up
$ exit #退出vm
$ vagrant halt #关闭虚机
$ vagrant destroy #删除虚机
```

创建的虚机工作目录在用户目录下的 ~/VirtualBox VMs/ 里。

本文涉及的基本使用，一般情况下够用了，更多待续。