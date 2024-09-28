### 安裝minikube
```
https://drive.google.com/drive/folders/1y0rtlpU3L6H_Fa_VUJL1nuvHR1QRlZ_W?usp=sharing
```

```
C:\Users\andyhsu>minikube start --driver=docker --memory 4096

* minikube v1.33.1 on Microsoft Windows 11 Pro 10.0.22621.2428 Build 22621.2428
* Kubernetes 1.30.0 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.30.0
* Using the docker driver based on existing profile
! You cannot change the memory size for an existing minikube cluster. Please first delete the cluster.
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.44 ...
* minikube 1.34.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.34.0
* To disable this notice, run: 'minikube config set WantUpdateNotification false'

* docker "minikube" container is missing, will recreate.
* Creating docker container (CPUs=2, Memory=4000MB) ...
! Image was not built for the current minikube version. To resolve this you can delete and recreate your minikube cluster using the latest images. Expected minikube version: v1.32.0 -> Actual minikube version: v1.33.1
* Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring bridge CNI (Container Networking Interface) ...
Bad local forwarding specification '0:localhost:8443'
* Verifying Kubernetes components...
  - Using image docker.io/kubernetesui/dashboard:v2.7.0
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
  - Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
* Some dashboard features require the metrics-server addon. To enable all features please run:
        minikube addons enable metrics-server
* Enabled addons: storage-provisioner, default-storageclass, dashboard
* kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```
C:\Users\andyhsu>kubectl get node
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   87s   v1.28.3
```

### 測試minikube
```
C:\Users\andyhsu>kubectl run nginx --image=nginx
pod/nginx created

C:\Users\andyhsu>kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          39s

C:\Users\andyhsu>kubectl port-forward nginx 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

C:\Users\andyhsu>kubectl delete pod nginx
pod "nginx" deleted
```
```
C:\Users\andyhsu>kubectl run windows --image=dockurr/windows
pod/windows created

C:\Users\andyhsu>kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
windows   1/1     Running   0          39s

C:\Users\andyhsu>kubectl port-forward nginx 8080:8006
Forwarding from 127.0.0.1:8080 -> 8006
Forwarding from [::1]:8080 -> 8006

C:\Users\andyhsu>kubectl delete pod windows
pod "windows" deleted
```
```
vi test.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx

C:\Users\andyhsu>kubectl create -f dep.yaml
deployment.apps/nginx-deployment created

C:\Users\andyhsu>kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-68f78587df-5nssg   1/1     Running   0          21s
nginx-deployment-68f78587df-9dh89   1/1     Running   0          21s
nginx-deployment-68f78587df-l4xqt   1/1     Running   0          21s
nginx-deployment-68f78587df-nj494   1/1     Running   0          21s
nginx-deployment-68f78587df-p5dld   1/1     Running   0          22s

C:\Users\andyhsu>kubectl delete -f  dep.yaml
deployment.apps "nginx-deployment" deleted
```
```
c:\> kubectl run ubuntu --rm -it --image ubuntu:22.04 -- /bin/bash
If you don't see a command prompt, try pressing enter.

root@ubuntu:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@ubuntu:/# exit
exit
Session ended, resume using 'kubectl attach ubuntu -c ubuntu -i -t' command when the pod is running
pod "ubuntu" deleted
```

### 開啟關閉minikube
```
C:\> minikube start
* minikube v1.33.1 on Microsoft Windows 10 Pro 10.0.19045.2965 Build 19045.2965
* Using the docker driver based on existing profile
* Starting "minikube" primary control-plane node in "minikube" cluster
* Pulling base image v0.0.44 ...
* Restarting existing docker container for "minikube" ...
* Preparing Kubernetes v1.30.0 on Docker 26.1.1 ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: default-storageclass, storage-provisioner
* kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

C:\> minikube stop
* Stopping node "minikube"  ...
* Powering off "minikube" via SSH ...
* 1 node stopped.
```
