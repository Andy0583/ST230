### 下載CSM Wizard
---
> https://dell.github.io/csm-docs/docs/getting-started/installation/installationwizard/src/index.html

![](https://github.com/Andy0583/OCP/blob/main/Image/csm/csm-5.png?raw=true)</p>
* 其餘皆為預設即可，並下載values.yaml檔案


### 安裝CSM
> 安裝及更新 Helm 3.0
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm repo add dell https://dell.github.io/helm-charts
helm repo update            
```
> 建立NameSpace
```
kc ns csi-unity
```

> 開啟瀏覽器輸入 "K8S IP : Port"，若成功會顯示如下圖 </p>


