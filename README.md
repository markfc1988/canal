# canal
只做了 mysql-to-mysql


# canal目录
canal的配置以及版本，官网下载的1.1.7
canal-adapter官方不维护镜像了，自己打包做的


## server-启动
```
docker run -d \
  --name canal-server \
  --restart always \
  -p 11111:11111 \
  -e CANAL_SERVER_IP=0.0.0.0 \
  -e CANAL_SERVER_PORT=11111 \
  -v ./canal-server:/home/admin/canal-server \
  canal/canal-server:v1.1.7
```

## adapter启动
```
docker build -t canal-adapter:v1.1.7 .

docker run -d \
  --name canal-adapter \
  --restart always \
  -p 8081:8081 \
  -v ./canal-adapter/logs:/opt/canal-adapter/logs \
  -v ./canal-adapter/conf:/opt/canal-adapter/conf \
  canal-adapter:v1.1.7
```

## admin启动
```
docker run -d \
  --name canal-admin \
  --restart always \
  -p 8089:8089 \
  -v ./canal-admin/conf:/home/admin/canal-admin/conf \
  -v ./canal-admin/logs:/home/admin/canal-admin/logs \
  canal/canal-admin:v1.1.7

```

# mysql目录
这个目录下有配置文件，做mysql源端-canal-mysql目的端的数据启动参数

## mysql启动脚本-源端
docker run -d --name mysqlsrc \
  -e MYSQL_ROOT_PASSWORD=admin@123456 \
  -v /Users/mark/data/mysql/srcdata:/var/lib/mysql \
  -v /Users/mark/data/mysql/mysrc.cnf:/etc/mysql/conf.d/my.cnf \
  -p 3306:3306 \
  mysql:8.0

```
### 源数据库 创建canal用户
CREATE USER 'canal'@'%' IDENTIFIED BY 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;

###  mysql 命令查看状态呢
SHOW VARIABLES LIKE 'server_id';
SHOW VARIABLES LIKE 'binlog_format';
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'bind_address'
```

## mysql启动脚本-目的端
```
docker run -d --name mysqldes \
  -e MYSQL_ROOT_PASSWORD=admin@123456 \
  -v /Users/mark/data/mysql/desdata:/var/lib/mysql \
  -v /Users/mark/data/mysql/mydes.cnf:/etc/mysql/conf.d/my.cnf \
  -p 3309:3306 \
  mysql:8.0
  
# 查看状态
SHOW VARIABLES LIKE 'server_id';
SHOW VARIABLES LIKE 'read_only';
SHOW VARIABLES LIKE 'super_read_only';
SHOW VARIABLES LIKE 'binlog_format';
SHOW VARIABLES LIKE 'log_bin';

# 创建用户同步数据用
CREATE USER 'canal_sink'@'%' IDENTIFIED BY 'canal_sink';
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'canal_sink'@'%';
FLUSH PRIVILEGES;

# 创建用户admin服务用(如果有创建忽略)
CREATE USER 'canal_manager'@'%' IDENTIFIED BY 'canal_manager';
GRANT ALL ON canal_manager.* TO 'canal_manager'@'%';
FLUSH PRIVILEGES;
```
## 测试用例
```
# 建库，需要同步的库，源端目标端都需要
CREATE DATABASE IF NOT EXISTS test CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
# 建表
USE test;

CREATE TABLE IF NOT EXISTS user_info (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  age INT,
  email VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# 插入数据
INSERT INTO user_info (name, age, email)
VALUES
('Blice', 28, 'alice@example.com');

```
# canal
