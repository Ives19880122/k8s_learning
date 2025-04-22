# 302-Taroko-K8S-Pod

## Kubernetes POD

### 常用Taroko K8s指令

- 啟動
  - kci tk8s
  - Are you sure ? YES
- 關閉
  - kco tk8s


### Node & POD overview

![1](./images/1.png)

- 1個Node預設最多110 POD
  - Taroko設定80 POD
- 工作一定要問Node設定多少個POD
- 通常Pod內不會超過三個

##### 補充-IP分配 

- 去頭去尾
- 前10 server
- 後5 
- 一半 DHCP(128起跳)
- 一半 手動配

### Kubernetes Object 佈署命令

![2](./images/2.png)

- 維運兩個方式都會用到

1. kubectl run (命令式 Imperative) - Manage K8s object (POD, Controller, Service) using CLI

2. kubectl create (聲明式 Declarative) - By defining K8s objects in yaml file

- yml是機敏資料，可以知道k8s佈署的方式


## Single-Container Pods

- 輸入`kls`
```
tk8s:Up (Internal:1.32.2)
---------------------------------------
NAMESPACE            NAME                                         READY   STATUS    RESTARTS      AGE
kube-system          calico-kube-controllers-6b65fb5f89-66t62     1/1     Running   2 (70m ago)   18h
kube-system          canal-9dc5v                                  2/2     Running   4 (70m ago)   18h
kube-system          canal-ql5xj                                  2/2     Running   4 (70m ago)   18h
kube-system          canal-rv6n7                                  2/2     Running   4 (70m ago)   18h
kube-system          coredns-66967f4c59-sc4gn                     1/1     Running   2 (70m ago)   18h
kube-system          coredns-66967f4c59-zkpkl                     1/1     Running   2 (70m ago)   18h
kube-system          etcd-tk8s-control-plane                      1/1     Running   2 (70m ago)   19h
kube-system          kube-apiserver-tk8s-control-plane            1/1     Running   2 (70m ago)   19h
kube-system          kube-controller-manager-tk8s-control-plane   1/1     Running   2 (70m ago)   19h
kube-system          kube-proxy-4g4b7                             1/1     Running   2 (70m ago)   18h
kube-system          kube-proxy-lstnd                             1/1     Running   2 (70m ago)   18h
kube-system          kube-proxy-tpn5w                             1/1     Running   5 (70m ago)   19h
kube-system          kube-scheduler-tk8s-control-plane            1/1     Running   2 (70m ago)   19h
kube-system          metrics-server-8467fcc7b7-d9x4b              1/1     Running   4 (70m ago)   18h
local-path-storage   local-path-provisioner-84944977bd-fpmmz      1/1     Running   3 (70m ago)   18h
taroko               taroko-tkadm-74788d9f7c-qttc4                2/2     Running   4 (70m ago)   18h
```

- 三個kube-proxy

### 建立 echoserver Pod

- 指令 `kubectl run echoserver --image=registry.k8s.io/echoserver:1.10 --port 8080`
```
pod/echoserver created
```

- image標準格式
  - 存放的伺服器: `registry.k8s.io`
  - image名稱: `echoserver`
  - 版本代號: `1.10`