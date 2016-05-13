# Vagrant

## 好处

在开发分和测试布式系统的时候，我们经常希望能在本地（host机器上）建立一个虚拟机群（guests）。我们可以利用本机安装的虚拟机软件的GUI或者command line tools来一个一个地创建虚拟机，一个一个地配置网络，把它们连接起来。而另一种更高效的方式是写一个Vagrantfile来描述我们的机群，然后用Vagrant来创建和管理这些虚拟机。

Vagrant解决的第二个问题是：不同的机器上安装的虚拟机软件可能不同，比如有的安装了VirtualBox，有的是VMWare。但是只要我们用Vagrant来创建和管理虚拟机群，Vagrant替我们操心怎么和VirtualBox上和VMWare打交道。

Vagrant解决的第三个问题是：我们可以把自己的虚拟机保存成磁盘文件（虚拟机镜像文件），然后发布到网上，让其他人下载后执行。虽然不同的虚拟机软件提供不同的镜像文件格式，Vagrant使用一个统一的文件格式，称为box。

## 用法

一般的用法是：

1. 在host机器上创建一个目录，在里面放一个Vagrantfile。通常我们通过执行vagrant init命令来生成一个很简单的Vagrantfile，然后编辑修改之。Vagrantfile实际上是一个Ruby源程序文件，里面调用一些预制的函数，来说明我们需要的虚拟机的数量和配置。其中包括每台虚拟机的box的URL。
1. vagrant up命令来下载所需的boxes，并且启动虚拟机（群）（guests）。
1. vagrant status列出按照Vagrantfile描述启动的所有guests。
1. vagrant ssh <vm_name> 来登录进特定的某台guest。
1. vagrant halt <vm_name> 关机。
1. vagrant destroy 关闭和删除所有guest。注意，这不会删除下载到本地的boxes。

## 坑

1. 可以把host的某个目录映射到guest里。有几种方式映射，其中最常用的是通过NFS。这需要host上有nfsd。Mac OS X启动时会启动nfsd。Linux里很容易安装nfsd。确定host上nfsd在运行之后，只需要在Vagrantfile里加一行：

```
config.vm.synced_folder ".", "/home/core/share", nfs: true, mount_options: ['nolock,vers=3,udp']
```

即可把host上的目录`./`（Vagrantfile所在目录）映射到guest的`/home/core/share`。如果用vagrant ssh登录到guest里，并且往`/home/core/share`里写文件，那么在host上可以看到创建的文件。

## 参考材料

* 官网帮助：https://www.vagrantup.com/docs/
* Vagrant使用简介：http://xuclv.blog.51cto.com/5503169/1239250
* 使用Vagrant（零）——为什么要使用Vagrant：http://www.ituring.com.cn/article/131600
