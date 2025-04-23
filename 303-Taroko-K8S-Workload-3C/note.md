# 303-Taroko-K8S-Workload-3C

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
- `ns`: Namespace
  - Kubernetes 中的邏輯隔離單位，用於組織和管理資源。
  - 每個 Workload 都運行在特定的 Namespace 中。
  - network policy要在namespace下運作
- `Netpol`: 網路策略，用於控制 Pod 之間的通信。
- `Limits`: 資源限制，確保 Pod 不會超出指定的 CPU 和內存使用量。
- `Quota`: 資源配額，用於限制 Namespace 中的資源使用。