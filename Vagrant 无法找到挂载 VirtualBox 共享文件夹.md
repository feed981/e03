# Vagrant 无法找到挂载 VirtualBox 共享文件夹

```bash 
vagrant reload

Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000,_netdev vagrant /vagrant

The error output from the command was:

/sbin/mount.vboxsf: shared folder '/home/vagrant/vagrant' was not found (check VM settings / spelling)
```

1. 共享文件夹设置不正确：在 Vagrantfile 中配置的共享文件夹可能存在拼写错误或路径问题。
```ruby
# 确保 Vagrantfile 中的路径和配置准确
config.vm.synced_folder "D:/local/nginx", "/vagrant/nginx"
```

2. VirtualBox Guest Additions 未正确安装：错误提示中提到的 vboxsf 文件系统是 VirtualBox Guest Additions 提供的。如果 Guest Additions 没有安装或未正确配置，系统将无法识别 vboxsf 文件系统

```bash
# 查看 Guest Additions 的状态，如果没有输出，说明 Guest Additions 未加载或安装有问题
lsmod | grep vboxguest
vboxguest             356352  2 vboxsf

# 手动安装或更新 Guest Additions
sudo apt update
sudo apt install -y virtualbox-guest-dkms virtualbox-guest-utils

sudo mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant
/sbin/mount.vboxsf: shared folder '/home/vagrant/vagrant' was not found (check VM settings / spelling)
```
3. 有可能是默认挂载设置导致了这个问题

- 出现 /home/vagrant/vagrant 这个路径提示可能是因为：

- Vagrant 配置与 VirtualBox 共享文件夹管理器的配置冲突：默认的同步路径 /vagrant 可能被重新识别为另一个路径。
Vagrant 版本或 Box 的问题：有些 Box（例如较旧版本的 Ubuntu 或其他不支持 vboxsf 文件系统的 Box）在默认同步文件夹设置上会出现兼容性问题。

```ruby
# 禁用默认同步文件夹： 在 Vagrantfile 中加入以下行来禁用默认挂载：
config.vm.synced_folder ".", "/vagrant", disabled: true

# 自定义同步文件夹路径： 使用一个明确的目录指定共享文件夹，例如 /vagrant/nginx。在 Vagrantfile 中指定清晰的源路径和目标路径：
config.vm.synced_folder "D:/local/nginx", "/vagrant/nginx"
```

4. 重新启动 Vagrant： 关闭并重新启动虚拟机以应用新的共享文件夹配置
```bash
vagrant halt
vagrant up
```
