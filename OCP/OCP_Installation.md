### 將RHEL ISO上傳至bastion中
---
```
mkdir /var/repo
mount -o loop rhel-baseos-9.1-x86_64-dvd.iso /var/repo/
```

### 設定Repository
---
```
cat > /etc/yum.repos.d/rhel9-local.repo << EOF
[Local-BaseOS]
name=Red Hat Enterprise Linux 9 - BaseOS
metadata_expire=-1
gpgcheck=1
enabled=1
baseurl=file:///var/repo//BaseOS/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
[Local-AppStream]
name=Red Hat Enterprise Linux 9 - AppStream
metadata_expire=-1
gpgcheck=1
enabled=1
baseurl=file:///var/repo//AppStream/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
EOF
```
```
yum clean all
subscription-manager clean
sed -i 's/enabled=1/enabled=0/' /etc/yum/pluginconf.d/subscription-manager.conf
```

### 關閉防火牆
---
```
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i 's/enforcing/disabled/' /etc/selinux/config
```

### 安裝Bind
---
```
yum install bind bind-utils -y
```
```
vi /etc/named.conf
options {
        listen-on port 53 { any; };
#       listen-on-v6 port 53 { ::1; };
        allow-query     { any; };
        forwarders { 8.8.8.8; };
        dnssec-enable no;
        dnssec-validation no;
zone "ocp.andy.com" IN {
        type master;
        file "named.ocp.andy.com";
};

zone "25.12.172.in-addr.arpa" IN {
        type master;
        file "rev.25.12.172";
};
```
```
cat > /var/named/named.ocp.andy.com << EOF
$TTL 1D
@       IN SOA  @ bastion.ocp.andy.com. (
                                        2020040819      ; serial
                                        3H      ; refresh
                                        15M     ; retry
                                        1W      ; expire
                                        1D )    ; minimum
@       IN NS   bastion.ocp.andy.com.
@       IN A    172.12.25.41
bastion         IN      A       172.12.25.41
master-1        IN      A       172.12.25.42
master-2        IN      A       172.12.25.43
master-3        IN      A       172.12.25.44
worker-1        IN      A       172.12.25.45
worker-2        IN      A       172.12.25.46
worker-3        IN      A       172.12.25.47
api             IN      A       172.12.25.48
*.apps          IN      A       172.12.25.49
EOF
```
```
cat > /var/named/rev.25.12.172 << EOF
$TTL 1D
@       IN SOA  @ bastion.ocp.andy.com. (
                                        2020040819      ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@       IN NS   bastion.ocp.andy.com.
25.12.172.in-addr.arpa      IN      PTR     bastion.ocp.andy.com
41   IN      PTR     bastion.ocp.andy.com.
42   IN      PTR     master-1.ocp.andy.com.
43   IN      PTR     master-2.ocp.andy.com.
44   IN      PTR     master-3.ocp.andy.com.
45   IN      PTR     worker-1.ocp.andy.com.
46   IN      PTR     worker-2.ocp.andy.com.
47   IN      PTR     worker-3.ocp.andy.com.
48   IN      PTR     api.ocp.andy.com.
49   IN      PTR     apps.ocp.andy.com.
EOF
```
```
chgrp named /var/named/named.ocp.andy.com
chmod 640 /var/named/named.ocp.andy.com
chgrp named /var/named/rev.25.12.172
chmod 640 /var/named/rev.25.12.172
systemctl enable named
systemctl start named
systemctl status named
```

### 產生SSH
---
```
ssh-keygen -t rsa -b 4096 -N '' -f ~/.ssh/id_rsa
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
cat /root/.ssh/id_rsa.pub
```

### 建置Master * 3 + Worker * 3
---
> Master：4 core / 8GB Mem / 100GB HD(Thin) ; Worker：4 core / 16GB Mem / 100GB HD(Thin)</p>
> 需紀錄網卡 Mac address及設定每台VM EnableUUID（如下）
```
VM -> Edit Settiongs -> Advanced -> Edit Configuration
disk.EnableUUID / TRUE
```

### 設定Redhat Console-1
---
> https://console.redhat.com/
```
1.Create Clasrer
2.Select "Datacenter" and "Bare Metal (x86_64)"
3.Select "Interactive"
4.Enter Cluster name / Base domain(網域名稱) / OpenShift version(選擇版本) / Host network configuration：Static IP
5.Enter DNS / Machine network(網段非IP) / Default gateway
6.Enter VM Mac address
7.Not selected Operators
8.Select "Add Host" - "Full Image" / Paste SSH public key / Generate Discovery ISO - Download Discovery ISO
9.Start VM using ISO
```

### 設定Redhat Console-2
---
```
1.Config VM Role
2.Enter "API IP" and "Ingress IP"
3.Install Cluster
```
> 需等待 1~2 小時，完成後需紀錄密碼

### 驗證
---
> 設定DNS為bastion</p>
> 連至https://console-openshift-console.apps.ocp.andy.com/</p>
  (ocp：Cluster name / andy.com：Base domain)

### OC Tool installation
---
> GUI右上角"Command Line Tools"，下載"Download oc for Linux for x86_64"，上傳至bastion
```
tar xvf oc.tar
cp oc /usr/bin/
```

### 登陸OCP
---
```
oc login api.ocp.andy.com:6443 -u kubeadmin -p jvCYn-Eo3Cn-hFCDN-4xthg
oc get node
```
