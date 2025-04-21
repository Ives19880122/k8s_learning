# 300-Kubernetes-Overview

- [老師講義網址](https://goo.gl/9BZvMv)
  - 投影片檔,可以開啟

- 授課老師 陳松林 [mail](oc99.98@gmail.com)

## 引言

- 交接痛苦，沒有相關文件資訊。

- 歷史除錯紀錄遺失
  - 現況git commit
  - 實際沒有特別寫文件

## Docker

- Containerization
- 降低Human Error
- App Container
- `Running Any App Anywhere`
- 文件 -> Dockerfile

### Application Container - 軟體貨櫃

- 一台電腦(虛擬電腦)
- `Linux namespace`: 提供軟體不同的`隔離`功能
  - 表示系統是在Linux底下跑
- `CGroup`: 分配硬體資源(CPU,Memory,Network)
  - Control Group
  - K8s 六顆星重點
- `Overlay2`: 由Application Container Image建立Container檔案系統
  - `Linux底下`
  - 特性:啟動電腦，檔案出現，關閉電腦，檔案消失。
    - 可用`Volume`解決問題。
- `Virtual Bridge/Slirp4netns`: 建立虛擬網路，連結Internet
- 程式App需要的相依檔案包含在Application Container內

#### WASM

- WebAssembly
- 與JVM概念一樣
- Container時代有機會成形取代Overlay2
- `課外實驗`:樹梅派單板電腦跑K3S的WASM

### 認識 Open Container Initiative

- https://opencontainers.org/
- OCI 組織
- App Container 標準化

### Application Container 生成運作圖

![圖1](./images/1.png)

- OCI 三個標準
  - `Distribution`
  - `Image`
  - `Runtime`

- image內部會放 設定檔 / 壓縮檔 / extension lib

### Docker Container Runtime

![圖2](./images/2.png)
- 兩個背景程式 跑在 dockerd

#### dockerd
- d: `daemon`
- Follow oci distribution
- 下載image
- 虛擬網路

#### containerd

- Follow oci Runtime
- 執行runc `oci Runtime`
- 做出Container

##### containerd-shim

- 負責拿來監控產生的Container
  - 目前執行狀況，使用資源
  - 回報給containerd
- 一對一的關係
- 如果版本號不同時，shim, containerd
  - 會對不起來，無法正常運作
  - `講師分享`：自己手刻不使用套件，可能遇到的問題