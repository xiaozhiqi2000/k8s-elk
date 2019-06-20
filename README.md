# k8s ELK

## cephfs节点获取admin key
```
ceph auth get-key client.admin | base64
QVFCSVNzMWJ6a05lR3hBQVFaS21Jc2tZZE94T0JxM2Q4eGY0UXc9PQ==
```

## k8s master节点创建 admin secret
```
$ kubectl apply -f ceph-secret.yaml

$ kubectl get secret -n kube-system
```

## k8s master节点创建 ceph storageclass
```
$ kubectl apply -f rbd-storage-class.yaml

$ kubectl get storageclass -n kube-system
```

## k8s master节点创建 elk 集群
```
$ kubectl apply -f elk/

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

```

## 查看绑定的PV, PVC
```
$ kubectl get pv,pvc -n kube-system
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                       STORAGECLASS   REASON   AGE
persistentvolume/pvc-001c7c40-9278-11e9-89e7-86581d859264   50Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-4   elk-rbd                 5m24s
persistentvolume/pvc-c5e61fc3-9271-11e9-89e7-86581d859264   80Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-0   elk-rbd                 49m
persistentvolume/pvc-dd9bb1fc-9277-11e9-89e7-86581d859264   50Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-2   elk-rbd                 6m24s
persistentvolume/pvc-e1db5c87-9271-11e9-89e7-86581d859264   80Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-1   elk-rbd                 49m
persistentvolume/pvc-f144616e-9277-11e9-89e7-86581d859264   50Gi       RWO            Delete           Bound    kube-system/elasticsearch-logging-elasticsearch-logging-3   elk-rbd                 5m51s

NAME                                                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-0   Bound    pvc-c5e61fc3-9271-11e9-89e7-86581d859264   80Gi       RWO            elk-rbd        50m
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-1   Bound    pvc-e1db5c87-9271-11e9-89e7-86581d859264   80Gi       RWO            elk-rbd        49m
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-2   Bound    pvc-dd9bb1fc-9277-11e9-89e7-86581d859264   50Gi       RWO            elk-rbd        6m25s
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-3   Bound    pvc-f144616e-9277-11e9-89e7-86581d859264   50Gi       RWO            elk-rbd        5m52s
persistentvolumeclaim/elasticsearch-logging-elasticsearch-logging-4   Bound    pvc-001c7c40-9278-11e9-89e7-86581d859264   50Gi       RWO            elk-rbd        5m27s

```

## 查看容器内挂载的rbd
```
$ kubectl exec -it elasticsearch-logging-0 -n kube-system -- df -Th
Filesystem              Type     Size  Used Avail Use% Mounted on
overlay                 overlay   42G  6.8G   36G  16% /
tmpfs                   tmpfs     64M     0   64M   0% /dev
tmpfs                   tmpfs    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs       42G  6.8G   36G  16% /etc/hosts
shm                     tmpfs     64M     0   64M   0% /dev/shm
/dev/rbd0               xfs       80G   77M   80G   1% /usr/share/elasticsearch/data
tmpfs                   tmpfs    3.9G   12K  3.9G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                   tmpfs    3.9G     0  3.9G   0% /proc/acpi
tmpfs                   tmpfs    3.9G     0  3.9G   0% /proc/scsi
tmpfs                   tmpfs    3.9G     0  3.9G   0% /sys/firmware

```
