# 使用 Vagrant 和 VirtualBox 创建 Ubuntu 虚拟机时安装XFCE 桌面环境

Vagrant 启动虚拟机时通常是在无头模式（headless mode），没有可视化的用户界面

## Vagrantfile 启用图形用户界面（GUI）

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64" # 或您选择的其他 Ubuntu 版本

  config.vm.network "private_network", ip: "192.168.33.11"

  # 设置端口转发规则，以便通过 VNC 客户端连接到虚拟机
  config.vm.network "forwarded_port", guest: 5901, host: 5901

  config.vm.provider "virtualbox" do |v|
    v.gui = true  # 启用 GUI ，指示 Vagrant 启动虚拟机时显示 VirtualBox 的用户界面
  end
end
```

```bash
vagrant up
```

## 安装桌面环境 XFCE 

```bash
# 更新软件包列表
sudo apt update

# 安装桌面环境
sudo apt install xfce4 xfce4-goodies

# 安装 VNC 服务器（可选）
sudo apt install tightvncserver
```

## 配置 VNC 服务器

1. 启动 VNC 服务器并设置密码
```
vncserver
```
2. 将 ~/.vnc/xstartup 文件修改为以下内容，以便在启动 VNC 服务器时能够正确加载 XFCE 桌面环境

修改前
```bash
vagrant@ubuntu-bionic:~$ cat .vnc/xstartup
#!/bin/sh

xrdb $HOME/.Xresources
xsetroot -solid grey
#x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#x-window-manager &
# Fix to make GNOME work
export XKL_XMODMAP_DISABLE=1
/etc/X11/Xsession
```

修正后
```bash
vagrant@ubuntu-bionic:~$ sudo nano .vnc/xstartup
#!/bin/sh

xrdb $HOME/.Xresources
startxfce4 &

#xsetroot -solid grey
#作用：这行命令用于设置 VNC 桌面的背景颜色为灰色。
#是否保留：如果您希望在 VNC 会话启动时看到一个灰色背景，可以保留这一行。如果不需要特定的背景颜色，您可以删除它

#x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#x-window-manager &
# Fix to make GNOME work

#export XKL_XMODMAP_DISABLE=1
#作用：这行命令用于禁用 XKB（X Keyboard Extension）中的 Xmodmap 功能，通常是为了避免某些键盘布局问题。
#是否保留：如果您没有遇到与键盘布局相关的问题，可以选择删除这一行。如果您在使用 VNC 时遇到键盘问题，建议保留这一行。


#/etc/X11/Xsession
#作用：这行命令用于启动系统的默认 X 会话，通常会加载系统配置的桌面环境。
#是否保留：如果您使用 startxfce4 & 启动 XFCE 桌面环境，那么可以删除这一行，因为 startxfce4 已经会启动 XFCE 的所有必要组件。如果您希望使用其他桌面环境（如 GNOME），则可能需要保留这一行。
```
3. 设置可执行权限

确保 xstartup 文件具有可执行权限

```bash
chmod +x ~/.vnc/xstartup
```

重启 VNC 服务器

```bash
vncserver -kill :1
vncserver :1
```

4. 通过在 Vagrantfile 中设置适当的端口转发规则，重启虚拟机并应用新的配置

```bash
vagrant reload
vagrant ssh
```


5. 启动 VNC 服务器
```bash
vncserver :1
```

## 连接到 VNC

1. 启动 VNC 服务器
```
vncserver
```

2. 使用 VNC 客户端（如 TigerVNC、RealVNC 等）连接到虚拟机 

https://www.realvnc.com/en/connect/download/viewer/

```
<虚拟机 IP>:5901
192.168.33.11:5901
```

## 总结
1. 需要手动启动 VNC 服务器：每次启动虚拟机后，您都需要手动运行 vncserver 命令来启动 VNC 服务。
2. 确保配置正确：检查 xstartup 文件和其他配置，以确保能够成功加载桌面环境。
3. 使用 VNC 客户端连接：通过客户端连接到虚拟机的指定端口。
