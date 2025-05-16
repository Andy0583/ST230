### 1、環境需求
* OCPv測試需搭配CSI StorageClass / VolumeSnapshotClass
* 準備相關ISO檔案，需上傳至OCP PVC中

### 2、建置Bootable Volumes
* OCP console -> "Virtualization" -> "Bootable Volumes" -> "Add Volume" -> "With form"
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-5.png)   
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-6.png)  
* 查看進度條，等待ISO上傳完成

### 3、VM建置（可行步驟，非標準步驟）
* OCP console -> "Virtualization" -> "VirtualMachines" -> "Create VirtualMachines" -> "From InstanceType"
  1.Select volume to boot from：選擇ISO
  2.Select InstanceType：選擇VM規格
  3.VirtualMachine details：填寫VM相關資訊
  4.點選"Customize VirtualMachine" -> "Storage" -> 除roodisk外其他皆移除
  5.點選"Add disk" -> "Volume" ()
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-7.png)  
  6.點選"Create VirtualMachine"
* 查看OCP console -> "Virtualization" -> "VirtualMachines"，即可看到相關資訊
  ![](https://github.com/Andy0583/OCP/blob/main/Image/ocp/ocp-8.png)
* 可點選"Open web console"開啟Console進行操作
