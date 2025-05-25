### 下載CSM Wizard
---
> https://dell.github.io/csm-docs/docs/getting-started/installation/installationwizard/src/index.html

![](https://github.com/Andy0583/OCP/blob/main/Image/csm/csm-5.png?raw=true)</p>
* 其餘皆為預設即可，並下載values.yaml檔案

> 修改values.yaml檔案，只保留Unity之後資訊
      


### 安裝CSM
---
> 需先安裝完成CSI，並更新 Helm repo
```
helm repo add dell https://dell.github.io/helm-charts
helm repo update      
helm install unity dell/container-storage-modules -n unity-csi -f values.yaml
```
