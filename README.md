# k8s ELK

## k8s master节点创建 elk 集群
```
$ kubectl apply -f ./

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

$ kubectl cluster-info | egrep -i "elastic|kibana"
Elasticsearch is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/elasticsearch-logging:db/proxy
Kibana is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kibana-logging/proxy

浏览器访问即可
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
