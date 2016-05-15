###1. 先快速浏览下官方文档

访问 https://www.vagrantup.com/docs/getting-started/ ，这里介绍的很全面。有快速上手，有命令详细介绍，先快速看一遍有利于后面的实际操作练习。

###2. 确定主机Vagrant的工作目录
这里选择：F:\kubernetes\vagrant,  执行:    
    
    cd F:\kubernetes\vagrant

###3. 初始化
    
    vagrant init hashicorp/precise64

这里需要解释一下，“hashicorp/precise64”表示的是要使用的box完整标识。其中hashicorp是用户名，precise64是box名称。这个命令执行时：hashicorp/precise64会被注入到当前工作目录下的vagrantfile里面。Vagrantfile是用ruby语言写的脚本，如果打开这个文件，去掉注释以后，就是下面这个样子:

<pre><code>
Vagrant.configure(2) do |config|  
    config.vm.box = "hashicorp/precise64"  
end  
</code></pre>

###4. 创建和配置虚拟机
执行
```
vagrant up
```
这个命令会根据Vagrantfile来创建和配置虚拟机。简单的说，vagrant会自动下载并安装虚拟机，并且把它启动。这个命令是vagrant里面最常用的命令。最后的执行结果是vagrant会启动一个虚拟机，并配置好SyncedFolder，同时把SyncedFolder分别在guest和host的什么位置都显示出来。所谓SyncedFolder就是在hostmachine开辟一个目录，默认就是工作目录，这个目录和虚拟机指定的目录（默认是/vagrant)保持文件和文件内容实时同步。

这里需要补充说明的是：在虚拟机挂起或者停止的状态下，vagrant up这个命令不是创建虚拟机，而是启动虚拟机。当把虚拟机删除掉了以后，直接使用vagrant up， 可以重建这个虚拟机。

下面会进行一些交互实验。

###5. 进入虚拟机
首先要通过配置环境变量确保环输入命令行时，能找到ssh.exe这个程序。如果ssh.exe没有，就装一个linux终端，比如git，MinGW, Cygwin等等。这些程序的安装目录下都有ssh.exe这样的文件。然后，进入工作目录，执行：
```
vagrant ssh
```

效果如下：

```
F:\kubernetes\vagrant>vagrant ssh
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
New release '14.04.4 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Welcome to your Vagrant-built virtual machine.
Last login: Sun May 15 03:31:55 2016 from 10.0.2.2
```

###6. 验证SyncedFolder

先说下SyncedFolder的位置：
```
guest： /vagrant
host：  F:\kubernetes\vagrant
```

接着验证两边分别创建文件，可以实时同步

在命令行里面，执行完vagrant ssh以后进入了guest机，然后：
cd  /vagrant
touch foo

可以看到在host的F:\kubernetes\vagrant目录下创建了一个文件foo。反过来，在host上创建文件，test.txt，在控制台里面执行：
ls
可以看到test.txt文件已经被创建。

再验证两边修改文件，可以实时同步，具体操作这里就省略了。


###7. 通过修改Vagrantfile来生成一个预安装软件的box


- 编写软件安装脚本如下：

```
apt-get update  
apt-get install -y apache2  
if ! [ -L /var/www ]; then  
  rm -rf /var/www  
  ln -fs /vagrant /var/www  
fi
```

具体功能是：安装apache2，并创建一个符号链接/war/www指向/vagrant
命令细节不再解释，有问题直接到：http://explainshell.com/ 这里查。


- 修改Vagrantfile

```
Vagrant.configure("2") do |config|
    config.vm.box="hashicorp/precise64"
    config.vm.provision:shell,path: "bootstrap.sh"
end
```

最关键的是：
```
config.vm.provision:shell,path: "bootstrap.sh"
```
意思是虚拟机的provision用shell这个工具，执行的脚本是：bootstrap.sh。其中，provision个人理解是：安装和配置内容的方式。这里指定通过shell安装其他程序，跟着配上shell执行路径："bootstrap.sh"。

- 执行修改后的vagrantfile

如果当下vagrant已经处于运行状态，执行：  
```
vagrant reload --provision
```
如果尚未运行，则执行：  
```
vagrant up
```
这样，我们就得到了一个新的box，且预置了apache，当我们执行：
wget -qO- 127.0.0.1
时，屏幕上会显示apache默认首页的html内容。

###8. 网络配置之端口映射

前面配置了apache，我们希望在host机上面输入一个网址能访问到guest机里面的apache服务。具体操作如下：

- 修改Vagrantfile：

```
Vagrant.configure("2") do |config|
      config.vm.box="hashicorp/precise64"
      config.vm.provision:shell,path: "bootstrap.sh"
      config.vm.network:forwarded_port,guest: 80,host: 4567
end
```

注意增加了这一行：
```
config.vm.network:forwarded_port,guest: 80,host: 4567
```
意思是host机器上的4567端口指向guest机上的80端口

- 执行修改后的vagrantfile
执行：
```
vagrant reload
```

- 待系统自动重启后，在host主机浏览器里面敲入：
```
http://127.0.0.1:4567/
```
会把apache服务器的主页面显示出来。


###9. 打包一个box

就是将现在的虚拟机环境和配置打包成一个box，执行：
```
vagrant package --output first.box
```
output属性表示box文件名称。


###10. 使用这个box
执行：
```
vagrant box add first.box --name first_add.box
```
查看box列表，结果如下：
```
F:\kubernetes\vagrant>vagrant box list
first_add.box       (virtualbox, 0)
hashicorp/precise64 (virtualbox, 1.1.0)
```

###11. 修改Vagrantfile，修改后如下：
```
Vagrant.configure("2") do |config|
  config.vm.box = "first_add"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.network :forwarded_port, guest: 80, host: 4567
end
```

###12. 直接执行
```
vagrant up
```

###13. 发现启动的还是同一个虚拟机
在virtualPC里只看到一个虚拟机

###14. 尝试指定box文件
```
vagrant up first_add
```
返回：
```
The machine with the name 'first_add' was not found configured for
this Vagrant environment.
```
####没有成功！####



####没解决：
没有办法在同一个机器里使用这个创建多first_add.box


###常见问题
<b>1. 提示这样的错误</b>

Stderr: VBoxManage.exe: error: AMD-V is disabled in the BIOS (or by the host OS)
 (VERR_SVM_DISABLED)
VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component ConsoleWrap,
 interface IConsole   
 
```     
回答：设置BIOS: Advanced BIOS Features -> CPU Feature -> Virtualization: Enabled
```

<b>2. 为什么Vagrant示例中的precise64版的box体积那么小？他们是怎么做成的？</b>   
```
回答：The Ubuntu boxes provided by the Vagrant project (such as "precise64") 
are base boxes. They were created from a minimal Ubuntu install from an ISO, 
rather than repackaging an existing environment.  
```
