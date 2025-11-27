# Browser-use 使用說明

**版本**：v0.9.6  
**文件日期**：2025年11月  
**目標讀者**：負責部署與維護的技術人員

---

## 1. 簡介

### 1.1 專案概述

Browser-use 是一個基於 Python 的 AI 瀏覽器自動化框架，透過 LLM（Large Language Model）驅動的 Agent 自主操作網頁瀏覽器，完成複雜的網頁互動任務。

**專案資訊：**
- **GitHub 專案**：<https://github.com/browser-use/browser-use>
- **參考文件**：<https://deepwiki.com/browser-use/browser-use>
- **官方文件**：<https://docs.browser-use.com>

### 1.2 核心架構

Browser-use 採用事件驅動架構，主要組件包括：

- **Agent**：主要協調器，管理任務執行與 LLM 驅動的動作循環
- **BrowserSession**：管理瀏覽器生命週期、CDP（Chrome DevTools Protocol）連線，並透過事件匯流排協調多個 Watchdog 服務
- **Tools**：動作註冊表，將 LLM 決策映射到瀏覽器操作（點擊、輸入、滾動等）
- **DomService**：提取與處理 DOM 內容，處理元素高亮與無障礙樹生成
- **LLM Integration**：支援多種 LLM 提供商的抽象層

### 1.3 技術棧

- **語言**：Python >= 3.11
- **瀏覽器控制**：CDP（Chrome DevTools Protocol）
- **依賴管理**：`uv`（推薦）或 `pip`
- **容器化**：支援 Docker 部署
- **授權**：MIT License

---

## 2. 環境變數詳解 (Environment Variables)

### 2.1 環境變數總覽

Browser-use 支援透過環境變數或 `.env` 檔案進行配置。以下表格列出所有可用的環境變數：

| 變數名稱 | 預設值 | 必填 | 說明 |
|---------|--------|------|------|
| **LLM API Keys** |
| `OPENAI_API_KEY` | - | 否* | OpenAI API 金鑰（使用 OpenAI 時必填） |
| `ANTHROPIC_API_KEY` | - | 否* | Anthropic API 金鑰（使用 Claude 時必填） |
| `GOOGLE_API_KEY` | - | 否* | Google API 金鑰（使用 Gemini 時必填） |
| `BROWSER_USE_API_KEY` | - | 否* | Browser Use Cloud API 金鑰（使用 ChatBrowserUse 時必填） |
| `DEEPSEEK_API_KEY` | - | 否 | DeepSeek API 金鑰 |
| `GROK_API_KEY` | - | 否 | Grok API 金鑰 |
| `NOVITA_API_KEY` | - | 否 | Novita API 金鑰 |
| **Azure OpenAI 專用** |
| `AZURE_OPENAI_KEY` | - | 否* | Azure OpenAI API 金鑰（使用 Azure OpenAI 時必填） |
| `AZURE_OPENAI_API_KEY` | - | 否* | Azure OpenAI API 金鑰（`AZURE_OPENAI_KEY` 的別名） |
| `AZURE_OPENAI_ENDPOINT` | - | 否* | Azure OpenAI 服務端點 URL（使用 Azure OpenAI 時必填） |
| `AZURE_OPENAI_DEPLOYMENT` | - | 否 | Azure OpenAI Deployment 名稱（若未指定，使用 `model` 參數） |
| **日誌與遙測** |
| `BROWSER_USE_LOGGING_LEVEL` | `info` | 否 | 日誌級別：`debug`, `info`, `warning`, `error`, `critical` |
| `CDP_LOGGING_LEVEL` | `WARNING` | 否 | CDP 日誌級別 |
| `BROWSER_USE_DEBUG_LOG_FILE` | - | 否 | Debug 日誌檔案路徑 |
| `BROWSER_USE_INFO_LOG_FILE` | - | 否 | Info 日誌檔案路徑 |
| `ANONYMIZED_TELEMETRY` | `true` | 否 | 是否啟用匿名遙測（`true`/`false`） |
| **Browser Use Cloud** |
| `BROWSER_USE_CLOUD_SYNC` | `true` | 否 | 是否啟用雲端同步（`true`/`false`） |
| `BROWSER_USE_CLOUD_API_URL` | `https://api.browser-use.com` | 否 | Browser Use Cloud API URL |
| `BROWSER_USE_CLOUD_UI_URL` | - | 否 | Browser Use Cloud UI URL |
| **路徑配置** |
| `XDG_CACHE_HOME` | `~/.cache` | 否 | XDG 快取目錄 |
| `XDG_CONFIG_HOME` | `~/.config` | 否 | XDG 配置目錄 |
| `BROWSER_USE_CONFIG_DIR` | `~/.config/browseruse` | 否 | Browser-use 配置目錄 |
| `BROWSER_USE_CONFIG_PATH` | - | 否 | 配置檔案完整路徑（MCP 專用） |
| **代理設定** |
| `BROWSER_USE_PROXY_URL` | - | 否 | 代理伺服器 URL（格式：`http://host:port`） |
| `BROWSER_USE_NO_PROXY` | - | 否 | 不使用代理的網域列表（逗號分隔） |
| `BROWSER_USE_PROXY_USERNAME` | - | 否 | 代理伺服器使用者名稱 |
| `BROWSER_USE_PROXY_PASSWORD` | - | 否 | 代理伺服器密碼 |
| **瀏覽器設定** |
| `BROWSER_USE_HEADLESS` | - | 否 | 是否以無頭模式執行（`true`/`false`，MCP 專用） |
| `BROWSER_USE_ALLOWED_DOMAINS` | - | 否 | 允許存取的網域列表（逗號分隔，MCP 專用） |
| **LLM 設定** |
| `BROWSER_USE_LLM_MODEL` | - | 否 | 預設 LLM 模型名稱（MCP 專用） |
| `DEFAULT_LLM` | - | 否 | 預設 LLM 提供商 |
| `SKIP_LLM_API_KEY_VERIFICATION` | `false` | 否 | 跳過 LLM API 金鑰驗證（`true`/`false`） |
| **執行環境** |
| `IN_DOCKER` | 自動偵測 | 否 | 是否在 Docker 容器中執行（`true`/`false`） |
| `IS_IN_EVALS` | `false` | 否 | 是否在評估模式中執行（`true`/`false`） |
| `WIN_FONT_DIR` | `C:\Windows\Fonts` | 否 | Windows 字型目錄（僅 Windows） |
| **其他** |
| `BROWSER_USE_CALCULATE_COST` | `false` | 否 | 是否計算 API 成本（`true`/`false`） |
| `BROWSER_USE_VERBOSE_OBSERVABILITY` | `false` | 否 | 是否啟用詳細可觀測性（`true`/`false`） |

\* **必填條件**：僅在使用對應的 LLM 提供商時為必填。

### 2.2 Azure OpenAI 環境變數詳解

使用 Azure OpenAI 時，需要設定以下環境變數：

#### 必填變數

| 變數名稱 | 說明 | 範例值 |
|---------|------|--------|
| `AZURE_OPENAI_KEY` | Azure OpenAI 服務的 API 金鑰 | `abc123def456...` |
| `AZURE_OPENAI_ENDPOINT` | Azure OpenAI 服務端點 URL | `https://your-resource.openai.azure.com` |

#### 選填變數

| 變數名稱 | 說明 | 預設值 | 範例值 |
|---------|------|--------|--------|
| `AZURE_OPENAI_DEPLOYMENT` | Deployment 名稱（若未指定，使用 `model` 參數） | `model` 參數值 | `gpt-4-deployment` |
| `AZURE_OPENAI_API_VERSION` | API 版本（在程式碼中設定，預設為 `2024-12-01-preview`） | `2024-12-01-preview` | `2024-12-01-preview` |

#### Azure OpenAI 設定範例

在 `.env` 檔案中設定：

```bash
# Azure OpenAI 必填設定
AZURE_OPENAI_KEY=your-azure-openai-api-key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com

# Azure OpenAI 選填設定
AZURE_OPENAI_DEPLOYMENT=gpt-4-deployment
```

或在程式碼中直接設定：

```python
from browser_use.llm import ChatAzureOpenAI

llm = ChatAzureOpenAI(
    model='gpt-4.1-mini',
    api_key='your-azure-openai-api-key',
    azure_endpoint='https://your-resource.openai.azure.com',
    azure_deployment='gpt-4-deployment',  # 選填
    api_version='2024-12-01-preview',    # 選填
)
```

---

## 3. 安裝與部署 (Installation & Deployment)

### 3.1 本機安裝

#### 3.1.1 前置需求

- Python >= 3.11, < 4.0
- `uv` 套件管理器（推薦）或 `pip`

#### 3.1.2 使用 uv 安裝（推薦）

```bash
# 1. 安裝 uv（若尚未安裝）
pip install uv

# 2. 建立虛擬環境
uv venv --python 3.11
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 3. 安裝 browser-use
uv pip install browser-use

# 4. 安裝 Chromium 瀏覽器
uvx browser-use install
```

#### 3.1.3 使用 pip 安裝

```bash
# 1. 建立虛擬環境
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 2. 安裝 browser-use
pip install browser-use

# 3. 安裝 Chromium 瀏覽器
uvx browser-use install
```

#### 3.1.4 安裝可選依賴

```bash
# CLI 工具
uv pip install "browser-use[cli]"

# 程式碼執行功能（Code-Use Mode）
uv pip install "browser-use[code]"

# AWS 整合
uv pip install "browser-use[aws]"

# Oracle Cloud Infrastructure 整合
uv pip install "browser-use[oci]"

# 影片錄製功能
uv pip install "browser-use[video]"

# 安裝所有可選依賴
uv pip install "browser-use[all]"
```

### 3.2 Docker 部署

#### 3.2.1 建置 Docker 映像檔

```bash
# 從專案根目錄建置
docker build -t browseruse:latest .

# 或使用快速建置（Dockerfile.fast）
docker build -f Dockerfile.fast -t browseruse:fast .
```

#### 3.2.2 執行 Docker 容器

```bash
# 基本執行
docker run -v "$PWD/data":/data browseruse:latest

# 掛載環境變數檔案
docker run -v "$PWD/data":/data --env-file .env browseruse:latest

# 指定環境變數
docker run -v "$PWD/data":/data \
  -e AZURE_OPENAI_KEY=your-key \
  -e AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com \
  browseruse:latest
```

#### 3.2.3 Docker 容器配置

- **資料目錄**：`/data`（可透過 Volume 掛載）
- **暴露埠口**：`9222`（CDP）、`9242`（其他服務）
- **使用者**：非特權使用者 `browseruse`（UID: 911, GID: 911）

### 3.3 Kubernetes Helm 部署

> **注意**：由於專案包含 `Dockerfile`，具備容器化服務特徵，以下提供 Kubernetes Helm Chart 部署指引。

#### 3.3.1 Helm Chart 結構

建立以下 Helm Chart 結構：

```
browser-use/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    ├── secret.yaml
    └── pvc.yaml
```

#### 3.3.2 Chart.yaml

```yaml
apiVersion: v2
name: browser-use
description: Browser-use AI browser automation framework
version: 0.9.6
appVersion: 0.9.6
```

#### 3.3.3 values.yaml

```yaml
replicaCount: 1

image:
  repository: browseruse
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 9222
  nodePort: 30922

persistence:
  enabled: true
  storageClass: nfs-client
  accessMode: ReadWriteMany
  size: 10Gi
  mountPath: /data

env:
  # Azure OpenAI 設定
  AZURE_OPENAI_KEY: ""
  AZURE_OPENAI_ENDPOINT: ""
  AZURE_OPENAI_DEPLOYMENT: ""
  
  # 其他 LLM API Keys
  OPENAI_API_KEY: ""
  ANTHROPIC_API_KEY: ""
  GOOGLE_API_KEY: ""
  BROWSER_USE_API_KEY: ""
  
  # 日誌設定
  BROWSER_USE_LOGGING_LEVEL: "info"
  ANONYMIZED_TELEMETRY: "true"
  
  # 代理設定
  BROWSER_USE_PROXY_URL: ""
  BROWSER_USE_PROXY_USERNAME: ""
  BROWSER_USE_PROXY_PASSWORD: ""

resources:
  requests:
    memory: "2Gi"
    cpu: "1000m"
  limits:
    memory: "4Gi"
    cpu: "2000m"

nodeSelector: {}
tolerations: []
affinity: {}
```

#### 3.3.4 templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "browser-use.fullname" . }}
  labels:
    {{- include "browser-use.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "browser-use.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "browser-use.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: browser-use
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: cdp
          containerPort: 9222
        - name: service
          containerPort: 9242
        env:
        {{- range $key, $value := .Values.env }}
        - name: {{ $key }}
          value: {{ $value | quote }}
        {{- end }}
        volumeMounts:
        - name: data
          mountPath: {{ .Values.persistence.mountPath }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
      volumes:
      - name: data
        {{- if .Values.persistence.enabled }}
        persistentVolumeClaim:
          claimName: {{ include "browser-use.fullname" . }}-pvc
        {{- else }}
        emptyDir: {}
        {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

#### 3.3.5 templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "browser-use.fullname" . }}
  labels:
    {{- include "browser-use.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: cdp
      protocol: TCP
      name: cdp
      {{- if eq .Values.service.type "NodePort" }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
  selector:
    {{- include "browser-use.selectorLabels" . | nindent 4 }}
```

#### 3.3.6 templates/pvc.yaml

```yaml
{{- if .Values.persistence.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "browser-use.fullname" . }}-pvc
  labels:
    {{- include "browser-use.labels" . | nindent 4 }}
spec:
  accessModes:
    - {{ .Values.persistence.accessMode }}
  resources:
    requests:
      storage: {{ .Values.persistence.size }}
  {{- if .Values.persistence.storageClass }}
  storageClassName: {{ .Values.persistence.storageClass }}
  {{- end }}
{{- end }}
```

#### 3.3.7 部署指令

```bash
# 1. 安裝 Helm Chart
helm install browser-use ./browser-use \
  --set env.AZURE_OPENAI_KEY=your-key \
  --set env.AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com

# 2. 或使用 values.yaml
helm install browser-use ./browser-use -f values.yaml

# 3. 檢查部署狀態
kubectl get pods -l app=browser-use

# 4. 查看日誌
kubectl logs -l app=browser-use
```

#### 3.3.8 NFS Persistent Volume 設定範例

若使用 NFS 作為儲存後端，需要先建立 NFS PersistentVolume：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: browser-use-nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-client
  nfs:
    server: your-nfs-server-ip
    path: /path/to/nfs/share
```

---

## 4. 操作指南 (Operations)

### 4.1 基本操作

#### 4.1.1 啟動服務

**本機執行：**

```bash
# 啟動 CLI 介面
browser-use

# 或使用 Python 腳本
python your_script.py
```

**Docker 執行：**

```bash
docker run -v "$PWD/data":/data --env-file .env browseruse:latest
```

#### 4.1.2 執行基本任務

建立 `example.py`：

```python
from browser_use import Agent, Browser, ChatBrowserUse
import asyncio

async def main():
    browser = Browser()
    llm = ChatBrowserUse()
    
    agent = Agent(
        task="搜尋 Browser-use GitHub 專案並回報星標數",
        llm=llm,
        browser=browser,
    )
    
    history = await agent.run()
    print(history.final_result())

if __name__ == "__main__":
    asyncio.run(main())
```

執行：

```bash
python example.py
```

### 4.2 進階設定 (LLM Integration)

#### 4.2.1 Azure OpenAI 整合設定

**方法 1：使用環境變數（推薦）**

在 `.env` 檔案中設定：

```bash
AZURE_OPENAI_KEY=your-azure-openai-api-key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com
AZURE_OPENAI_DEPLOYMENT=gpt-4-deployment  # 選填
```

在程式碼中使用：

```python
from browser_use import Agent, Browser
from browser_use.llm import ChatAzureOpenAI
import asyncio
import os
from dotenv import load_dotenv

load_dotenv()

async def main():
    browser = Browser()
    
    # 從環境變數讀取設定
    llm = ChatAzureOpenAI(
        model='gpt-4.1-mini',
        api_key=os.getenv('AZURE_OPENAI_KEY'),
        azure_endpoint=os.getenv('AZURE_OPENAI_ENDPOINT'),
        azure_deployment=os.getenv('AZURE_OPENAI_DEPLOYMENT'),  # 選填
        api_version='2024-12-01-preview',  # 選填，預設值
    )
    
    agent = Agent(
        task="您的任務描述",
        llm=llm,
        browser=browser,
    )
    
    history = await agent.run()
    return history

if __name__ == "__main__":
    asyncio.run(main())
```

**方法 2：直接在程式碼中指定**

```python
from browser_use import Agent, Browser
from browser_use.llm import ChatAzureOpenAI
import asyncio

async def main():
    browser = Browser()
    
    llm = ChatAzureOpenAI(
        model='gpt-4.1-mini',
        api_key='your-azure-openai-api-key',
        azure_endpoint='https://your-resource.openai.azure.com',
        azure_deployment='gpt-4-deployment',  # 選填
        api_version='2024-12-01-preview',    # 選填
    )
    
    agent = Agent(
        task="您的任務描述",
        llm=llm,
        browser=browser,
    )
    
    history = await agent.run()
    return history

if __name__ == "__main__":
    asyncio.run(main())
```

#### 4.2.2 Azure OpenAI Deployment Name 與 API Version

**Deployment Name：**

- 若未指定 `azure_deployment`，Browser-use 會使用 `model` 參數作為 Deployment 名稱
- 建議在 Azure Portal 中確認 Deployment 名稱，並明確指定

**API Version：**

- 預設值：`2024-12-01-preview`
- 可在程式碼中透過 `api_version` 參數覆蓋
- 建議使用 Azure 支援的最新 API 版本

**完整範例：**

```python
from browser_use.llm import ChatAzureOpenAI

llm = ChatAzureOpenAI(
    model='gpt-4.1-mini',                    # 模型名稱
    api_key='your-key',                      # API 金鑰
    azure_endpoint='https://xxx.openai.azure.com',  # 端點
    azure_deployment='gpt-4-deployment',     # Deployment 名稱（選填）
    api_version='2024-12-01-preview',        # API 版本（選填）
)
```

### 4.3 故障排除 (Troubleshooting)

#### 4.3.1 Azure 連線錯誤

**錯誤 401（未授權）：**

- **原因**：API 金鑰錯誤或過期
- **解決方案**：
  1. 檢查 `AZURE_OPENAI_KEY` 環境變數是否正確
  2. 確認 API 金鑰在 Azure Portal 中有效
  3. 檢查 API 金鑰是否有足夠權限

**錯誤 404（找不到資源）：**

- **原因**：端點 URL 錯誤或 Deployment 不存在
- **解決方案**：
  1. 確認 `AZURE_OPENAI_ENDPOINT` 格式正確：`https://your-resource.openai.azure.com`
  2. 檢查 Deployment 名稱是否正確（在 Azure Portal 中確認）
  3. 確認 Deployment 狀態為「已部署」

**錯誤 429（請求過多）：**

- **原因**：超過 Azure OpenAI 的速率限制
- **解決方案**：
  1. 降低並發請求數量
  2. 實作重試機制（Browser-use 內建）
  3. 考慮升級 Azure OpenAI 服務層級

#### 4.3.2 NFS 掛載失敗

**症狀**：Pod 無法掛載 PersistentVolume

**解決方案：**

1. **檢查 NFS 伺服器連線：**
   ```bash
   # 從 Pod 內測試
   kubectl exec -it <pod-name> -- ping <nfs-server-ip>
   ```

2. **檢查 PersistentVolume 狀態：**
   ```bash
   kubectl get pv
   kubectl describe pv <pv-name>
   ```

3. **檢查 PersistentVolumeClaim 狀態：**
   ```bash
   kubectl get pvc
   kubectl describe pvc <pvc-name>
   ```

4. **確認 StorageClass 設定：**
   ```bash
   kubectl get storageclass
   kubectl describe storageclass nfs-client
   ```

5. **檢查 NFS 路徑權限：**
   - 確認 NFS 伺服器上的路徑存在且可讀寫
   - 檢查 NFS 匯出設定（`/etc/exports`）

#### 4.3.3 Python 套件相依性問題

**錯誤：ModuleNotFoundError**

**解決方案：**

1. **確認虛擬環境已啟動：**
   ```bash
   source .venv/bin/activate  # Linux/macOS
   .venv\Scripts\activate    # Windows
   ```

2. **重新安裝依賴：**
   ```bash
   uv pip install browser-use
   # 或
   pip install browser-use
   ```

3. **安裝可選依賴：**
   ```bash
   uv pip install "browser-use[all]"
   ```

**錯誤：版本衝突**

**解決方案：**

1. **使用 `uv` 管理依賴（推薦）：**
   ```bash
   uv sync
   ```

2. **或使用 `pip` 升級：**
   ```bash
   pip install --upgrade browser-use
   ```

#### 4.3.4 Chromium 瀏覽器問題

**錯誤：找不到 Chromium**

**解決方案：**

```bash
# 安裝 Chromium
uvx browser-use install

# 或手動指定路徑
export CHROMIUM_PATH=/path/to/chromium
```

**錯誤：Chromium 無法啟動（Docker 環境）**

**解決方案：**

1. **確認 Docker 容器有足夠權限：**
   ```dockerfile
   # Dockerfile 中已設定非特權使用者
   USER browseruse
   ```

2. **檢查 `/dev/shm` 大小（Docker）：**
   ```bash
   docker run --shm-size=2g browseruse:latest
   ```

#### 4.3.5 日誌除錯

**啟用詳細日誌：**

```bash
# 環境變數
export BROWSER_USE_LOGGING_LEVEL=debug

# 或設定日誌檔案
export BROWSER_USE_DEBUG_LOG_FILE=/path/to/debug.log
export BROWSER_USE_INFO_LOG_FILE=/path/to/info.log
```

**查看 Docker 容器日誌：**

```bash
docker logs <container-id>
docker logs -f <container-id>  # 即時追蹤
```

**查看 Kubernetes Pod 日誌：**

```bash
kubectl logs <pod-name>
kubectl logs -f <pod-name>     # 即時追蹤
kubectl logs -l app=browser-use  # 查看所有相關 Pod
```

---

## 5. 範例與截圖 (Examples)

### 5.1 基本使用範例

**範例 1：搜尋任務**

```python
from browser_use import Agent, Browser, ChatBrowserUse
import asyncio

async def main():
    browser = Browser(headless=False)
    llm = ChatBrowserUse()
    
    agent = Agent(
        task="前往 GitHub 搜尋 'browser-use' 專案並回報星標數",
        llm=llm,
        browser=browser,
    )
    
    history = await agent.run(max_steps=20)
    print(f"任務完成：{history.final_result()}")

if __name__ == "__main__":
    asyncio.run(main())
```

> [圖片說明：此處應顯示 Browser-use Agent 執行搜尋任務的終端機輸出，包含步驟記錄與最終結果]

**範例 2：表單填寫**

```python
from browser_use import Agent, Browser, ChatBrowserUse
import asyncio

async def main():
    browser = Browser(headless=False)
    llm = ChatBrowserUse()
    
    agent = Agent(
        task="前往 https://example.com/form 並填寫表單：姓名=張三，電子郵件=zhang@example.com",
        llm=llm,
        browser=browser,
    )
    
    history = await agent.run(max_steps=30)
    print(f"表單填寫完成：{history.is_successful()}")

if __name__ == "__main__":
    asyncio.run(main())
```

> [圖片說明：此處應顯示瀏覽器自動填寫表單的過程截圖]

### 5.2 Azure OpenAI 整合範例

**完整範例：使用 Azure OpenAI 執行任務**

```python
import asyncio
import os
from dotenv import load_dotenv
from browser_use import Agent, Browser
from browser_use.llm import ChatAzureOpenAI

load_dotenv()

async def main():
    # 設定瀏覽器
    browser = Browser(
        headless=False,
        window_size={'width': 1280, 'height': 720},
    )
    
    # 設定 Azure OpenAI LLM
    llm = ChatAzureOpenAI(
        model='gpt-4.1-mini',
        api_key=os.getenv('AZURE_OPENAI_KEY'),
        azure_endpoint=os.getenv('AZURE_OPENAI_ENDPOINT'),
        azure_deployment=os.getenv('AZURE_OPENAI_DEPLOYMENT', 'gpt-4.1-mini'),
        api_version='2024-12-01-preview',
    )
    
    # 建立 Agent
    agent = Agent(
        task="前往 Google 搜尋 'Azure OpenAI' 並提取前 3 個搜尋結果的標題",
        llm=llm,
        browser=browser,
        max_steps=25,
    )
    
    # 執行任務
    history = await agent.run()
    
    # 輸出結果
    print("=" * 50)
    print("任務執行歷史：")
    print(f"總步驟數：{history.number_of_steps()}")
    print(f"是否成功：{history.is_successful()}")
    print(f"最終結果：{history.final_result()}")
    print("=" * 50)
    
    # 關閉瀏覽器
    await browser.stop()

if __name__ == "__main__":
    asyncio.run(main())
```

**執行輸出範例：**

```
==================================================
任務執行歷史：
總步驟數：8
是否成功：True
最終結果：
1. Azure OpenAI Service | Microsoft Azure
2. Azure OpenAI Service documentation
3. Get started with Azure OpenAI Service
==================================================
```

> [圖片說明：此處應顯示 Azure OpenAI 整合範例的執行結果截圖，包含終端機輸出與瀏覽器畫面]

### 5.3 進階功能範例

**自訂工具整合：**

```python
from browser_use import Agent, Browser, Tools, ActionResult, ChatBrowserUse
import asyncio

# 建立自訂工具
tools = Tools()

@tools.action(description='取得當前時間')
def get_current_time() -> ActionResult:
    from datetime import datetime
    current_time = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    return ActionResult(
        extracted_content=f"當前時間：{current_time}",
        success=True,
    )

async def main():
    browser = Browser()
    llm = ChatBrowserUse()
    
    agent = Agent(
        task="取得當前時間並顯示在網頁上",
        llm=llm,
        browser=browser,
        tools=tools,
    )
    
    history = await agent.run()
    return history

if __name__ == "__main__":
    asyncio.run(main())
```

> [圖片說明：此處應顯示自訂工具整合的執行流程圖]

---

## 附錄

### A. 相關資源

- **官方文件**：<https://docs.browser-use.com>
- **GitHub 專案**：<https://github.com/browser-use/browser-use>
- **社群支援**：
  - GitHub Issues：<https://github.com/browser-use/browser-use/issues>
  - Discord：<https://link.browser-use.com/discord>
  - 企業支援：<support@browser-use.com>

### B. 版本資訊

- **當前版本**：v0.9.6
- **Python 需求**：>= 3.11, < 4.0
- **授權**：MIT License

### C. 更新日誌

請參考 GitHub Releases：<https://github.com/browser-use/browser-use/releases>

---

**文件結束**
