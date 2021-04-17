# 1. Making yaml for Persistence volume
```
PS D:\k8s\volume> cat .\nginx-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv
spec:
  storageClassName: standard
  volumeMode: Filesystem
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/data"
    type: DirectoryOrCreate

PS D:\k8s\volume> cat .\nginx-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostpath-pvc
spec:
  storageClassName: standard
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
```

# 2. Create Persistence volume
```
PS D:\k8s\volume> kubectl.exe create -f .\nginx-pv.yaml
persistentvolume/hostpath-pv created

PS D:\k8s\volume> kubectl.exe create -f .\nginx-pvc.yaml
persistentvolumeclaim/hostpath-pvc created

PS D:\k8s\volume> kubectl.exe get pv,pvc
NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
persistentvolume/hostpath-pv   1Gi        RWO            Retain           Bound    default/hostpath-pvc   standard                54s

NAME                                 STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/hostpath-pvc   Bound    hostpath-pv   1Gi        RWO            standard       22s
```

# 3. Login into minikube-m02 and create the config file for nginx
```
PS C:\Users\developer> minikube.exe node list
minikube        192.168.99.100
minikube-m02    192.168.99.101

PS C:\Users\developer> ssh docker@192.168.99.101
docker@192.168.99.101's password: tcuser

$ cat /tmp/data/default.conf
upstream proxy.com {
        server 172.17.0.3:5001;
}

server {
        listen 80;
        server_name localhost;
        location / {
                root /usr/share/nginx/html;
                index index.html index.htm;
                proxy_pass https://proxy.com;
        }
}

$ exit
```

# 4. Create the yaml for deployment of mongodb and run it
```
PS D:\k8s\deployment> cat .\mongo_latest.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-test
spec:
  selector:
    matchLabels:
      run: mongo-test
  replicas: 1
  template:
    metadata:
      labels:
        run: mongo-test
    spec:
      containers:
      - name: mongodb
        image: docker.io/mongo
        ports:
        - containerPort: 27017
        
PS D:\k8s\deployment> kubectl.exe create -f .\mongo_latest.yaml
deployment.apps/mongo-test created

PS D:\k8s\deployment> kubectl.exe describe pod mongo-test
Name:         mongo-test-8974576b4-jzg25
Namespace:    default
Priority:     0
Node:         minikube-m02/192.168.99.101
Start Time:   Sat, 17 Apr 2021 20:00:49 +0900
Labels:       pod-template-hash=8974576b4
              run=mongo-test
Annotations:  <none>
Status:       Running
IP:           172.17.0.2
IPs:
  IP:           172.17.0.2
Controlled By:  ReplicaSet/mongo-test-8974576b4
Containers:
  mongodb:
    Container ID:   docker://31279b91b6a271dee5f0646de498978fdee747abf7c4e55ae19eb3a20d93701f
    Image:          docker.io/mongo
    Image ID:       docker-pullable://mongo@sha256:b66f48968d757262e5c29979e6aa3af944d4ef166314146e1b3a788f0d191ac3
    Port:           27017/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 17 Apr 2021 20:00:56 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-cjxcb (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-cjxcb:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-cjxcb
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  42s   default-scheduler  Successfully assigned default/mongo-test-8974576b4-jzg25 to minikube-m02
  Normal  Pulling    41s   kubelet            Pulling image "docker.io/mongo"
  Normal  Pulled     35s   kubelet            Successfully pulled image "docker.io/mongo" in 5.838444182s
  Normal  Created    35s   kubelet            Created container mongodb
  Normal  Started    35s   kubelet            Started container mongodb
  
```

# 5. Create the yaml for deployment of employee and run it
```
PS D:\k8s\deployment> cat .\employee_latest.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: employee-test
spec:
  selector:
    matchLabels:
      run: employee-test
  replicas: 1
  template:
    metadata:
      labels:
        run: employee-test
    spec:
      containers:
      - name: employee
        image: developeronizuka/employee
        ports:
        - containerPort: 5001
        - containerPort: 5000
        env:
        - name: MONGO
          value: "172.17.0.2"
        command: ["/usr/local/dotnet/publish/Employee"]

PS D:\k8s\deployment> kubectl.exe create -f .\employee_latest.yaml
deployment.apps/employee-test created
```


# 6. Create the yaml for deployment of employee and run it
```
PS D:\k8s\deployment> cat .\ngingx_1.14.2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  selector:
    matchLabels:
      run: nginx-test
  replicas: 1
  template:
    metadata:
      labels:
        run: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d
        ports:
        - containerPort: 80
      volumes:
      - name: nginx-conf
        persistentVolumeClaim:
         claimName: hostpath-pvc
```
