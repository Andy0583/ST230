### 1、下載SCG及解壓縮
```
[root@bastion ~]# chmod 777 SCG-5.28.00.14-K8S.bin

[root@bastion ~]# ./SCG-5.28.00.14-K8S.bin --extract
OpenShift CLI (oc) is installed.
Alias 'kubectl' set for 'oc' command.
Extracting...
openshift is installed and flag is set to: true
cleanuping rke2 files
Extracted files: /root/SCG/images.tar
```

### 2、上傳至Worker及設定
```
[root@bastion ~]# scp ~/SCG/images.tar core@worker-1.ocp.andy.com:~/images.tar

[root@bastion ~]# ssh -i ~/.ssh/id_rsa core@worker-1.ocp.andy.com
Red Hat Enterprise Linux CoreOS 418.94.202504231329-0
  Part of OpenShift 4.18, RHCOS is a Kubernetes-native operating system
  managed by the Machine Config Operator (`clusteroperator/machine-config`).
WARNING: Direct SSH access to machines is not recommended; instead,
make configuration changes via `machineconfig` objects:
  https://docs.openshift.com/container-platform/4.18/architecture/architecture-rhcos.html

[core@worker-1 ~]$ ls
images.tar

[core@worker-1 ~]$ sudo podman load -i images.tar
Getting image source signatures
Copying blob 1d384fe652ac done
======= 略 =======

[core@worker-1 ~]$ exit
logout
Connection to worker-1.ocp.andy.com closed.
```

### 3、修改相關參數
```
[root@bastion ~]# vi ~/SCG/SCG/values.yaml
storageClassName: sc-unity
TimeZone: Asia/Taipei
IpAddress: 172.30.100.100
Type: LoadBalancer

[root@bastion ~]# oc label node/worker-1 kubernetes.io/hostname=crc --overwrite
node/worker-1 labeled
```

### 4、安裝SCG
```
[root@bastion ~]# ./SCG-5.28.00.14-K8S.bin --install
OpenShift CLI (oc) is installed.
Alias 'kubectl' set for 'oc' command.
namespace/scg created
Applying custome scc changes..
securitycontextconstraints.security.openshift.io/custom-scc unchanged
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:custom-scc added: "default"
namespace/scg annotated
namespace/scg annotated
namespace/scg labeled
W0518 23:55:45.829993   51722 warnings.go:70] spec.template.spec.hostAliases[1].ip: duplicate ip "172.30.100.100"
NAME: scg-1
LAST DEPLOYED: Sun May 18 23:55:45 2025
NAMESPACE: scg
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
Please wait while SCG services are starting....SCG services started. Please provision SCG by going to URL https://<none>:5700
```

### 5、網路設定
```
[root@bastion ~]# oc get svc -n scg
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP  PORT(S)      AGE
scg-app   ClusterIP   172.30.103.101   <none>        5700/TCP,5701/TCP,5702/TCP,5703/TCP,5704/TCP,5705/TCP,5500/TCP,5501/TCP,5502/TCP,5503/TCP,5504/TCP,5505/TCP,5506/TCP,5507/TCP,5508/TCP,5509/TCP,5510/TCP,162/TCP,162/UDP,9443/TCP,5400/TCP,5401/TCP,5402/TCP,5403/TCP,5404/TCP,5405/TCP,5406/TCP,5407/TCP,5408/TCP,5409/TCP,5410/TCP,5411/TCP,5412/TCP,5413/TCP,21/TCP,25/TCP   24m
```
```yaml
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: scg
  namespace: scg
spec:
  host: scg.apps.ocp.andy.com
  to:
    kind: Service
    name: scg-app
    weight: 100
  port:
    targetPort: 5700
  tls:
    termination: passthrough
    insecureEdgeTerminationPolicy: Redirect
  wildcardPolicy: None
```
```
[root@bastion ~]# oc create -f route-scg.yaml
route.route.openshift.io/scg created

[root@bastion ~]# oc get route -n scg
NAME   HOST/PORT               PATH   SERVICES   PORT   TERMINATION            WILDCARD
scg    scg.apps.ocp.andy.com          scg-app    5700   passthrough/Redirect   None
```
### 6、驗證
```
[root@bastion ~]# oc get pvc -n scg
NAME                             STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
scg-pvc-scgappvolume-scg-app-0   Bound    csivol-a6639951dd   10Gi       RWO            sc-unity       <unset>                 23m

[root@bastion ~]# oc get pod -n scg
NAME        READY   STATUS    RESTARTS   AGE
scg-app-0   3/3     Running   0          22m
```

*  開啟Web https://scg.apps.ocp.andy.com/ </p>
   ![](https://github.com/Andy0583/OCP/blob/main/Image/scg-1.png?raw=true)   
