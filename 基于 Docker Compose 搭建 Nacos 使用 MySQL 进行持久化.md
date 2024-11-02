# 基于 Docker Compose 搭建 Nacos 使用 MySQL 进行持久化
https://github.com/alibaba/nacos/blob/develop/distribution/conf/mysql-schema.sql
```sql
select * from config_info ci;
```

## docker-compose.yml

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

# 定义一个名为 nacos_net 的自定义网络，使得容器之间可以相互通信。
networks:
  nacos_net:
    name: nacos_net

# 定义一个名为 mysql_data 的数据卷，用于持久化存储 MySQL 数据。
volumes:
  mysql_data:
```
## 检查 Docker 网络

确认 nacos_net 网络是否已经正确连接到两个容器 nacos、mysql


```bash
sudo docker network inspect nacos_net
```
"Containers": {} 部分为空，代表目前没有任何容器连接到该网络

```bash
[
    {
        "Name": "nacos_net",
        "Id": "bcc511335d9e81aef98cf86a56d32699f2481eaf01fd7f70d94e69bec60e97a7",
        "Created": "2024-11-01T11:02:20.103677562Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.25.0.0/16",
                    "Gateway": "172.25.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "01c81b404b9547dea3c0cd018e82ac762805d8670f392efe3b9e5511b5a0d96d": {
                "Name": "nacos",
                "EndpointID": "924e49eb5b420780ee098f49eb4bab06be98aa2c445af2315965adb71e616c18",
                "MacAddress": "02:42:ac:19:00:02",
                "IPv4Address": "172.25.0.2/16",
                "IPv6Address": ""
            },
            "f6d4498526097f41784d1d4ae72ef2cdabf7c4b217dd866f311620d47697b349": {
                "Name": "mysql",
                "EndpointID": "bf723669f0b20d3b0813c3915bc35b5b6962f65f38d518b324e3324c737a32cf",
                "MacAddress": "02:42:ac:19:00:03",
                "IPv4Address": "172.25.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "nacos_net",
            "com.docker.compose.project": "vagrant",
            "com.docker.compose.version": "2.18.1"
        }
    }
]
```

如果启动服务后仍然无法创建网络，可以手动创建它
```bash
sudo docker network create nacos_net
```

重启 Docker Compose 服务
```bash 
sudo docker compose down
sudo docker compose up -d
```

## 参考

nacos配置中心持久化相关配置

https://blog.csdn.net/qiangqiang12138/article/details/105505971

