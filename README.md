# 常用的服务容器化命令


[TOC]

## mysql

基础使用：
```shell
docker run --name mysql-demo -d -p3366:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7 
```

基于mysql构建镜像(如更新mysql配置、初始化表)：
```shell
docker build -t mysql:5.7-test ./mysqlDockerfile
```

将初始化sql语句并将存储内容挂载在容器中：
```shell
docker run --name mysql-demo -d -p3366:3306 \
-v $PWD/initdb.sql:/docker-entrypoint-initdb.d/initdb.sql \
-v $PWD/mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7 
```

## mysqld-exporter

对目标mysql设定一个指定的监控用户(该用户权限是在一定范围内的)：
```shell
docker exec -it mysql-demo bash
mysql -u root --password=123456 -e "GRANT REPLICATION CLIENT, PROCESS ON *.* TO 'monitor'@'%' IDENTIFIED BY '123456';"
mysql -u root --password=123456 -e "GRANT SELECT ON performance_schema.* TO 'monitor'@'%';"
mysql -u root --password=123456 -e "flush privileges;"
```

运行mysqld-exporter服务：
```shell
docker run -d --name mysqld-exporter \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="monitor:123456@(192.168.100.58:3366)/" \
  prom/mysqld-exporter
```
