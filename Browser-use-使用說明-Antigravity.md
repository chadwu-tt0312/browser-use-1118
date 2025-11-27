# Browser-use 使用說明

**版本**：v1.0
**日期**：2025-11-27
**目標讀者**：負責部署與維護的 DevOps 工程師及後端開發人員。

---

## 1. 簡介

Browser-use 是一個開源的 AI 瀏覽器自動化框架，允許 AI Agent 透過 Chrome DevTools Protocol (CDP) 直接操控瀏覽器以完成複雜任務。本專案旨在為開發與維運團隊提供標準化的部署與操作指引。

* **專案網頁**：[GitHub](https://github.com/browser-use/browser-use)
* **技術文件**：[DeepWiki](https://deepwiki.com/browser-use/browser-use)

## 2. 環境變數詳解 (Environment Variables)

請參閱 `.env.example` 進行配置。以下為關鍵變數說明：

| 變數名稱 | 預設值 | 必填 | 說明 |
| :--- | :--- | :--- | :--- |
| `BROWSER_USE_LOGGING_LEVEL` | `info` | 否 | 系統日誌等級 (debug, info, warning, error) |
| `BROWSER_USE_API_KEY` | - | 否 | Browser Use Cloud API 金鑰 (若使用雲端版) |
| `ANONYMIZED_TELEMETRY` | `true` | 否 | 是否啟用匿名遙測 |
| `BROWSER_USE_HEADLESS` | `false` | 否 | 是否以無頭模式 (Headless) 執行瀏覽器 |
| **LLM 整合 (Azure OpenAI)** | | | |
| `AZURE_OPENAI_API_KEY` | - | **是** | Azure OpenAI 資源的金鑰 |
| `AZURE_OPENAI_ENDPOINT` | - | **是** | Azure OpenAI 資源的端點 URL |
| `OPENAI_API_VERSION` | - | **是** | API 版本 (例如 `2024-02-15-preview`) |
| `AZURE_DEPLOYMENT_NAME` | - | **是** | 模型部署名稱 (Deployment Name) |

> [!IMPORTANT]
> 若使用 Azure OpenAI，請務必同時設定 `AZURE_OPENAI_API_KEY` 與 `AZURE_OPENAI_ENDPOINT`，並確認網路環境可連通 Azure 服務。

## 3. 安裝與部署 (Installation & Deployment)

本專案支援「本機 Python 開發」與「Kubernetes 容器化部署」兩種模式。

### 3.1 本機安裝 (Python)

專案包含 `pyproject.toml`，建議使用 `uv` 或 `pip` 進行安裝。

```bash
# 方法 A: 使用 uv (推薦)
uv sync --all-extras

# 方法 B: 使用 pip
pip install .
```

### 3.2 Kubernetes Helm 部署

由於專案具備 `Dockerfile` (Expose 9242, 9222) 與資料卷 `/data`，以下提供標準 Helm Chart 部署範本。

#### `values.yaml` 範例
```yaml
image:
  repository: browseruse/browseruse
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: NodePort
  ports:
    - name: http
      port: 9242
      targetPort: 9242
      nodePort: 30242
    - name: debug
      port: 9222
      targetPort: 9222
      nodePort: 30222

persistence:
  enabled: true
  size: 10Gi
  storageClass: nfs-client  # 請依據叢集環境調整
```

#### `deployment.yaml` 範例
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: browser-use
spec:
  replicas: 1
  selector:
    matchLabels:
      app: browser-use
  template:
    metadata:
      labels:
        app: browser-use
    spec:
      containers:
        - name: browser-use
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 9242
            - containerPort: 9222
          envFrom:
            - configMapRef:
                name: browser-use-config
            - secretRef:
                name: browser-use-secrets
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: browser-use-pvc
```

## 4. 操作指南 (Operations)

### 4.1 基本操作

啟動服務 (Docker 範例)：
```bash
docker run -p 9242:9242 -v $(pwd)/data:/data --env-file .env browseruse/browseruse
```

### 4.2 進階設定：Azure OpenAI 整合

若需將預設 LLM 替換為 Azure OpenAI，請在 `.env` 或 Kubernetes Secret 中設定：

```bash
# .env
AZURE_OPENAI_API_KEY="sk-..."
AZURE_OPENAI_ENDPOINT="https://<resource>.openai.azure.com/"
OPENAI_API_VERSION="2024-02-15-preview"
# 務必指定部署名稱，否則 SDK 可能無法找到模型
AZURE_DEPLOYMENT_NAME="gpt-4-turbo"
```

> [!TIP]
> 在程式碼中初始化 `LangChain` 或 `BrowserUse` 物件時，系統會自動讀取上述環境變數。無需手動傳遞參數。

### 4.3 故障排除 (Troubleshooting)

| 問題現象 | 可能原因 | 排查建議 |
| :--- | :--- | :--- |
| **Azure 401 Unauthorized** | 金鑰錯誤或過期 | 檢查 `AZURE_OPENAI_API_KEY` 是否正確，並確認該 Key 是否屬於對應的 Endpoint。 |
| **Azure 404 Not Found** | 部署名稱錯誤 | 確認 `AZURE_DEPLOYMENT_NAME` 與 Azure Portal 中的 Model Deployment Name 完全一致。 |
| **Pod Pending (PVC)** | NFS 掛載失敗 | 檢查 StorageClass 設定，或確認 NFS Server IP 白名單是否包含 K8s Node。 |
| **ModuleNotFoundError** | Python 相依遺失 | 確認是否已執行 `uv sync` 或 `pip install`，且虛擬環境 (venv) 已啟用。 |

## 5. 範例與截圖 (Examples)

### 5.1 執行結果範例

當 Agent 成功執行任務時，Console 應顯示如下日誌：

```text
INFO [browser_use.agent] 🤖 Agent started task: "Go to google.com and search for 'browser-use'"
INFO [browser_use.browser] 🌐 Navigating to https://google.com
INFO [browser_use.agent] 👁️  Observed DOM elements: [Input#APjFqb, Button#Gb_70]
INFO [browser_use.agent] 🖱️  Action: Type "browser-use" into Input#APjFqb
INFO [browser_use.agent] ✅ Task completed successfully.
```

> [圖片說明：此處應顯示 Browser-use CLI 執行時的終端機截圖，包含彩色日誌輸出]

### 5.2 Azure 入口網站設定

> [圖片說明：此處應顯示 Azure OpenAI Studio 的 "Deployments" 頁面，標示出 Deployment Name 與 Model Name 的對應關係]
