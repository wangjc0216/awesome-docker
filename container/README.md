# 常用的服务容器化命令

## content of table

* [mysql](#mysql)
* [mysqld-exporter](#mysqld-exporter)
* [redis](#redis)
* [redis-exporter](#redis-exporter)
* [nginx](#nginx)
* [busybox](#busybox)
* [ftpd_server](#ftpd_server)



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
[项目链接](https://github.com/prometheus/mysqld_exporter)

## redis
基础使用：
```shell
docker run -itd --name redis-test -p 6379:6379 redis:6.2.3
```

## redis-exporter
oliver006/redis_exporter:v1.23.1

```shell
docker run -d --name redis-exporter -p 9121:9121 oliver006/redis_exporter:v1.23.1 -redis.addr  10.0.0.199:6379
```
[项目链接](https://github.com/oliver006/redis_exporter)

## nginx
```shell
docker run --name nginx-demo -d  -p 8080:80  -v $PWD/nginx.conf:/etc/nginx/nginx.conf  nginx:1.14.2 
```
默认的nginx.conf基础上增加server部分，如下面nginx.conf：
```shell
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;
    
    # 自己加的部分，80端口默认跳转到/tmp/index.html页面
    server {
	listen 80;
        location / {
    	  root /tmp;
	  index index.html;
        }
   }    
    # 在该目录下会有一个default.conf,其中报错location / 中的index.html,也就是默认的NGINX页面
    include /etc/nginx/conf.d/*.conf;
}
```
其实不需要这样去处理，可以通过挂载index.html文件来进行修改。如：
```shell
docker run --name nginx-demo -d  -p 8080:80  -v $PWD/index.html:/usr/share/nginx/html/index.html  nginx:1.14.2
```
其中index.html如下：
```shell
<html>

<head>
<meta charset="UTF-8">
<title>监控系统入口</title>
</head>

<body>

<p>监控系统</p>


<a href="http://192.168.9.105:30601">日志监控平台</a>

<p/>

<a href="http://192.168.9.105:30000">指标监控平台</a>
</body>

</html>
```

##  busybox

```shell
docker run --name busybox-demo -d  busybox sleep 3600
```

## ftpd_server
```shell
docker run -d --name ftpd_server \ 
-p 1023:21 \
-p 30010-30019:30010-30019 \ 
-e "FTP_PASSIVE_PORTS=30010:30019" \  
-v  /tmp/:/home/miles \
-e FTP_USER_HOME=/home/miles \ 
-e FTP_USER_NAME=miles \
-e FTP_USER_PASS=123456 \
-e "PUBLICHOST=192.168.9.105" \ 
stilliard/pure-ftpd
```
[参考链接](https://mileslin.github.io/2020/02/%E4%BD%BF%E7%94%A8-Docker-%E5%BB%BA%E7%BD%AE-FTP-SFTP-%E7%92%B0%E5%A2%83/)
