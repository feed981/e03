# 使用 Docker 和 Nginx 的 docker-compose.yml 文件时遇到挂载网站根目录的错误
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
