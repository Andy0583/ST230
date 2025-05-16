### 1、建立NameSpace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: minio-ocp
  labels:
    name: minio-ocp
```
```
[[root@bastion ~]# oc create -f ns.yaml
namespace/minio-ocp created

[root@bastion ~]# oc get ns
NAME                                               STATUS   AGE
minio-ocp                                          Active   3h31m
```

### 2、建立PVC
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: minio-ocp
  namespace: minio-ocp
spec:
  storageClassName: sc-unity
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
  volumeMode: Filesystem
```
```
[root@bastion ~]# oc create -f pvc.yaml
persistentvolumeclaim/minio-ocp created

[root@bastion ~]# oc get pvc -n minio-ocp
NAME        STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
minio-ocp   Bound    csivol-f66d75e2d6   20Gi       RWX            sc-unity       <unset>                 2m22s
```

### 3、建立Minio
```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: minio
  namespace: minio-ocp
  annotations:
    deployment.kubernetes.io/revision: '1'
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: minio
    spec:
      volumes:
        - name: minio-ocp
          persistentVolumeClaim:
            claimName: minio-ocp
      containers:
        - resources: {}
          readinessProbe:
            tcpSocket:
              port: 9090
            timeoutSeconds: 5
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          terminationMessagePath: /dev/termination-log
          name: container
          command:
            - /bin/bash
            - '-c'
          ports:
            - containerPort: 9090
              protocol: TCP
            - containerPort: 9000
              protocol: TCP
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: minio-ocp
              mountPath: /data
          terminationMessagePolicy: File
          image: 'quay.io/minio/minio:latest'
          args:
            - 'minio server /data --console-address :9090'
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      nodeSelector:
        node-role.kubernetes.io/worker: ''
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```
```
[root@bastion ~]# oc create -f minio-ocp.yaml
deployment.apps/minio created

[root@bastion ~]# oc get pod -n minio-ocp
NAME                     READY   STATUS    RESTARTS   AGE
minio-76f9577994-bz5tw   1/1     Running   0          39s

[root@bastion ~]#  oc describe pod/minio-76f9577994-bz5tw -n minio-ocp | grep server
      minio server /data --console-address :9090
```

### 4、建立SVC
```yaml
kind: Service
apiVersion: v1
metadata:
  name: minio-webui
  namespace: minio-ocp
  labels:
    app: minio
spec:
  ports:
    - protocol: TCP
      port: 9090
      targetPort: 9090
      name: webui
  type: ClusterIP
  selector:
    app: minio
---
kind: Service
apiVersion: v1
metadata:
  name: minio-api
  namespace: minio-ocp
  labels:
    app: minio
spec:
  ports:
    - protocol: TCP
      port: 9000
      targetPort: 9000
      name: api
  type: ClusterIP
  selector:
    app: minio
```
```
[root@bastion ~]# oc create -f minio-svc.yaml
service/minio-webui created
service/minio-api created

[root@bastion ~]# oc get svc -n minio-ocp
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
minio-api     ClusterIP   172.30.30.90   <none>        9000/TCP   18s
minio-webui   ClusterIP   172.30.50.56   <none>        9090/TCP   18s
```

### 5、建立Route
```yaml
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: webui
  namespace: minio-ocp
  labels:
    app: minio
spec:
  host: webui-minio-ocp.apps.ocp.andy.com
  to:
    kind: Service
    name: minio-webui
    weight: 100
  port:
    targetPort: webui
  wildcardPolicy: None
---
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: s3
  namespace: minio-ocp
  labels:
    app: minio
spec:
  host: s3-minio-ocp.apps.ocp.andy.com
  to:
    kind: Service
    name: minio-api
    weight: 100
  port:
    targetPort: api
  wildcardPolicy: None
```
```
[root@bastion ~]# oc create -f minio-route.yaml
route.route.openshift.io/webui created
route.route.openshift.io/s3 created

[root@bastion ~]# oc get routes -n minio-ocp
NAME    HOST/PORT                           PATH   SERVICES      PORT    TERMINATION   WILDCARD
s3      s3-minio-ocp.apps.ocp.andy.com             minio-api     api                   None
webui   webui-minio-ocp.apps.ocp.andy.com          minio-webui   webui                 None
```

### 6、MinIO設定
* 開啟 http://webui-minio-ocp.apps.ocp.andy.com/  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-1.png?raw=true)    
* 輸入Username: minioadmin / Password: minioadmin，點選"Login"
* Create Bucket (名稱不接受大寫)  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-2.png?raw=true)
* Create Access Key（Key自動產生無須修改，必需紀錄）  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-3.png?raw=true)

### 7、MinIO測試
* 設定S3 Browser連接  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-4.png?raw=true)
* 查看是否有剛建立Bucket，並上傳檔案至Bucket內  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-ㄓ.png?raw=true)
