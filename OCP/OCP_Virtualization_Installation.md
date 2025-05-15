### 1、環境需求
* Master + Worker：16 core / 32GB Mem / 300GB HD(Thin)
* CPU：Expose hardware assisted virtualization to the guest OS
* OCPv需建立VolumeSnapshotClass

### 2、Virtualization安裝
* 可於OCP安裝過程中一同安裝，亦可於裝完後再行安裝，本LAB採用OCP安裝後再行安裝Virtualization
* 從OCP console -> "Operator" -> "OperatorHub" -> 搜尋『Virtualization』 -> 點選"OpenShift Virtualization"
* 於OpenShift Virtualization中點選 "Install"  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp-1.png)
* 再點選 "Install" ，點選"Create HyperConverged"  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp-2.png)
