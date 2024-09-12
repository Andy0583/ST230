### 檔案下載
```
https://drive.google.com/drive/folders/1y0rtlpU3L6H_Fa_VUJL1nuvHR1QRlZ_W?usp=sharing
```

### 執行minikube
```
minikube.exe start --driver=docker --memory 4096
```

### 測試minikube
```
C:\> kubectl run nginx --image=nginx
pod/nginx created

C:\> kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2m4s

C:\> kubectl port-forward nginx 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

C:\> kubectl delete pod nginx
pod "nginx" deleted
```
```
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

C:\Users\admin\Downloads>kubectl create -f dep.yaml
deployment.apps/nginx-deployment created

C:\Users\admin\Downloads>kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-68f78587df-5nssg   1/1     Running   0          21s
nginx-deployment-68f78587df-9dh89   1/1     Running   0          21s
nginx-deployment-68f78587df-l4xqt   1/1     Running   0          21s
nginx-deployment-68f78587df-nj494   1/1     Running   0          21s
nginx-deployment-68f78587df-p5dld   1/1     Running   0          22s

C:\Users\admin\Downloads>kubectl delete -f  dep.yaml
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
