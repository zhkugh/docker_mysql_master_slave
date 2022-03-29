---------------- Commands for SERVER-1 ------------------
IP-Address e.g. = 172.27.0.2

---

# DROP USER 'canal'@'%';

CREATE USER 'canal'@'%' IDENTIFIED BY 'canal123456';

GRANT REPLICATION SLAVE ON _._ TO 'canal'@'%' IDENTIFIED BY 'canal123456';

FLUSH TABLES WITH READ LOCK;

SHOW MASTER STATUS;

UNLOCK TABLES;

---------------- Commands for SERVER-2 -----------------
IP-Address e.g. = 172.27.0.3

---

STOP SLAVE;

RESET SLAVE;

CHANGE MASTER TO
MASTER_HOST = '172.27.0.2',
MASTER_USER = 'root',
MASTER_PASSWORD = '123456',
MASTER_PORT = 3306,
MASTER_LOG_FILE = 'mysql-bin.000001',
MASTER_LOG_POS = 615,
MASTER_CONNECT_RETRY = 10;

START SLAVE;

SHOW SLAVE STATUS \G;

--------------------- 配置说明 ---------------------
基于 Docker 拉取 Mysql:5.7 作为基础配置镜像

---

1. 在 docker-compose.yml 编排完成 Mysql 主从服务
   1.1 设定服务名称
   1.2 设定容器名称
   1.3 设定端口
   1.4 设定数据库密码
   1.5 映射 Mysql 「mysqld.cnf」配置文件，设定 server-id （同一局域网内注意要唯一）、开启二进制日志功能

2. 执行 docker-compose up --build 编译运行

3. 执行 docker inspect mysql-5.7.master 查看保存 IPAddress，用于 slave 库 配置
   3.1 docker inspect + 「容器地址、容器名称」检阅详细信息

4. 执行 docker exec -it mysql-5.7.master /bin/bash 进入主库容器
   4.1 执行 myql -uroot -p123456
   4.2 创建一个专门用来复制的用户，执行 4.2.1 、4.2.2
   4.2.1 CREATE USER 'canal'@'%' IDENTIFIED BY 'canal123456';
   4.2.2 GRANT REPLICATION SLAVE ON _._ TO 'canal'@'%' IDENTIFIED BY 'canal123456';
   4.3 执行 SHOW MASTER STATUS; 查看主节点状态，获取从库连接到主库的信息

5. 执行 docker exec -it mysql-5.7.slave /bin/bash 进入从库容器
   4.1 执行 myql -uroot -p123456
   4.2 配置连接到主服务器的相关信息 ，执行 4.2.1 、4.2.2、4.2.3、4.2.4
   4.2.1 STOP SLAVE;
   4.2.2
   CHANGE MASTER TO
   MASTER_HOST = '172.27.0.2',
   MASTER_USER = 'root',
   MASTER_PASSWORD = '123456',
   MASTER_PORT = 3306,
   MASTER_LOG_FILE = 'mysql-bin.000001',
   MASTER_LOG_POS = 615,
   MASTER_CONNECT_RETRY = 30;
   4.2.3 START SLAVE;
   4.2.4 show slave status \G; 查看同步状态
