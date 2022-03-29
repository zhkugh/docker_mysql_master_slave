### Commands for SERVER-1 , IP-Address e.g. = 172.27.0.2

```
    DROP USER 'canal'@'%';

    CREATE USER 'canal'@'%' IDENTIFIED BY 'canal123456';

    GRANT REPLICATION SLAVE ON *.* TO 'canal'@'%' IDENTIFIED BY 'canal123456';

    SHOW MASTER STATUS;
```

### Commands for SERVER-2 , IP-Address e.g. = 172.27.0.3

```
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
```

#### 配置说明 , 基于 Docker 拉取 Mysql:5.7 作为基础配置镜像

1. 在 docker-compose.yml 编排完成 Mysql 主从服务

```
   1.1 设定服务名称
   1.2 设定容器名称
   1.3 设定端口
   1.4 设定数据库密码
   1.5 映射 Mysql 「mysqld.cnf」配置文件，设定 server-id （同一局域网内注意要唯一）、开启二进制日志功能
```

docker-compose.yml

```
version: "3.1"

    services:

      mysql-master:
        image: mysql:5.7
        container_name: mysql-5.7.master
        ports:
          - "6606:3306"
        environment:
          - MYSQL_ROOT_PASSWORD=123456
        volumes:
          - ./mysql/mysqld.master.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf

      mysql-slave:
        image: mysql:5.7
        container_name: mysql-5.7.slave
        ports:
          - "7706:3306"
        environment:
          - MYSQL_ROOT_PASSWORD=123456
        volumes:
          - ./mysql/mysqld.slave.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf
```

2. 执行 docker-compose up --build 编译运行

3. 执行 docker inspect mysql-5.7.master 查看保存 IPAddress，用于 slave 库 配置

```
 3.1 docker inspect + 「容器地址、容器名称」检阅详细信息
```

4. 执行 docker exec -it mysql-5.7.master /bin/bash 进入主库容器

```
   4.1 执行 myql -uroot -p123456
   4.2 创建一个专门用来复制的用户，执行 4.2.1 、4.2.2
       4.2.1 CREATE USER 'canal'@'%' IDENTIFIED BY 'canal123456';
       4.2.2 GRANT REPLICATION SLAVE ON *.* TO 'canal'@'%' IDENTIFIED BY 'canal123456';
   4.3 执行 SHOW MASTER STATUS; 查看主节点状态，获取从库连接到主库的信息
```

5. 执行 docker exec -it mysql-5.7.slave /bin/bash 进入从库容器

```
   5.1 执行 myql -uroot -p123456
   5.2 配置连接到主服务器的相关信息 ，执行 4.2.1 、4.2.2、4.2.3、4.2.4
       5.2.1 STOP SLAVE;
       5.2.2
           CHANGE MASTER TO
           MASTER_HOST = '172.27.0.2',
           MASTER_USER = 'root',
           MASTER_PASSWORD = '123456',
           MASTER_PORT = 3306,
           MASTER_LOG_FILE = 'mysql-bin.000002',
           MASTER_LOG_POS = 1638,
           MASTER_CONNECT_RETRY = 30;
       5.2.3 START SLAVE;
       5.2.4 show slave status \G; 查看同步状态 ⬇️
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.27.0.2
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 2866
               Relay_Log_File: 0e6defd75e88-relay-bin.000002
                Relay_Log_Pos: 1548
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2866
              Relay_Log_Space: 1762
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 0f1db376-ae69-11ec-ac07-0242ac190002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

ERROR:
No query specified
```
