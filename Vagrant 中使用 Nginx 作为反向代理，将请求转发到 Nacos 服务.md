
# Vagrant 中使用 Nginx 作为反向代理，将请求转发到 Nacos 服务


docker-compose.yml
```yml
version: '3.5'

services:
  nacos:
    image: nacos/nacos-server:v2.2.2
    container_name: nacos
    restart: always
    ports:
      - "8848:8848"  # Nacos Web 界面访问端口
    environment:
      - MODE=standalone  # 单机模式
      - SPRING_DATASOURCE_PLATFORM=mysql  # 这告诉 Nacos 使用 MySQL 数据库
      - MYSQL_SERVICE_HOST=mysql          # 这里要确保服务名称与 mysql 容器的名称一致
      - MYSQL_SERVICE_PORT=3306           # 端口应保持 3306
      - MYSQL_SERVICE_DB_NAME=nacos_config
      - MYSQL_SERVICE_USER=nacos_user
      - MYSQL_SERVICE_PASSWORD=nacos_password
    volumes:
      - ./logs:/home/nacos/logs  # 持久化日志
    networks:
      - nacos_net

  mysql:
    image: mysql:5.7
    container_name: mysql
    ports:
      - "3308:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: nacos_config
      MYSQL_USER: nacos_user
      MYSQL_PASSWORD: nacos_password
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - nacos_net

  nginx:
    image: nginx:1-alpine
    container_name: nginx
    ports:
      - "80:80"
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /vagrant/nginx/conf/nginx.conf:/etc/nginx/nginx.conf    # 主配置文件
      - /vagrant/nginx/html:/usr/share/nginx/html       # 网站根目录
      - /vagrant/nginx/log:/var/log/nginx                       # 日志文件目录
    networks:
      - nacos_net

networks:
  nacos_net:
    name: nacos_net
volumes:
  mysql_data:
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
cat /vagrant/nginx/conf/nginx.conf

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
