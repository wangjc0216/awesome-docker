# 常用服务的Kubernetes编排

## Mysql
- [x] [mysql deployment](./deploy-mysql.yaml)
- [ ] [mysql statefulset](./sts-mysql.yaml): 还需要搞PVC、PV，可以参照官方文档
```shell
参考Service内容：
https://kubernetes.io/zh/docs/concepts/services-networking/service/#headless-services
参考有状态应用：
https://kubernetes.io/zh/docs/tasks/run-application/run-single-instance-stateful-application/
```

## Mysqld-Exporter
- [x] [mysqld-exporter deployment](./deploy-mysqld-exporter.yaml) 

## busybox
```shell
kubectl run busybox --rm -ti  --image=busybox /bin/sh   --image-pull-policy=IfNotPresent
/ # ping ... (来ping一些内容)
```

## redis
- [x] [redis deployment](./deploy-redis.yaml)

## redis-exporter
- [x] [redis-exporter deployment(sidecar)](./deploy-redis-exporter.yaml): 与redis写在一起
- [x] [redis-exporter deployment](./deploy-redisexporter.yaml)：与redis分开写

[参考链接](https://github.com/oliver006/redis_exporter/blob/master/contrib/k8s-redis-and-exporter-deployment.yaml)

