# 1. Making yaml for Persistence volume
```
PS D:\k8s\deployment> kubectl.exe get sc
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  46h

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



# 3. Create the yaml for deployment of mongodb and run it
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

# 4. Create the yaml for deployment of employee and run it
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

PS D:\k8s\deployment> kubectl.exe describe pod employee-test
Name:         employee-test-85f866c989-g7f2x
Namespace:    default
Priority:     0
Node:         minikube-m02/192.168.99.101
Start Time:   Sat, 17 Apr 2021 20:03:33 +0900
Labels:       pod-template-hash=85f866c989
              run=employee-test
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:           172.17.0.3
Controlled By:  ReplicaSet/employee-test-85f866c989
Containers:
  employee:
    Container ID:  docker://bbb984372627f3733e75daf3519d138b034148b63f9420a2daf4c285f6876d87
    Image:         developeronizuka/employee
    Image ID:      docker-pullable://developeronizuka/employee@sha256:ad36f06fcb5aa8d4da7dc36ac9bf42223617c3330e8dfcaef1b5a30ba9f71084
    Ports:         5001/TCP, 5000/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /usr/local/dotnet/publish/Employee
    State:          Running
      Started:      Sat, 17 Apr 2021 20:03:43 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO:  172.17.0.2
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
  Normal  Scheduled  19m   default-scheduler  Successfully assigned default/employee-test-85f866c989-g7f2x to minikube-m02
  Normal  Pulling    19m   kubelet            Pulling image "developeronizuka/employee"
  Normal  Pulled     19m   kubelet            Successfully pulled image "developeronizuka/employee" in 8.492547907s
  Normal  Created    19m   kubelet            Created container employee
  Normal  Started    19m   kubelet            Started container employee
```

# 5. Check if the web func works well through curl
```
PS D:\k8s\deployment> kubectl.exe exec -it pod/mongo-test-8974576b4-jzg25 /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

root@mongo-test-8974576b4-jzg25:/# echo "nameserver 8.8.8.8" | tee /etc/resolv.conf > /dev/null
root@mongo-test-8974576b4-jzg25:/# apt-get update -qq && apt install curl
root@mongo-test-8974576b4-jzg25:/# curl https://172.17.0.3:5001 -k
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title> - Employee</title>
    <link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" href="/">Employee</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="/Home/Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">

<h1>List of Employees</h1>

<h2></h2>

<a href="/Home/Insert"> Add New Employee</a>

<br /><br />


<table border="1" cellpadding="10">
</table>

<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>

        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2020 - Employee - <a href="/Home/Privacy">Privacy</a>
        </div>
    </footer>
    <script src="/lib/jquery/dist/jquery.min.js"></script>
    <script src="/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/site.js"></script>

</body>
</html>
```

# 6. Login into minikube-m02 and create the config file for nginx
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

# 7. Create the yaml for deployment of nginx and run it
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

PS D:\k8s\deployment> kubectl.exe create -f .\ngingx_1.14.2.yaml
deployment.apps/nginx-test created

PS D:\k8s\deployment> kubectl.exe expose deployment nginx-test --type=LoadBalancer
service/nginx-test exposed

PS D:\k8s\deployment> kubectl.exe get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/employee-test-85f866c989-g7f2x   1/1     Running   0          23m
pod/mongo-test-8974576b4-jzg25       1/1     Running   0          26m
pod/nginx-test-695c6675f5-pfm78      1/1     Running   0          77s

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP        46h
service/nginx-test   LoadBalancer   10.107.127.225   <pending>     80:30165/TCP   53s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/employee-test   1/1     1            1           23m
deployment.apps/mongo-test      1/1     1            1           26m
deployment.apps/nginx-test      1/1     1            1           77s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/employee-test-85f866c989   1         1         1       23m
replicaset.apps/mongo-test-8974576b4       1         1         1       26m
replicaset.apps/nginx-test-695c6675f5      1         1         1       77s
```

# 8. Create the yaml for deployment of nginx and run it
```
Acess the minikube-m02's IP address and the port nubmer which is written in the output of "kubectl get all".
In my case, http://192.168.99.101:30165 is target address and port.
```
