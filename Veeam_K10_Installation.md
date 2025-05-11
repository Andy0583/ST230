### 安裝Veeam K10
```
[root@bastion veeam]# kubectl create ns k10
namespace/k10 created

[root@bastion veeam]# helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
"aliyun" has been added to your repositories

[root@bastion veeam]# helm repo add kasten https://charts.kasten.io/
"kasten" has been added to your repositories

[root@bastion veeam]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kasten" chart repository
...Successfully got an update from the "aliyun" chart repository
Update Complete. ⎈Happy Helming!⎈

[root@bastion veeam]#
cat >> /etc/profile << EOF
export KUBECONFIG=/root/.kube/config
EOF

[root@bastion veeam]# . /etc/profile

[root@bastion veeam]# oc get volumesnapshotclass
NAME              DRIVER                  DELETIONPOLICY   AGE
snapclass-unity   csi-unity.dellemc.com   Delete           19h

[root@bastion veeam]# kubectl annotate volumesnapshotclass snapclass-unity k10.kasten.io/is-snapshot-class=true
volumesnapshotclass.snapshot.storage.k8s.io/snapclass-unity annotated

[root@bastion veeam]# oc get sc
NAME             PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
sc-unity         csi-unity.dellemc.com   Delete          Immediate           true                   19h
sc-unity-iscsi   csi-unity.dellemc.com   Delete          Immediate           true                   19m

[root@bastion veeam]# kubectl annotate storageclass sc-unity k10.kasten.io/volume-snapshot-class=k10-snapclass
storageclass.storage.k8s.io/sc-unity annotated

[root@bastion veeam]# helm install k10 kasten/k10 --namespace k10 --set global.persistence.storageClass=sc-unity --set eula.accept=true --set eula.company="Ginnet" --set eula.email="andyhsu@ginnet.com.tw"
NAME: k10
LAST DEPLOYED: Sun May 11 17:17:24 2025
NAMESPACE: k10
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing Kasten’s K10 Data Management Platform 7.5.10!
Documentation can be found at https://docs.kasten.io/.
How to access the K10 Dashboard:
To establish a connection to it use the following `kubectl` command:
`kubectl --namespace k10 port-forward service/gateway 8080:80`
The Kasten dashboard will be available at: `http://127.0.0.1:8080/k10/#/`

# 大約等待四分鐘

[root@bastion veeam]# oc get pod -n k10
NAME                                     READY   STATUS    RESTARTS   AGE
aggregatedapis-svc-9cf556cb5-cczg4       1/1     Running   0          3m58s
auth-svc-77659b6fd4-svlf9                1/1     Running   0          3m57s
catalog-svc-784cdb5d55-8tx8q             2/2     Running   0          3m58s
console-plugin-9db95d9b8-jj25s           1/1     Running   0          3m58s
console-plugin-9db95d9b8-pnmkj           1/1     Running   0          3m58s
console-plugin-proxy-5dc448bcd8-w264p    1/1     Running   0          3m58s
controllermanager-svc-5c8b778964-sslrb   1/1     Running   0          3m57s
crypto-svc-68f7978748-75zlj              4/4     Running   0          3m57s
dashboardbff-svc-74b6db5944-zr7bb        2/2     Running   0          3m57s
executor-svc-5ccd8575d9-jdg7h            1/1     Running   0          3m57s
executor-svc-5ccd8575d9-rb4q4            1/1     Running   0          3m57s
executor-svc-5ccd8575d9-w8zrp            1/1     Running   0          3m57s
frontend-svc-7d88fc9c88-b5xsq            1/1     Running   0          3m58s
gateway-674b987fd-8f9xn                  1/1     Running   0          3m58s
jobs-svc-74554f56f4-xhgb8                1/1     Running   0          3m58s
kanister-svc-6b7bd65c5c-cm5jv            1/1     Running   0          3m58s
logging-svc-6874db97b6-vbrf7             1/1     Running   0          3m58s
metering-svc-75497fbd6b-68gtd            1/1     Running   0          3m58s
state-svc-8f78997f5-wn4rm                2/2     Running   0          3m58s
```

### 網路設定
```
[root@bastion veeam]# kubectl edit svc gateway -n k10
  ports:
  - name: http
    nodePort: 30000
    port: 80
    protocol: TCP
    targetPort: 8000
  selector:
    service: gateway
  sessionAffinity: None
  type: LoadBalancer
```
> 開啟Web http://172.12.25.45:30000/k10/#/dashboard

### 移除Veeam K10
```
[root@bastion veeam]# helm uninstall k10 -n k10
release "k10" uninstalled
```
