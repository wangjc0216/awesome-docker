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

