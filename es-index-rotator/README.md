# 通过启动 CronJob pod 通过curl清理es index

## 构建镜像
```
$ cd es-index-rotator

$ docker build -t 192.168.1.240:5000/library/es-index-rotator:0.0.1 .

$ docker push 192.168.1.240:5000/library/es-index-rotator:0.0.1
```

## 启动 CronJob pod
```
$ kubectl apply -f rotator.yaml
```

## 查看 cronjob
```
$ kubectl get cronjob -n kube-system 
NAME               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
es-index-rotator   3 1 */1 * *   False     0        19h             20h
```
## 查看日志清理情况
```
$ kubectl get pod -n kube-system |grep es-index-rotator
es-index-rotator-1557507780-7xb89             0/1     Completed   0          19h

# 查看日志，可以了解日志清理情况
$ kubectl logs -n kube-system es-index-rotator-1557507780-7xb89 es-index-rotator 
```
