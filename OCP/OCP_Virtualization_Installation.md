### 1、環境需求
* Master + Worker：16 core / 32GB Mem / 300GB HD(Thin)
* CPU：Expose hardware assisted virtualization to the guest OS

### 2、Virtualization安裝
* 可於OCP安裝過程中一同安裝，亦可於裝完後再行安裝，本LAB採用OCP安裝後再行安裝Virtualization
* 從OCP console -> "Operator" -> "OperatorHub" -> 搜尋『Virtualization』 -> 點選"OpenShift Virtualization"
* 於OpenShift Virtualization中點選 "Install"  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-1.png?raw=true)
* 再點選 "Install" ，點選"Create HyperConverged" -> "Create"  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-2.png?raw=true)
* 等待出現下圖後，點選"Refresh Web Console"  
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-3.png?raw=true)
* 查看Web Console左側導覽列上，是否新增"Virtualization"
* 第一次建立PVC，需點  https://cdi-uploadproxy-openshift-cnv.apps.ocp.andy.com/v1beta1/upload-form-async  
  (需依據自己網域名稱修改網址)
