# PolyVis Ops MVP Learning Plan

## 目標

PolyVis Ops 的 MVP 目標不是重新訓練 diffusion model，而是把已完成、可交付的 diffusion model 包成一個能展示工程化能力的小系統：

```text
React + Vite 表單
  -> FastAPI API
  -> diffusion model inference
  -> PostgreSQL 紀錄
  -> local image volume
  -> Docker Compose
  -> GitHub Actions
  -> Azure deployment
```

最後要能在面試時展示：

- 使用者輸入射出成型製程參數。
- 後端呼叫 diffusion model 生成光彈圖。
- 每次生成的參數、模型版本、任務狀態、推論時間、圖片路徑都寫入 PostgreSQL。
- 本地可用 Docker Compose 一行啟動。
- GitHub Actions 能跑測試與 build。
- Azure 上有可公開展示的 demo URL。

## MVP 範圍

### 必做

- React + Vite 生成表單
- FastAPI `/api/health`
- FastAPI `/api/generate`
- FastAPI `/api/jobs`
- PostgreSQL 紀錄生成任務
- Docker Compose 本地環境
- GitHub Actions CI
- Azure Container Apps + Azure Database for PostgreSQL 部署

### 不做

- 不重訓模型
- 不做模型優化
- 不做 dashboard
- 不做 auth
- 不做 Celery / Redis queue
- 不做 Blob Storage / S3
- 不做 Kubernetes
- 不做 Terraform
- 不做大型平台化設計

圖片儲存先使用 backend local volume。Blob Storage / S3 只放到後續加分版。

## MVP Timeline

| 階段 | 重點 | 教材 | KPI |
|---|---|---|---|
| 1 | FastAPI 包模型 | [FastAPI Tutorial](https://fastapi.tiangolo.com/tutorial/) | `POST /api/generate` 可呼叫現有 diffusion model；`GET /api/health` 正常 |
| 2 | PostgreSQL 紀錄 | [FastAPI SQL DB](https://fastapi.tiangolo.com/tutorial/sql-databases/), [PostgreSQL SELECT](https://www.postgresql.org/docs/current/static/sql-select.html) | 每次生成寫入 `generation_jobs`、`model_versions`、`generated_images` |
| 3 | React + Vite 表單 | [React Learn](https://react.dev/learn), [Vite Guide](https://vite.dev/guide/) | 前端可輸入製程參數、送出、顯示生成圖與最近 10 筆紀錄 |
| 4 | Docker Compose | [Docker Compose Guide](https://docs.docker.com/guides/docker-compose/) | `docker compose up --build` 啟動 frontend、backend、postgres |
| 5 | 測試 + CI | [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) | backend 測 health/generate/DB insert；frontend build；GitHub Actions 綠燈 |
| 6 | Azure 部署 | [Azure Container Apps GitHub Actions](https://learn.microsoft.com/en-us/azure/container-apps/github-actions), [Azure PostgreSQL](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/quickstart-create-server-portal) | 有公開 demo URL；backend 連 Azure PostgreSQL；可完整展示 |

建議工期是 4-6 週。每天 2-3 小時時，優先順序是：

1. 先完成可跑的功能。
2. 再補測試與 Docker。
3. 最後部署與整理展示素材。

## Stage 1: FastAPI 包模型

### 學習目標

- 會建立 FastAPI app。
- 會定義 Pydantic request / response schema。
- 會把既有 diffusion model 包成可呼叫的 inference function。
- 會處理基本 validation 與錯誤回應。

### 實作內容

- 建立 backend 專案結構。
- 將現有 diffusion model 整理成：

```python
def generate_fringe(
    material_temp: float,
    mold_temp: float,
    injection_speed: float,
    packing_time: float,
    injection_pressure: float,
    seed: int,
    model_version: str,
) -> str:
    """Generate an image and return the local image path."""
```

- 建立 API：

```text
GET  /api/health
POST /api/generate
```

### KPI

- `GET /api/health` 回傳 `{"status": "ok"}`。
- `POST /api/generate` 可接收製程參數。
- API 回傳 `job_id`、`image_path`、`model_version`、`infer_ms`。
- inference 錯誤時有清楚的 `failed` response，不讓 server crash。

## Stage 2: PostgreSQL 紀錄

### 學習目標

- 會用 PostgreSQL 建 schema。
- 會用 SQL 查詢生成紀錄。
- 會讓 FastAPI 在生成流程中寫入資料庫。

### 最小資料表

```sql
model_versions (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL UNIQUE,
  checkpoint_path TEXT,
  git_sha TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

generation_jobs (
  id UUID PRIMARY KEY,
  material_temp DOUBLE PRECISION NOT NULL,
  mold_temp DOUBLE PRECISION NOT NULL,
  injection_speed DOUBLE PRECISION NOT NULL,
  packing_time DOUBLE PRECISION NOT NULL,
  injection_pressure DOUBLE PRECISION NOT NULL,
  seed INTEGER NOT NULL,
  model_version_id INTEGER NOT NULL REFERENCES model_versions(id),
  status TEXT NOT NULL,
  infer_ms INTEGER,
  error_message TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  completed_at TIMESTAMPTZ
);

generated_images (
  id UUID PRIMARY KEY,
  job_id UUID NOT NULL REFERENCES generation_jobs(id),
  image_path TEXT NOT NULL,
  width INTEGER,
  height INTEGER,
  checksum TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 展示 SQL

查某組製程參數最近生成紀錄：

```sql
SELECT *
FROM generation_jobs
WHERE material_temp = 195
  AND mold_temp = 55
ORDER BY created_at DESC
LIMIT 10;
```

查每個模型版本平均推論時間：

```sql
SELECT mv.name, AVG(gj.infer_ms) AS avg_infer_ms
FROM generation_jobs gj
JOIN model_versions mv ON gj.model_version_id = mv.id
WHERE gj.status = 'done'
GROUP BY mv.name
ORDER BY avg_infer_ms ASC;
```

查最近失敗任務：

```sql
SELECT id, status, error_message, created_at
FROM generation_jobs
WHERE status = 'failed'
ORDER BY created_at DESC
LIMIT 10;
```

### KPI

- 每次 `/api/generate` 都新增一筆 `generation_jobs`。
- 成功後新增一筆 `generated_images`。
- `status` 至少支援 `running`、`done`、`failed`。
- 可以用 SQL 查最近紀錄、平均推論時間、失敗任務。

## Stage 3: React + Vite 表單

### 學習目標

- 會用 React component 拆表單與結果顯示。
- 會用 `fetch` 呼叫 FastAPI。
- 會處理 loading / error / success state。

### 頁面範圍

只做一頁：

```text
左側：製程參數輸入
右側：生成結果圖片
下方：最近 10 筆生成紀錄
```

表單欄位：

- 料溫 `material_temp`
- 模溫 `mold_temp`
- 射速 `injection_speed`
- 保壓時間 `packing_time`
- 射壓 `injection_pressure`
- seed
- model version

### KPI

- 使用者可從瀏覽器送出生成請求。
- 送出時按鈕進入 loading 狀態。
- 成功時顯示生成圖片。
- 失敗時顯示錯誤訊息。
- 頁面可顯示最近 10 筆 jobs。

## Stage 4: Docker Compose

### 學習目標

- 會寫 backend Dockerfile。
- 會寫 frontend Dockerfile。
- 會用 Compose 串 frontend、backend、postgres。
- 會使用 environment variables。

### Compose 服務

```text
frontend
backend
postgres
```

### KPI

- `docker compose up --build` 可啟動整套 MVP。
- backend 可連到 Compose 裡的 PostgreSQL。
- frontend 可呼叫 backend API。
- 圖片輸出目錄使用 volume 保存。
- 有 `.env.example` 說明必要變數。

## Stage 5: 測試 + CI

### 學習目標

- 會用 pytest 測 FastAPI endpoint。
- 會測 request validation。
- 會測 DB insert。
- 會用 GitHub Actions 跑 backend test 與 frontend build。

### 最小測試

- `GET /api/health` returns 200。
- `POST /api/generate` 缺欄位時 returns 422。
- `POST /api/generate` 成功時會建立 job。
- `GET /api/jobs` returns list。

### CI KPI

- push / pull request 會觸發 GitHub Actions。
- backend tests 通過。
- frontend build 通過。
- CI badge 或 workflow 狀態為綠燈。

## Stage 6: Azure 部署

### 學習目標

- 會把 backend container 部署到 Azure Container Apps。
- 會建立 Azure Database for PostgreSQL。
- 會設定 backend production environment variables。
- 會讓 frontend 呼叫雲端 backend。

### MVP 部署架構

```text
React static frontend
  -> Azure Container Apps backend
  -> Azure Database for PostgreSQL
  -> backend local image volume
```

### KPI

- backend 有公開 URL。
- `/api/health` 在雲端正常。
- 雲端 backend 可連 Azure PostgreSQL。
- 前端可完成一次生成流程。
- 面試時可以展示 live demo 或部署截圖。

## Gate Checklist

### API Gate

- [ ] curl 可送出 `/api/generate`。
- [ ] response 包含 `job_id`。
- [ ] response 包含 `image_path`。
- [ ] response 包含 `model_version`。
- [ ] response 包含 `infer_ms`。

### DB Gate

- [ ] PostgreSQL 能查最近生成紀錄。
- [ ] PostgreSQL 能查各模型版本平均推論時間。
- [ ] PostgreSQL 能查指定製程參數的生成歷史。

### Web Gate

- [ ] 瀏覽器可輸入參數。
- [ ] 瀏覽器可看到生成圖片。
- [ ] 瀏覽器可看到最近 10 筆紀錄。

### Ops Gate

- [ ] `docker compose up --build` 可重現本地環境。
- [ ] `.env.example` 完整。
- [ ] backend、frontend、postgres logs 可檢查。

### Portfolio Gate

- [ ] 有公開 demo URL 或部署截圖。
- [ ] GitHub Actions 綠燈。
- [ ] 有 3 條 SQL 查詢可展示。
- [ ] 可以用 3 分鐘講清楚：模型如何被包成 API、資料如何記錄、系統如何部署。

## 每日時間配置

每天 2-3 小時時，建議比例：

- 60% 實作功能
- 20% 看當天需要的官方文件
- 10% 測試與修 bug
- 10% 記錄學到什麼與面試講法

不要先刷完整課程。專案缺什麼，就學什麼。

## Fallback

- 如果 React 卡住：先用最簡單 HTML form + fetch，之後再整理 component。
- 如果 ORM 卡住：先用 SQLAlchemy Core 或 raw SQL，確保資料能進 PostgreSQL。
- 如果 Docker 卡住：先用本機 `uvicorn` + `npm run dev` 跑通，再回來 Docker 化。
- 如果 Azure 卡住超過 2 天：先保留本地 Docker Compose demo，補部署筆記與截圖，之後再完成雲端。

## 求職展示句

中文：

> 我把已完成的 diffusion model 包成一個可部署的工業 AI MVP。使用者輸入射出成型製程參數後，FastAPI 會呼叫模型生成光彈圖，並把參數、模型版本、任務狀態、推論時間與圖片路徑寫入 PostgreSQL。整套系統可以用 Docker Compose 本地重現，也能部署到 Azure。

英文：

> Built a deployable industrial AI MVP that serves an existing diffusion model through FastAPI, records generation jobs and model metadata in PostgreSQL, provides a React + Vite interface for process-parameter input, and supports local Docker Compose execution plus Azure deployment.
