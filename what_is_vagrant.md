##1. Vagrant是什么

这是我目前的个人理解。我们把一个虚拟机以及虚拟机里面的所有运行环境称为一个box。
Vagrant里面可以包含多个box，vagrant可以直接管理他们，包括快速安装，删除和枚举等。 
Vagrant可以直接加载box。因为box是描述了唯一的一种环境配置，在不同的地方使用这个box，运行效果不发生改变。这点是Vagrant的一个很大的优点，这样，我们配置好环境以后，生成box，以后即便在线上，运行效果也是一样的。如果我们想对线上的某个box里的服务测试，直接把这个box拉下来，然后用Vagrant启动，然后直接测试就可以了。需要说明的是，box文件不像我们想象的那么大，box文件里面只是包含对操作系统的描述，以及各种配置信息的描述，所有box文件并不大。
Vagrant可以自动生成box文件，也可以加载box文件。另外，通过Vagrant ssh命令，还可以进入box内部的操作系统，进行各种操作。在操作过程中，每当产生配置变化，都会被记录到一个Vagrantfile里面，这个Vagrantfile是未来生成box的主要依据。很自然地，我们可以通过git来管理Vagrantfile，这样，我们就可以追溯对环境配置做得各种改变。


总结一下，Vagrant是多节点集群的很理想的配置管理工具，我们只需做一次配置，Vagrant就可以通过生成box文件帮我们完成多个节点的同步。同时Vagrant和可以控制多个节点的启用和停止。

##2. 参考材料

* 官网帮助：https://www.vagrantup.com/docs/
* Vagrant使用简介：http://xuclv.blog.51cto.com/5503169/1239250
* 使用Vagrant（零）——为什么要使用Vagrant：http://www.ituring.com.cn/article/131600