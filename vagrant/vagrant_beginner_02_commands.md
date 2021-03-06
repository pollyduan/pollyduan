vagrant基本命令，根据操作的目的，可以对基本命令进行分类：

1 操作镜像

box package

2 操作虚拟机

connect destroy halt init powershell provision rdp reload resume share snapshot ssh suspend up

3 监控虚拟机

global-status ssh-config port status

4 其他

help login plugin push version

1 操作镜像
---

该命令有两个，用来管理本地镜像

### 1.1 vagrant box

#### 1.1.1 添加镜像到本地仓库

`$ vagrant box add [box-name] [box镜像文件或镜像名]`

#### 1.1.2 移除本地镜像

`$ vagrant box remove [box-name]`

##### box多版本共存的情况

如果box升级过，那么在box list中会出现两个同名，但版本不同的镜像。如：

```
$ vagrant box list |grep coreos
coreos-alpha                         (virtualbox, 745.1.0)
coreos-alpha                         (virtualbox, 928.0.0)
```

使用该镜像创建虚拟机的时候，默认会使用高版本的box。

如果想使用低版本，需要修改Vagrantfile,指定box-version:

在config.vm.box=xxx下一行，如上面的例子中，在“config.vm.box = "coreos-alpha"”后面增加一行配置信息：
config.vm.box_version = "745.1.0"

同样，如果想删除一个box，如下操作会失败：

```
$ vagrant box remove coreos-alpha
You requested to remove the box 'coreos-alpha' with provider
'virtualbox'. This box has multiple versions. You must
explicitly specify which version you want to remove with
the `--box-version` flag or specify the `--all` flag to remove all
versions. The available versions for this box are:

 * 745.1.0
 * 928.0.0
```

这时有两个选择：

- vagrant box remove coreos-alpha --all

删除所有同名镜像

- vagrant box remove coreos-alpha --box-version=745.1.0

删除指定版本的镜像


#### 1.1.3 升级镜像

检查镜像是否有升级？

```
$ cd ~/vm/ubuntu
$ vagrant box outdated
Checking if box 'ubuntu/trusty64' is up to date...
A newer version of the box 'ubuntu/precise64' is available! You currently
have version '20160122.0.0'. The latest is version '20160209.0.0'. Run
`vagrant box update` to update.
```

中央仓库有新版更新了，手动更新box。更新的结果并不是替换旧版本，而是在本地仓库中增加了新版的box镜像。

```
$ cd ~/vm/ubuntu
$ vagrant box update
==> default: Checking for updates to 'ubuntu/precise64'
    default: Latest installed version: 20160120.0.0
    default: Version constraints:
    default: Provider: virtualbox
==> default: Updating 'ubuntu/precise64' with provider 'virtualbox' from version
==> default: '20160120.0.0' to '20160201.0.0'...
==> default: Loading metadata for box 'https://atlas.hashicorp.com/ubuntu/precise64?access_token=cXR0wCgWXoRpMw.atlasv1.ydBAS4ev1YCWzSK4S1l6iVjssRbO5Q50a8YVnEPqoyYjcQVeaMdEsiQ8rz8tOcSHLuY'
==> default: Adding box 'ubuntu/precise64' (v20160201.0.0) for provider: virtualbox
    default: Downloading: https://atlas.hashicorp.com/ubuntu/boxes/precise64/versions/20160201.0.0/providers/virtualbox.box
==> default: Successfully added box 'ubuntu/precise64' (v20160201.0.0) for 'virtualbox'!

$ vagrant box list | grep precise64
ubuntu/precise64                     (virtualbox, 20160120.0.0)
ubuntu/precise64                     (virtualbox, 20160201.0.0)
```

也可以检查本地仓库中的所有镜像是否有升级,使用 --global 开关，这时候不需要进入工作目录

```
$ vagrant box outdated --global
* 'ubuntu/trusty64' is outdated! Current: 20160122.0.0. Latest: 20160209.0.0
* 'ubuntu/precise64' (v20160201.0.0) is up to date
* 'pollyduan/bento_oracle_xe' wasn't added from a catalog, no version information
* 'phusion/ubuntu-14.04-amd64' (v2014.04.30) is up to date
* 'hashicorp/precise32' (v1.0.0) is up to date
* 'hashicorp/boot2docker' (v1.7.8) is up to date
* 'debian/jessie64' wasn't added from a catalog, no version information
* 'coreos-alpha' is outdated! Current: 928.0.0. Latest: 955.0.0
* 'centos7' wasn't added from a catalog, no version information
* 'centos65' wasn't added from a catalog, no version information
* 'centos64' wasn't added from a catalog, no version information
* 'bento/ubuntu-14.04' (v2.2.3) is up to date
* 'bento/opensuse-13.2-x86_64' (v2.2.1) is up to date
* 'bento/freebsd-10.2' (v2.2.3) is up to date
* 'bento/fedora-22' (v2.2.3) is up to date
* 'bento/debian-8.2' (v2.2.3) is up to date
* 'bento/centos-7.2' (v2.2.3) is up to date
* 'bento/centos-6.7' (v2.2.3) is up to date
```

不进入工作目录进行升级

```
$ vagrant box update --box coreos-alpha
Checking for updates to 'coreos-alpha'
Latest installed version: 928.0.0
Version constraints: > 928.0.0
Provider: virtualbox
Updating 'coreos-alpha' with provider 'virtualbox' from version
'928.0.0' to '955.0.0'...
Loading metadata for box 'http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json'
Adding box 'coreos-alpha' (v955.0.0) for provider: virtualbox
Downloading: http://alpha.release.core-os.net/amd64-usr/955.0.0/coreos_production_vagrant.box
Box download is resuming from prior download progress
Progress: 1% (Rate: 92912/s, Estimated time remaining: 0:38:09)
...
```

### 1.2 打包镜像

创建vm以后，我们自己根据需要安装软件，配置环境，都一切就绪了。如何分发给小伙伴使用呢？这里就要涌到package命令了，把镜像打包分发。


```$ ll ~/VirtualBox\ VMs/ |grep ubuntu
drwx------  6 pollyduan  staff  204  2  2 11:18 ubuntu_default_1453944793418_7699
```

先看一下我们的vm目录名，这里有个容易混淆的目录：

- Vagrantfile所在的目录——vagrant的工作目录

- 虚拟机文件所在的目录——virtualbox的工作目录

这两个目录名不一定相同，如果在Vagrantfile中指定了vb.name，virtualbox工作目录就取这个名字；否则，命名为：vagrant工作目录_随机字符串。

或者，也可以使用virtual box的管理工具来看vm的名称。

```
$ VBoxManage list vms|grep ubuntu
"ubuntu_default_1453944793418_7699" {0362edc2-548b-400b-a55d-776b0a24cd8d}
```

package命令要使用的是virtual box工作目录。

	格式：vagrant package --base [virtualbox的工作目录] --output [保存的文件名，缺省为package.box]

```
$ vagrant package --base ubuntu_default_1453944793418_7699 --output ubuntu_myproject_test.box
==> ubuntu_default_1453944793418_7699: Clearing any previously set forwarded ports...
==> ubuntu_default_1453944793418_7699: Exporting VM...
==> ubuntu_default_1453944793418_7699: Compressing package to: /Users/pollyduan/vm/tmp/ubuntu_myproject_test.box
```

导出后，可以通过IM、ftp或其他方式分发给小伙伴，那么大家使用的环境就是一致的了。

2 操作虚拟机
--

### 2.1 启动vm

#### 2.1.1 对于单虚拟机

```
$ vagrant up
```

#### 2.1.2 如果同一个Vagrantfile定义了一个以上的虚拟机，则：

```
$ vagrant up [vm-name]
```

其他命令类似。如果省略vm-name，则依次启动所有vm。

### 2.2 重启

```
$ vagrant reload [vm-name]
```

### 2.3 关机

```
$ vagrant halt [vm-name]
```

### 2.4 销毁虚拟机

```
$ vagrant destroy [vm-name]
```

### 2.5 ssh登录虚拟机

```
$ vagrant ssh [vm-name]
```

### 2.6 休眠与唤醒

这一对冤家也无需多说，对于开发环境来说，个人觉得用处不是很大。

```
$ vagrant suspend
==> default: Saving VM state and suspending execution...
$ vagrant status
Current machine states:

default                   saved (virtualbox)

To resume this VM, simply run `vagrant up`.
$ vagrant resume
==> default: Resuming suspended VM...
==> default: Booting VM...
…...
```

### 2.7 快照

vagrant snapshot命令是vm的月光宝盒，如果vm中有任务没有跑完，需要关闭virtual box，就可以给vm做一个快照，保存vm当前所有的状态，在virtualbox重新启动后，再回复快照。

#### 2.7.1 查看当前保存的快照

```
$ vagrant snapshot list
==> default: No snapshots have been taken yet!
```

#### 2.7.2 创建一个命名快照

```
$ vagrant snapshot save shot1
==> default: Snapshotting the machine as 'shot1'...
$ vagrant snapshot list
shot1
```

#### 2.7.3 恢复快照

```
$ vagrant snapshot restore shot1
==> default: Forcing shutdown of VM...
==> default: Restoring the snapshot 'shot1'...
…...
```

恢复后，快照会一直存在，直到你手动删除它。

#### 2.7.4 删除快照

```
$ vagrant snapshot delete shot1
==> default: Deleting the snapshot 'shot1'...
Progress: 90%
Progress: 100%
==> default: Snapshot deleted!
```

这个操作会删除持久化的数据文件，稍微有点慢，耐心等待。这个内在的原理没有深入研究，有点不太理解，删除一个文件理论上应该比保存一个文件更快才对。

#### 2.7.5 盗梦空间

push和pop，每调用一次push命令会自动创建一个命名快照，名为：push+一串随机数，如：push_1455524411_6632；每调用一次pop，会逐级恢复到最新的快照，并删除快照。看例子：

```
$ vagrant snapshot list
==> default: No snapshots have been taken yet!
$ vagrant snapshot push
==> default: Snapshotting the machine as 'push_1455525041_2882'...
$ vagrant snapshot list
push_1455525041_2882
$ vagrant snapshot push
==> default: Snapshotting the machine as 'push_1455525049_7456'...
$ vagrant snapshot push
==> default: Snapshotting the machine as 'push_1455525058_6891'...
$ vagrant snapshot list
push_1455525041_2882
push_1455525049_7456
push_1455525058_6891
$ vagrant snapshot pop
==> default: Forcing shutdown of VM...
==> default: Restoring the snapshot 'push_1455525058_6891'...
==> default: Deleting the snapshot 'push_1455525058_6891'...
==> default: Snapshot deleted!
$ vagrant snapshot list
push_1455525041_2882
push_1455525049_7456
```

Tips: 文本的日志看起来还不够形象，在push三个snapshot后在virtual box中是树形显示的；每次pop，树枝会逐级退回，看起来更像穿越的感觉。
Tips: 在pop清空之前，随时可以通过restore恢复其中一个快照，同样快照不会删除；不影响pop的测试。

### 2.8 远程连接分享

远程连接通过share connect两个命令可以实现通过本机vagrant连接另外一台host上的虚机。

#### 2.8.1 允许ssh连接

```
$ vagrant share --ssh
==> default: Detecting network information for machine...
    default: Local machine address: 127.0.0.1
    default:
    default: Note: With the local address (127.0.0.1), Vagrant Share can only
    default: share any ports you have forwarded. Assign an IP or address to your
    default: machine to expose all TCP ports. Consult the documentation
    default: for your provider ('virtualbox') for more information.
    default:
    default: An HTTP port couldn't be detected! Since SSH is enabled, this is
    default: not an error. If you want to share both SSH and HTTP, please set
    default: an HTTP port with `--http`.
    default:
    default: Local HTTP port: disabled
    default: Local HTTPS port: disabled
    default: SSH Port: 2200
    default: Port: 2200
==> default: Generating new SSH key...
    default: Please enter a password to encrypt the key: [输入授权密码]
    default: Repeat the password to confirm:[再输一次]
    default: Inserting generated SSH key into machine...
==> default: Checking authentication and authorization...
==> default: Creating Vagrant Share session...
    default: Share will be at: vile-ibex-8238
==> default: Your Vagrant Share is running! Name: vile-ibex-8238
==> default:
==> default: You're sharing your Vagrant machine in "restricted" mode. This
==> default: means that only the ports listed above will be accessible by
==> default: other users (either via the web URL or using `vagrant connect`).
==> default:
==> default: You're sharing with SSH access. This means that another user
==> default: simply has to run `vagrant connect --ssh vile-ibex-8238`
==> default: to SSH to your Vagrant machine.
==> default:
==> default: Because you encrypted your SSH private key with a password,
==> default: the other user will be prompted for this password when they
==> default: run `vagrant connect --ssh`. Please share this password with them
==> default: in some secure way.
```

Tips: 你可以通过--name指定一个名称，否则会随机生成一个共享名，如本例中的vile-ibex-8238

#### 2.8.2 连接远端ssh虚机

```
$ vagrant connect --ssh vile-ibex-8238 --static-ip 10.2.136.211
Loading share 'vile-ibex-8238'...
The SSH key to connect to this share is encrypted. You will require
the password entered when creating to share to decrypt it. Verify you
access to this password before continuing.

Press enter to continue, or Ctrl-C to exit now.[回车]
Password for the private key:[输入授权密码]
Executing SSH...
vagrant@vagrant-ubuntu-trusty-64:~$
```

#### 2.8.3 共享http

vagrant share可以把host主机的http开放到远端，供任何人访问，这好像跟vm没什么关系，但的确它发生了。

```
$ ~/apache-tomcat-8.0.28/bin/startup.sh
$ vagrant share --http 80
==> default: Detecting network information for machine...
    default: Local machine address: 127.0.0.1
    default:
    default: Note: With the local address (127.0.0.1), Vagrant Share can only
    default: share any ports you have forwarded. Assign an IP or address to your
    default: machine to expose all TCP ports. Consult the documentation
    default: for your provider ('virtualbox') for more information.
    default:
    default: Local HTTP port: 80
    default: Local HTTPS port: disabled
    default: Port: 2200
==> default: Checking authentication and authorization...
==> default: Creating Vagrant Share session...
    default: Share will be at: enchanting-buffalo-1493
==> default: Your Vagrant Share is running! Name: enchanting-buffalo-1493
==> default: URL: http://enchanting-buffalo-1493.vagrantshare.com
==> default:
==> default: You're sharing your Vagrant machine in "restricted" mode. This
==> default: means that only the ports listed above will be accessible by
==> default: other users (either via the web URL or using `vagrant connect`).
```

我的host电脑在内网，在外网的任意一台电脑上，访问http://enchanting-buffalo-1493.vagrantshare.com，奇迹发生了。

话说这个有什么用呢？别忘记，vagrant有一个端口映射的功能，在后面的Vagrantfile配置里会提到，这样做的结果，就是可以在互联网任意一个角落可以访问到你的虚机的http服务。

### 2.9 windows相关的操作

powershelgl和rdp是windows vm相关的操作，未做测试，忽略。

### 2.10 虚机环境部署

provision用于通过Vagrantfile配置文件，对vm进行部署，如安装软件，发布应用等，在这里不多说，专门一章来记录。

### 2.11 指定vmid操作虚拟机

在3.3.2中，我们可以看到当前工作机中的所有虚机，其中第一列数据为vmid，我们可以无需进入vagrant工作目录，操作这些虚机。如：

```
$ vagrant up 63093ce
```

该方式适用于前面提到的up、reload、halt、destroy等命令。

3 监控虚拟机
---

### 3.1 查看sshd配置信息

```
$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile "/Users/pollyduan/vm/ubuntu/.vagrant/machines/default/virtualbox/private_key"
  IdentitiesOnly yes
  LogLevel FATAL
```

### 3.2 查看虚拟机开放的端口

```
$ vagrant port
The forwarded ports for the machine are listed below. Please note that
these values may differ from values configured in the Vagrantfile if the
provider supports automatic port collision detection and resolution.

    22 (guest) => 2222 (host)
```

### 3.3 查看虚拟机状态
#### 3.3.1 查看当前vm状态

```
$ cd ~/vm/ubuntu
$ vagrant status
Current machine states:

default                   poweroff (virtualbox)

The VM is powered off. To restart the VM, simply run `vagrant up`
```

状态可能是：

- not create | 执行vagrant init命令后，从未启动过
- poweroff | 关机
- running | 运行中
- saved | 休眠

#### 3.3.2 查看全部虚机状态

此命令无需进入vagrant工作目录。

```
$ vagrant global-status
id       name       provider   state    directory
--------------------------------------------------------------------------------
be5dee2  mfsmaster  virtualbox poweroff /Users/pollyduan/vm/mfs
a523de6  mfschunk1  virtualbox poweroff /Users/pollyduan/vm/mfs
8377e0d  mfschunk2  virtualbox poweroff /Users/pollyduan/vm/mfs
b772b1f  metalogger virtualbox poweroff /Users/pollyduan/vm/mfs
8a5f10e  mfsclient  virtualbox poweroff /Users/pollyduan/vm/mfs
63093ce  default    virtualbox poweroff /Users/pollyduan/vm/ubuntu
```

你可能会发现，为什么有的是default，有的是有名字的。这就是因为mfsxxxx是在vagrantfile中指定了vb.name，他对应的virtualbox工作目录也是这个值，而ubuntu这个虚机没有指定，所以是default，而且其virtualbox工作目录也是比较长的——ubuntu_default_1453944793418_7699。

global-status统计信息不是实时的，所有不能保证数据是绝对准确的。如果在vagrant up启动后，我们在virtualbox管理终端关闭vm，global-status是捕获不到的，它还是会显示running状态。

截至1.8.1还是这样的，应该算是一个bug。处女座可能无法接受这个现实，那么你可以进入vagrant工作目录，手动再指定一次vagrant halt，状态就同步了。

4 其他命令
---

### 4.1 help

略。

### 4.2 login

登录到中央仓库

### 4.3 plugin

插件管理，略。

### 4.4 push

发布镜像到中央仓库

### 4.5 version

略。