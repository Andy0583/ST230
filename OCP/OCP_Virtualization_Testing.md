### 1、環境需求
* OCPv測試需搭配CSI StorageClass / VolumeSnapshotClass
* 準備相關ISO檔案，需上傳至OCP PVC中

### 2、建置Bootable Volumes
* OCP console -> "Virtualization" -> "Bootable Volumes" -> "Add Volume" -> "With form"
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-5.png)   
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-6.png)
* 查看進度條，等待ISO上傳完成

### 3、VM建置
* OCP console -> "Virtualization" -> "VirtualMachines" -> "Create VirtualMachines" -> "From InstanceType" 

