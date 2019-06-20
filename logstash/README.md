## 自定义自己的 Logstash 镜像

```
$ cd logstash

$ docker build -t 192.168.1.240:5000/library/logstash:5.6.4-v1 .

$ docker push 192.168.1.240:5000/library/logstash:5.6.4-v1
```

## 自定义 Logstash 镜像使用当前 logstash yaml文件
```
$ kubectl apply -f logstash-deployment-service.yaml
```
