# 构建项目管理工具：Vagrant 中 Docker Compose 下的 MySQL 和 Redmine 配置指南
## 初始化脚本无效，手动生成

```shell
# 连接到 Docker 中运行的 MySQL 容器
sudo docker exec -it mysql mysql -u root -p
Enter password: 这个要看docker-compose.yml 当初设置的root password

# 创建DATABASE
mysql> CREATE DATABASE IF NOT EXISTS redmine_db;
Query OK, 1 row affected (0.01 sec)

# 创建新的用户
mysql> CREATE USER 'redmine_user'@'%' IDENTIFIED BY 'redmine_password';
Query OK, 0 rows affected (0.02 sec)

# 授予权限
mysql> GRANT ALL PRIVILEGES ON redmine_db.* TO 'redmine_user'@'%';
Query OK, 0 rows affected (0.02 sec)

# 查看当前有哪些DATABASE
mysql> show DATABASES;

# 重新加载权限
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```

## 无法创建新专案
先到 /网站管理/載入預設組態
## 無法載入預設組態
無法載入預設組態： Mysql2::Error: Incorrect string value: '\xE7\xAE\xA1\xE7\x90\x86...' for column 'name' at row 1
無法載入預設組態： Mysql2::Error: Incorrect string value: '\xE4\xBD\x8E' for column 'name' at row 1

这个错误通常与 MySQL 数据库的字符集或排序规则不兼容有关，特别是在存储包含非 ASCII 字符的数据时。建议确保 MySQL 和 Redmine 数据库的字符集设置为支持 Unicode 的 utf8mb4

```sql
USE redmine_db;

-- 确认数据库中的表和列的字符集
SELECT table_name, table_collation
FROM information_schema.tables
WHERE table_schema = 'redmine_db';

SELECT column_name, character_set_name, collation_name
FROM information_schema.columns
WHERE table_schema = 'redmine_db' AND table_name = '<your_table_name>';

-- 确保 redmine_db 数据库级别的字符集设置为 utf8mb4 和排序规则为 utf8mb4_unicode_ci
ALTER DATABASE redmine_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 查询数据库中的所有表并生成 ALTER 语句
-- 这会生成一系列 ALTER TABLE 语句，将所有表设置为 utf8mb4。执行这些语句后，再次尝试加载默认配置。
SELECT CONCAT
FROM information_schema.tables
WHERE table_schema = 'redmine_db';
```
