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

