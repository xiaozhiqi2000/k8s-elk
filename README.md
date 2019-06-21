# k8s ELK

## cephfs节点获取admin key 创建 ceph admin用户的 secret
```
$ ceph auth get-key client.admin | base64
$ QVFCSVNzMWJ6a05lR3hBQVFaS21Jc2tZZE94T0JxM2Q4eGY0UXc9PQ==
```

## k8s master节点创建 admin secret
```
$ kubectl apply -f ./cephfs/ceph-secret.yaml

$ kubectl get secret -n kube-system
```

## k8s master节点创建 ceph storageclass
```
$ kubectl apply -f ./cephfs/rbd-storage-class.yaml

$ kubectl get storageclass -n kube-system
```

## k8s master节点创建 elk 集群
```
$ kubectl apply -f es-statefulset-service.yaml -f kibana-deployment-service.yaml -f logstash-deployment-service.yaml -f ingress-kibana.yaml

$ kubectl get svc,pod  -n kube-system -o wide | egrep "kibana|elasticsearch|logstash"

service/elasticsearch-logging   ClusterIP   10.254.32.167   <none>        9200/TCP,9300/TCP        12m   k8s-app=elasticsearch-logging
service/kibana-logging          ClusterIP   10.254.29.233   <none>        5601/TCP                 10m   k8s-app=kibana-logging
service/logstash-application    ClusterIP   10.254.39.144   <none>        5044/TCP                 10m   app=logstash-application
pod/elasticsearch-logging-0                 1/1     Running   0          12m   10.254.75.3    docker-k8s-06   <none>           <none>
pod/elasticsearch-logging-1                 1/1     Running   0          12m   10.254.93.2    docker-k8s-05   <none>           <none>
pod/elasticsearch-logging-2                 1/1     Running   0          12m   10.254.101.3   docker-k8s-04   <none>           <none>
pod/elasticsearch-logging-3                 1/1     Running   0          11m   10.254.125.2   docker-k8s-02   <none>           <none>
pod/elasticsearch-logging-4                 1/1     Running   0          11m   10.254.92.6    docker-k8s-03   <none>           <none>
pod/kibana-logging-764ffc75fb-k7xpb         1/1     Running   0          10m   10.254.125.4   docker-k8s-02   <none>           <none>
pod/logstash-application-64fc9b7c68-lghdg   1/1     Running   0          10m   10.254.92.7    docker-k8s-03   <none>           <none>

$ kubectl get pv,pvc -n kube-system
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                       STORAGECLASS   REASON   AGE
persistentvolume/pvc-7659d0c3-9328-11e9-8061-beb7acb77999   50Gi       RWO            Retain           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-0   elk-rbd                 140m
persistentvolume/pvc-815f05ce-9328-11e9-8061-beb7acb77999   50Gi       RWO            Retain           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-1   elk-rbd                 140m
persistentvolume/pvc-8dd3a9d6-9328-11e9-8061-beb7acb77999   50Gi       RWO            Retain           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-2   elk-rbd                 139m
persistentvolume/pvc-aa42fe53-9328-11e9-8061-beb7acb77999   50Gi       RWO            Retain           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-3   elk-rbd                 139m
persistentvolume/pvc-b8b2439a-9328-11e9-8061-beb7acb77999   50Gi       RWO            Retain           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-4   elk-rbd                 138m

NAME                                                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-0   Bound    pvc-7659d0c3-9328-11e9-8061-beb7acb77999   50Gi       RWO            elk-rbd        140m
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-1   Bound    pvc-815f05ce-9328-11e9-8061-beb7acb77999   50Gi       RWO            elk-rbd        140m
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-2   Bound    pvc-8dd3a9d6-9328-11e9-8061-beb7acb77999   50Gi       RWO            elk-rbd        140m
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-3   Bound    pvc-aa42fe53-9328-11e9-8061-beb7acb77999   50Gi       RWO            elk-rbd        139m
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-4   Bound    pvc-b8b2439a-9328-11e9-8061-beb7acb77999   50Gi       RWO            elk-rbd        138m



查看ingress-kibana.yaml 绑定你的Hosts即可访问了浏览器访问即可
```

## filebeat搜集日志通过k8s节点的5044端口发给logstash需要ingress-nginx tcp代理
```
1. ingress-nginx 通过hostnetwork暴露端口
2. hostnetwork 暴露了80,443,5044
3. ingress-nginx 4层tcp穿透绑定lostash service 通过物理机5044穿透


 21 kind: ConfigMap
 22 apiVersion: v1
 23 metadata:
 24   name: tcp-services
 25   namespace: ingress-nginx
 26   labels:
 27     app.kubernetes.io/name: ingress-nginx
 28     app.kubernetes.io/part-of: ingress-nginx
 29 data:
 30   5044: "kube-system/logstash-application:5044"  # 这里
...
210     spec:
211       serviceAccountName: nginx-ingress-serviceaccount
212       hostNetwork: true     # 这里
213       nodeSelector:
214         ingress: proxy
....
247           ports:
248             - name: http
249               containerPort: 80
250             - name: https
251               containerPort: 443   
252             - name: logstash
253               containerPort: 5044  # 这里
```

## elasticsearch如果用永久存储，编辑es-statefulset.yaml修改成自己的永久存储即可，我这里用的ceph rbd storageclass
```
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-logging
      annotations:
        volume.beta.kubernetes.io/storage-class: elk-rbd
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: elk-rbd
      resources:
        requests:
          storage: 50Gi

```

## 清理elasticsearch 索引
```
$ kubectl apply -f es-index-rotator/rotator.yaml
```

## 参考
https://github.com/ITSvitCo/aws-k8s/tree/master/kubernetes-manifests/elasticsearch

https://github.com/cocowool/k8s-go/tree/master/elk

https://www.jianshu.com/p/af2eb5a75da8

https://blog.csdn.net/qq_33547169/article/details/86629261

https://www.qikqiak.com/post/install-efk-stack-on-k8s/
