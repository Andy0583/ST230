### 安裝Unity CSI
```
[root@bastion ~]# curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11913  100 11913    0     0  16921      0 --:--:-- --:--:-- --:--:-- 16897
.......略........

[root@bastion ~]# yum install git -y
Last metadata expiration check: 1:58:07 ago on Sat 10 May 2025 02:09:13 PM CST.
Dependencies resolved.
.......略........
Complete!

[root@bastion ~]# git clone -b v2.13.0 https://github.com/dell/csi-unity.git
Cloning into 'csi-unity'...
remote: Enumerating objects: 3097, done.
.......略........
Resolving deltas: 100% (1902/1902), done.


[root@bastion ~]# git clone https://github.com/kubernetes-csi/external-snapshotter/
Cloning into 'external-snapshotter'...
remote: Enumerating objects: 66405, done.
.......略........
Resolving deltas: 100% (36353/36353), done.

[root@bastion external-snapshotter]# oc kustomize client/config/crd | oc create -f -
customresourcedefinition.apiextensions.k8s.io/volumegroupsnapshotclasses.groupsnapshot.storage.k8s.io created
customresourcedefinition.apiextensions.k8s.io/volumegroupsnapshotcontents.groupsnapshot.storage.k8s.io created
.......略........
# AlreadyExists錯誤可忽略

[root@bastion external-snapshotter]# oc -n kube-system kustomize deploy/kubernetes/snapshot-controller | oc create -f -
serviceaccount/snapshot-controller created
role.rbac.authorization.k8s.io/snapshot-controller-leaderelection created
.......略........
deployment.apps/snapshot-controller created

[root@bastion external-snapshotter]# oc create namespace unity-csi
namespace/unity-csi created

[root@bastion external-snapshotter]# cd ~/csi-unity/dell-csi-helm-installer/

[root@bastion dell-csi-helm-installer]# git clone https://github.com/dell/helm-charts.git
Cloning into 'helm-charts'...
remote: Enumerating objects: 5871, done.
.......略........
Resolving deltas: 100% (3823/3823), done.

[root@bastion dell-csi-helm-installer]# openssl s_client -showcerts -connect 172.12.25.31:443 </dev/null 2>/dev/null | openssl x509 -outform PEM > ca_cert_0.pem
# -connect：Unity管理IP

[root@bastion dell-csi-helm-installer]# oc create secret generic unity-certs-0 -n unity-csi --from-file=cert-0=ca_cert_0.pem
secret/unity-certs-0 created
# -n：NameSpace

[root@bastion dell-csi-helm-installer]# cat > ~/csi-unity/dell-csi-helm-installer/secret.yaml << EOF
storageArrayList:
  - arrayId: "VIRT2444CBSK5X"
    username: "admin"
    password: "*********"
    endpoint: "https://172.12.25.31/"
    skipCertificateValidation: true
    isDefault: true
EOF
＃ 需修改相對映值

[root@bastion dell-csi-helm-installer]# oc create secret generic unity-creds -n unity-csi --from-file=config=secret.yaml
secret/unity-creds created

[root@bastion dell-csi-helm-installer]# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0    376      0 --:--:-- --:--:-- --:--:--   376
100 57.3M  100 57.3M    0     0   9.8M      0  0:00:05  0:00:05 --:--:-- 12.1M

[root@bastion dell-csi-helm-installer]# install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

[root@bastion dell-csi-helm-installer]# wget -O values.yaml https://github.com/dell/helm-charts/raw/csi-unity-2.13.0/charts/csi-unity/values.yaml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.109.133, 185.199.111.133, ...
.......略........
2025-05-10 16:19:05 (6.28 MB/s) - ‘values.yaml’ saved [10875/10875]

[root@bastion dell-csi-helm-installer]# ./csi-install.sh --namespace unity-csi --values ./values.yaml  --skip-verify-node
------------------------------------------------------
> Verifying Kubernetes and driver configuration
------------------------------------------------------
|- Kubernetes Version: 1.31
|
|- Driver: csi-unity
|
|- Verifying OpenShift version
  |
  |--> Verifying minimum OpenShift version                          Success
  |
  |--> Verifying maximum OpenShift version                          Success
|
|- Verifying that required namespaces have been created             Success
|
|- Verifying that required secrets have been created                Success
|
|- Verifying that optional secrets have been created                Success
|
|- Verifying helm version                                           Success
|
|- Verifying helm values version                                    Success

------------------------------------------------------
> Verification Complete - Success
------------------------------------------------------
|
|- Installing Driver                                                Success
  |
  |--> Waiting for Deployment unity-controller to be ready          Success
  |
  |--> Waiting for DaemonSet unity-node to be ready                 Success
------------------------------------------------------
> Operation complete
------------------------------------------------------

[root@bastion dell-csi-helm-installer]# oc get pods -n unity-csi
NAME                                READY   STATUS    RESTARTS        AGE
unity-controller-54d9f46784-ddhgz   5/5     Running   0               5m14s
unity-controller-54d9f46784-ttrlm   5/5     Running   0               5m14s
unity-node-k5qvh                    2/2     Running   0               5m14s
unity-node-lgcf6                    2/2     Running   0               5m14s
unity-node-zvzfv                    2/2     Running   0               5m14s
```

### 移除Unity CSI
```
[root@bastion dell-csi-helm-installer]# ./csi-uninstall.sh --namespace unity-csi
release "unity" uninstalled
Removal of the CSI Driver is in progress.
It may take a few minutes for all pods to terminate.
```


### 若需使用iSCSI需於每台Worker設定
```
[root@bastion ~]# ssh -i ~/.ssh/id_rsa core@172.12.25.47
The authenticity of host '172.12.25.47 (172.12.25.47)' can't be established.
.......略........

[core@worker-3 ~]$ sudo iscsiadm -m discovery -t st -p 172.12.25.33
172.12.25.33:3260,1 iqn.1992-04.com.emc:cx.virt2444cbsk5x.a1

[core@worker-3 ~]$ sudo iscsiadm --mode node --portal 172.12.25.33:3260,1 iqn.1992-04.com.emc:cx.virt2444cbsk5x.a1 --login
Logging in to [iface: default, target: iqn.1992-04.com.emc:cx.virt2444cbsk5x.a1, portal: 172.12.25.33,3260]
Login to [iface: default, target: iqn.1992-04.com.emc:cx.virt2444cbsk5x.a1, portal: 172.12.25.33,3260] successful.

[core@worker-3 ~]$ sudo systemctl restart iscsi

[core@worker-3 ~]$ exit
logout
Connection to 172.12.25.47 closed.
```
