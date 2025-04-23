# 303-Taroko-K8S-Workload-3C

- [303投影片連結](https://docs.google.com/presentation/d/1auDBFNJ6KJeg1aYc8Vf_jRycZuM31MjX/edit?usp=drive_link&ouid=106614399658975385059&rtpof=true&sd=true)

## Kubernetes Workload


### 認識 Kubernetes Workload

![1](./images/1.png)
- k8s企業的application
- application不只是單一pod
- 可以由多個不同的pod做出來
- 企業 application 不是 native pod
  - k8s禁止裸pod

### 認識 Kubernetes Workload Resources

![2](./images/2.png)
- Workload產生，要透過Workload Resources
- `Workload Resource` 是 Control plane的`Controller-Manager`
- `四個重點`
  - Deployment and ReplicaSet
    - `Deployment`: application的進板/退板
    - `ReplicaSet`: auto scaling 橫向擴充 (本尊,分身)
      - 如果變0? => k8s 把 application `關機`
      - 跟刪掉差在，設定參數有保留
    - 戰略目標：常常久久，穩穩當當
    - 工作中用到的情況比較高

  - StatefulSet
    - 用於管理有狀態的應用（Stateful Applications）。
    - 支援與 `Persistent Volume（PV）`整合，確保數據持久化。
    - 適合需要多節點協作的應用（如資料庫、分散式系統）。
  - DaemonSet
    - 確保每個 Node 上都運行一個 Pod 副本。
    - **特性**：
      - 適合需要在所有 Node 上執行的應用（如監控代理、日誌收集器）。
      - 當有新 Node 加入叢集時，DaemonSet 會自動在該 Node 上部署 Pod。
    - **應用場景**：
      - 系統監控（如 Prometheus Node Exporter）。
      - 日誌收集（如 Fluentd、Filebeat）。
      - 網路代理（如 Cilium、Calico）。

  - Job and CronJob
    - **Job**:
      - 用於執行一次性任務，確保任務成功完成。
      - 支援重試機制，確保任務執行成功。
      - **應用場景**：
        - 資料處理任務。
        - 批次任務（如備份、數據遷移）。

    - **CronJob**:
      - 用於執行定時任務，類似於 Linux 的 `cron`。
      - 支援基於時間表的任務執行。
      - **應用場景**：
        - 定期備份。
        - 定時清理任務。
        - 定期報表生成。
      - 程式放在pod裡面跑，絕對不會Retry。

----- 

### K8S Workload 運作架構

![3](./images/3.png)

- `deploy`: deployment
- `rs`: ReplicaSet
  - 確保指定數量的 Pod 副本在叢集中運行
- `hpa`: controller plan 的 sub-process
  - Horizontal Pod Autoscaler
  - 自動調整 Pod 的副本數量，根據資源使用情況進行橫向擴展
- `secret`: 存儲敏感數據, 帳號密碼憑證
- `cm`: ConfigMap, 存儲非敏感的配置資料
- `svc`: service
  - Kubernetes 中的網路抽象層，用於將流量從外部或內部路由到對應的 Pod。
  - 提供穩定的虛擬 IP（ClusterIP）或外部訪問方式（NodePort、LoadBalancer）。
  - service選pod, 表示有label selector
    - pod有對應的label
- `ing`: Ingress
  - 真正對外做連接的入口
  - 負責管理外部流量進入 Kubernetes 叢集。
  - 提供 HTTP/HTTPS 路由功能，將流量導向對應的 Service。
  - `reverse proxy nginx`: 改裝協助支援http以外的協定
  - `重要`: ingress將淘汰變成 `GatewayAPI`
- `PV`: Persistent Volume
  - 叢集中的存儲資源。
  - 來源:
    - K8S Volume
      - emptyDir, hostPath, Local
      - 雲端生態系通常會禁用，有資安疑慮
      - 最好弄清地端,雲端生態系的差異->原生
    - CSI標準提供儲存空間(上雲唯一選擇)
- `PVC`: Persistent Volume Claim
  - Pod 請求存儲資源的方式，用於數據持久化。
  - 寫在Pod裡面，要求PV
- `ns`: Namespace 辦公室
  - Kubernetes 中的邏輯隔離單位，用於組織和管理資源。
  - 每個 Workload 都運行在特定的 Namespace 中。
  - 三個在`namespace`下運作的要點
    - `Netpol`: network policy 網路策略，用於控制 Pod 之間的通信。(資安)
    - `Limits`: 資源限制，確保 Pod 不會超出指定的 CPU 和內存使用量。
    - `Quota`: 資源配額，用於限制 Namespace 中的資源使用。
      - 實務做法，壓數字等電話，抓錯就會接到改善電話。

-----

## Kubernetes Service Discovery

### kube-dns service & coredns deployment Controller

- 指令 `kubectl get all -n kube-system -l  k8s-app=kube-dns`
  - `-n`: namespace
  - `kube-system`: k8s的總管理處
  - `-l`: label
  - `k8s-app=kube-dns`: 標籤
    ```
    NAME                           READY   STATUS    RESTARTS        AGE
    pod/coredns-66967f4c59-sc4gn   1/1     Running   3 (6h44m ago)   46h
    pod/coredns-66967f4c59-zkpkl   1/1     Running   3 (6h44m ago)   46h

    NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
    service/kube-dns   ClusterIP   10.98.0.10   <none>        53/UDP,53/TCP,9153/TCP   46h

    NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/coredns   2/2     2            2           46h

    # 進板退板，會產生新的replicaset
    NAME                                 DESIRED   CURRENT   READY   AGE
    replicaset.apps/coredns-668d6bf9bc   0         0         0       46h
    replicaset.apps/coredns-66967f4c59   2         2         2       46h
    ```

  - 由上圖列出資源可看出
    - deployment coredns server
    - coredns產生兩個replicaset
    - repicaset產生兩個pod
    - 符合workload resources `deployment and replicaset`
  - k8s規劃service的ip尾碼一定是`10`
- 如何使用service/kube-dns？
  - `注意`: 只能在k8s內部其他workload使用，無法對外使用

- 指令 `kubectl get endpoints kube-dns -n kube-system`
  - 取得服務名稱 在 特定namespace底下
    ```
    NAME       ENDPOINTS                                                  AGE
    kube-dns   10.244.0.13:53,10.244.0.14:53,10.244.0.13:53 + 3 more...   47h
    ```
  - 檢查指令回傳，可以看到兩個IP位置，代表兩個pod
  - 注意`label`打錯會造成找不到服務endpoint，出錯比率很高 from 官網分享

-----

### kube-dns Service 名稱解析

- 指令 `kubectl run dnscmd --image=registry.k8s.io/e2e-test-images/agnhost:2.39`
  - google做的測試image, sre用的測試工具, 不只可以測dns
- 指令 `kubectl exec dnscmd -- cat /etc/resolv.conf`
  - 執行pod,傳入指令 印出 dns設定
    ```
    search default.svc.tk8s.k8s svc.tk8s.k8s tk8s.k8s dns.podman
    nameserver 10.98.0.10
    options ndots:5
    ```
  - `nameserver ip`只要是workload,pod都連到這個位置
  - 這個通常是印出原廠k8s規格設定 `這個位置不能改`，修改了會失聯
  - 錯誤作法：改`8.8.8.8`，絕對會出事

##### 補充resolv.conf

- from github copilot

```
search default.svc.tk8s.k8s svc.tk8s.k8s tk8s.k8s dns.podman
nameserver 10.98.0.10
options ndots:5
```

- Linux 系統中用於配置 DNS（域名系統）解析的文件，Kubernetes 中的 Pod 也會使用這個文件來進行域名解析。

1. `search` 字段
   - 作用：
     - 定義域名搜索後綴，用於簡化域名解析。
     - 當 Pod 嘗試解析一個短域名（如 my-service）時，系統會自動將這些後綴依次附加到短域名後進行解析。
   - 解釋：
     - default.svc.tk8s.k8s：
       - 表示當前 Pod 所在的 Namespace（default）中的 Service。
       - 如果 Pod 嘗試解析 my-service，實際會嘗試解析 my-service.default.svc.tk8s.k8s。
     - svc.tk8s.k8s：
       - 表示叢集中所有 Namespace 的 Service。
       - 如果 my-service.default.svc.tk8s.k8s 無法解析，會嘗試 my-service.svc.tk8s.k8s。
     - tk8s.k8s：
       - 表示叢集的根域名。
       - 如果前兩個後綴無法解析，會嘗試 my-service.tk8s.k8s。
     - dns.podman：
       - 額外的自定義域名後綴，可能用於特定的網路配置或測試環境。
2. nameserver 字段
   - 作用：
     - 定義 DNS 伺服器的 IP 地址，Pod 會將域名解析請求發送到這個地址。
   - 解釋：
     - nameserver 10.98.0.10：
       - 這是 Kubernetes 中的 kube-dns 或 CoreDNS Service 的 ClusterIP。
       - 所有 Pod 的域名解析請求都會發送到這個地址，由 kube-dns 或 CoreDNS 負責解析。
3. options ndots:5
   - 作用：
     - 定義域名中必須包含的點（.）數量，超過這個數量的域名會被認為是完全限定域名（FQDN，Fully Qualified Domain Name）。
     - 如果域名的點數少於 ndots 的值，系統會將 search 後綴依次附加進行解析。
   - 解釋：

     - ndots:5：
       - 如果域名中少於 5 個點（如 my-service），系統會將 search 後綴依次附加進行解析。
       - 如果域名中有 5 個或更多的點（如 my-service.default.svc.tk8s.k8s），系統會直接嘗試解析該域名。

###### 解析流程範例

假設 Pod 嘗試解析域名 my-service，解析流程如下：

1. 嘗試解析 my-service.default.svc.tk8s.k8s。
2. 如果失敗，嘗試解析 my-service.svc.tk8s.k8s。
3. 如果仍然失敗，嘗試解析 my-service.tk8s.k8s。
4. 如果所有嘗試都失敗，返回解析錯誤。

###### 注意事項

1. `nameserver` 的 IP 地址不能修改：
   - 這是 Kubernetes 內部的 DNS 伺服器地址，修改為其他地址（如 8.8.8.8）會導致 Pod 無法解析叢集內的服務域名。
   - 如果需要解析外部域名，kube-dns 或 CoreDNS 會自動將請求轉發到外部 DNS 伺服器。

2. `search` 後綴的順序：

   - Kubernetes 自動生成的 search 後綴順序是固定的，通常以 Pod 所在的 Namespace 為優先。
3. `ndots` 的影響：

   - 如果 ndots 設置過低，可能導致域名解析失敗，因為系統會直接嘗試解析短域名而不附加 search 後綴。

###### 總結

- `resolv.conf` 是 Kubernetes 中 Pod 的 DNS 配置文件，用於域名解析。
- `search` 定義了域名解析的後綴順序，`nameserver` 指定了 Kubernetes 的內部 DNS 伺服器，`options ndots:5` 控制了域名是否附加後綴進行解析。
- 正確配置和理解這些參數對於 Kubernetes 叢集內的服務通信至關重要。