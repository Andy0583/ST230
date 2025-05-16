### 1、安裝MinIO
```
[root@bastion ~]# oc create ns minio
namespace/minio created
```
```
[root@bastion ~]# cat > ~/minio.yaml << EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: minio-pvc
spec:
  storageClassName: sc-unity
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
  volumeMode: Filesystem
---
kind: Secret
apiVersion: v1
metadata:
  name: minio-secret
stringData:
  # change the username and password to your own values.
  # ensure that the user is at least 3 characters long and the password at least 8
  minio_root_user: minio
  minio_root_password: minio123
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: minio
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
        - name: data
          persistentVolumeClaim:
            claimName: minio-pvc
      containers:
        - resources:
            limits:
              cpu: 250m
              memory: 1Gi
            requests:
              cpu: 20m
              memory: 100Mi
          readinessProbe:
            tcpSocket:
              port: 9000
            initialDelaySeconds: 5
            timeoutSeconds: 1
            periodSeconds: 5
            successThreshold: 1
            failureThreshold: 3
          terminationMessagePath: /dev/termination-log
          name: minio
          livenessProbe:
            tcpSocket:
              port: 9000
            initialDelaySeconds: 30
            timeoutSeconds: 1
            periodSeconds: 5
            successThreshold: 1
            failureThreshold: 3
          env:
            - name: MINIO_ROOT_USER
              valueFrom:
                secretKeyRef:
                  name: minio-secret
                  key: minio_root_user
            - name: MINIO_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: minio-secret
                  key: minio_root_password
          ports:
            - containerPort: 9000
              protocol: TCP
            - containerPort: 9090
              protocol: TCP
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: data
              mountPath: /data
              subPath: minio
          terminationMessagePolicy: File
          image: >-
            quay.io/minio/minio:latest
          args:
            - server
            - /data
            - --console-address
            - :9090
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: Recreate
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
---
kind: Service
apiVersion: v1
metadata:
  name: minio-service
spec:
  ipFamilies:
    - IPv4
  ports:
    - name: api
      protocol: TCP
      port: 9000
      targetPort: 9000
    - name: ui
      protocol: TCP
      port: 9090
      targetPort: 9090
  internalTrafficPolicy: Cluster
  type: ClusterIP
  ipFamilyPolicy: SingleStack
  sessionAffinity: None
  selector:
    app: minio
---
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: minio-api
spec:
  to:
    kind: Service
    name: minio-service
    weight: 100
  port:
    targetPort: api
  wildcardPolicy: None
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
---
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: minio-ui
spec:
  to:
    kind: Service
    name: minio-service
    weight: 100
  port:
    targetPort: ui
  wildcardPolicy: None
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
EOF
```
```
[root@bastion ~]# oc create -f minio.yaml -n minio
persistentvolumeclaim/minio-pvc created
secret/minio-secret created
deployment.apps/minio created
service/minio-service created
route.route.openshift.io/minio-api created
route.route.openshift.io/minio-ui created
```
```
[root@bastion ~]# oc get pod -n minio
NAME                     READY   STATUS    RESTARTS   AGE
minio-76449f8dfb-mlmv5   1/1     Running   0          53s

[root@bastion ~]# oc get route minio-api -n minio
NAME        HOST/PORT                             PATH   SERVICES        PORT   TERMINATION     WILDCARD
minio-api   minio-api-default.apps.ocp.andy.com          minio-service   api    edge/Redirect   None

[root@bastion ~]# oc get route minio-ui -n minio
NAME       HOST/PORT                            PATH   SERVICES        PORT   TERMINATION     WILDCARD
minio-ui   minio-ui-default.apps.ocp.andy.com          minio-service   ui     edge/Redirect   None
```
### 2、MinIO設定
* 開啟 https://minio-ui-minio.apps.ocp.andy.com/login  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-1.png?raw=true)    
* 輸入Username: minio / Password: minio123，點選"Login"
* Create Bucket (名稱不接受大寫)  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-2.png?raw=true)
* Create Access Key（Key自動產生無須修改）  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/mino/minio-3.png?raw=true)
