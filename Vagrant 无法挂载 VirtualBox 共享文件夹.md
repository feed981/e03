# Vagrant 无法挂载 VirtualBox 共享文件夹

```bash 
vagrant reload

...
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

1. VirtualBox Guest Additions 未正确安装：错误提示中提到的 vboxsf 文件系统是 VirtualBox Guest Additions 提供的。如果 Guest Additions 没有安装或未正确配置，系统将无法识别 vboxsf 文件系统

2. 内核模块未加载：即使 Guest Additions 已安装，如果相应的内核模块未加载，也可能会遇到此问题。

3. 共享文件夹设置不正确：在 Vagrantfile 中配置的共享文件夹可能存在拼写错误或路径问题。