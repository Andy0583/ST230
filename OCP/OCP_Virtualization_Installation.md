### 1、環境需求
* Master + Worker：16 core / 32GB Mem / 300GB HD(Thin)
* CPU：Expose hardware assisted virtualization to the guest OS
* OCPv需建立VolumeSnapshotClass

### 2、Virtualization安裝
* 可於OCP安裝過程中一同安裝，亦可於裝完後再行安裝，本LAB採用OCP安裝後再行安裝Virtualization


oc login api.ocp.andy.com:6443 -u kubeadmin -p RXreJ-dS3Gr-G2fwm-Wu3bx
