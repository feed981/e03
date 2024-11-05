# Nginx相关

## 端口 80 被其他进程占用

```bash
sudo docker compose up -d
Error response from daemon: driver failed programming external connectivity on endpoint nginx
(b28ce2ae41212ec290559195c7e6c7e38bd377ecb5b1994d8f3908477dbc073d):
Error starting userland proxy: listen tcp4 0.0.0.0:80: bind: address already in use

sudo systemctl stop nginx
sudo systemctl disable nginx
sudo docker compose restart nginx
```

## Vagrant 无法找到挂载 VirtualBox 共享文件夹

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

## 网站根目录挂载错误
预期: /vagrant/nginx/html/app-web/index.html 而不是 /vagrant/nginx/html/index.html

修改前
```bash
sudo nano docker-compose.yml
 
  nginx:
    volumes:
      - /vagrant/nginx/conf/nginx.conf:/etc/nginx/nginx.conf    # 主配置文件
      - /vagrant/nginx/html/app-web:/usr/share/nginx/html       # 网站根目录

sudo nano /vagrant/nginx/conf/nginx.conf

        location / {
            root /usr/share/nginx/html;
 
vagrant@ubuntu-bionic:~$ sudo docker exec -it nginx ls /usr/share/nginx/html
app-web     index.html
```

修改后
```bash
sudo nano docker-compose.yml
 
  nginx:
    volumes:
      - /vagrant/nginx/conf/nginx.conf:/etc/nginx/nginx.conf    # 主配置文件
      - /vagrant/nginx/html:/usr/share/nginx/html       # 网站根目录

sudo nano /vagrant/nginx/conf/nginx.conf

        location / {
            root /usr/share/nginx/html/app-web;
```			

```bash
sudo docker compose restart nginx
```

## 虚拟机直接访问到宿主机上的服务

```yml
version: '3.5'

services:
  nacos:
    networks:
      - nacos_net

  nginx:
    networks:
      - nacos_net


networks:
  nacos_net:
    name: nacos_net
```
1. 因为 NGINX 在不同的 Docker 网络（如 nacos_net）中运行，不同网络的 Docker 容器之间可能需要特殊的配置或者 NAT 规则来实现通信。
```bash
# 在 NGINX 容器中使用 curl 测试连接
sudo docker exec -it nginx /bin/sh
/ # curl -X POST http://192.168.33.11:51601/user/api/v1/login/login_auth
curl: (7) Failed to connect to 192.168.33.11 port 51601 after 0 ms: Could not connect to server

/ # curl -X POST http://10.0.2.2:51601/user/api/v1/login/login_auth
{"host":null,"code":503,"errorMessage":"服务器内部错误","data":null}/ # exit
```
- 宿主机没有绑定到 192.168.33.11，所以虚拟机访问宿主机上的服务时无法通过这个 IP 地址访问。
- 10.0.2.2 是 Vagrant 专用，适合简单的宿主机访问
由于 10.0.2.2 是一个默认的 NAT 配置 IP，Vagrant 自动管理其访问权限，所以它通常是最简单、最有效的宿主机访问方式。这样，你可以避免额外的网络配置，并且虚拟机可以直接访问到宿主机上的服务。


```bash
user nginx;
#user www-data;
worker_processes auto;

events {
    worker_connections 768;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;

#    include  /vagrant/nginx/conf/leadnews.conf/*.conf;
upstream  heima-app-gateway{
  server 10.0.2.2:51601;
#server 192.168.33.11:51601;
}

    server {
        listen 80;
        server_name localhost;


        location / {
            root /usr/share/nginx/html/app-web;
            index index.html;
            try_files $uri $uri/ =404;
        }

        location /app/ {
                rewrite ^/app/(.*)$ /$1 break;  # 去掉 /app 前缀
                proxy_pass http://heima-app-gateway;
                proxy_set_header HOST $host;  # 不改变源请求头的值
                proxy_pass_request_body on;  #开启获取请求体
                proxy_pass_request_headers on;  #开启获取请求头
                proxy_set_header X-Real-IP $remote_addr;   # 记录真实发出请求的客户端IP
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  #记录代理信息
        }
    }
}
```
