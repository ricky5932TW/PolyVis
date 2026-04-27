# 8 週執行手冊：DDIM Photoelastic Generator → Production

> 客製版本｜路線：平均（MLE / Full-stack / Backend with ML 三向均衡）
> 模型基準：DDIM 50 步，採取「減步 + 量化 + GPU fallback」優化路線
> 預估每週投入：30–40 小時（如果是全職衝刺，週末可額外加碼）

---

## 0. 全局原則（不可妥協）

1. **每週日晚上 30 分鐘 retrospective**：寫下完成 / 未完成 / 學到什麼 / 下週調整。這個習慣本身就是 PM 訊號。
2. **Public by default**：所有 code push 到 GitHub public repo，commit message 認真寫。面試官會看 commit history。
3. **每週至少一次 git tag**：`v0.1-week1`, `v0.2-week2`...，方便回溯也方便面試展示「進度感」。
4. **卡關超過 4 小時 → 先記下、跳過、週末回頭**。不要在一個技術點上死磕兩天，會拖垮整個 timeline。
5. **錢的紅線**：AWS billing alert 設 USD $30 / 月。Modal / Replicate 用免費額度。
6. **Demo URL 要從 Week 5 起一直可用**。你會反覆把這個 URL 丟給面試官，cold start 不能超過 5 秒。

---

## 1. 模型推論優化路線（你的 DDIM 專屬）

這條路線的每一步都是面試素材，請按順序做、留下 benchmark 數字。

| 階段 | 動作 | 預期 latency（256×256） | 何時做 |
|------|------|------------------------|--------|
| 0 | Baseline: PyTorch CPU, 50 steps | ~30–60s | Week 1 結尾 |
| 1 | 降步：25 steps | ~15–30s | Week 4 |
| 2 | 降步：15 steps + 比較 SSIM/PSNR | ~10–18s | Week 4 |
| 3 | ONNX export + ONNX Runtime CPU | ~6–12s | Week 7（加分） |
| 4 | Modal serverless GPU fallback（可選） | ~1–3s | Week 7（加分） |

**重要**：每個階段都用同一組固定 random seed + 同一組製程參數做 benchmark，把結果存進一個 `benchmarks.md`，包含：
- Latency p50 / p95
- 生成圖與 baseline 的 SSIM / PSNR / LPIPS
- 視覺比較圖（一張 grid）

這個檔案你會放進 README，面試時直接秀。

---

## 2. Timeline 總覽

| 週 | 主題 | 主菜（必做） | 加分菜方向 | Demo 狀態 |
|----|------|--------------|-----------|-----------|
| 1 | JS/TS + 後端骨架 | hello-world 全棧 + Docker | MLE: model wrap | Local |
| 2 | DB + SQL | PostgreSQL schema + ORM + migration | Backend: 進階 SQL | Local |
| 3 | 前端 UI + 模型整合 | 完整參數表單 + 結果展示 | Full-stack: UX 細節 | Local |
| 4 | Async 架構 + 模型優化 | Celery + Redis + S3 + 減步 | MLE: benchmark | Local + S3 |
| 5 | AWS 部署 | ECS Fargate + RDS + Vercel | Backend: VPC | **Public URL** |
| 6 | CI/CD + 監控 + Auth | GitHub Actions + Sentry + OAuth | Full-stack: UI polish | Public URL + auto deploy |
| 7 | 性能 + ML 優化 | Cache + rate limit + ONNX | MLE: GPU fallback | 優化版上線 |
| 8 | 求職衝刺 | Demo 影片 + 文章 + 投履歷 | — | 完成品 |

---

## 3. 每週詳細計畫

### 📌 Week 1：JS/TS 急救包 + 後端骨架

**這週你要回答的問題**：「我能不能在 5 分鐘內把整套東西在新電腦上跑起來？」

#### 主菜（必做）
- [ ] JavaScript ES6+ 核心：解構、spread、async/await、modules
- [ ] TypeScript 基礎：type vs interface、泛型、utility types
- [ ] React 核心 hooks：useState, useEffect, useRef
- [ ] Next.js App Router 基礎：page、layout、server component vs client component
- [ ] FastAPI quickstart + Pydantic v2
- [ ] Docker 基礎 + 寫一份 Dockerfile + docker-compose.yml
- [ ] **建立 monorepo 結構**：

```
ddim-photoelastic/
├── frontend/          (Next.js)
├── backend/           (FastAPI)
├── ml/                (你的 DDIM 模型 + 推論 wrapper)
├── docker-compose.yml
├── README.md
└── docs/
    ├── architecture.md
    └── benchmarks.md
```

#### 加分菜（MLE 方向）
- [ ] 把 DDIM 模型包成一個 `Inferencer` class，介面長這樣：
```python
class DDIMInferencer:
    def __init__(self, model_path: str, num_steps: int = 50): ...
    def generate(self, params: dict) -> np.ndarray: ...
    def warmup(self): ...
```
- [ ] 跑出 baseline benchmark 寫進 `docs/benchmarks.md`

#### Daily Breakdown
- **Mon**：JS/TS（4hr）+ react.dev tutorial 前半
- **Tue**：react.dev tutorial 後半 + 開 Next.js hello world
- **Wed**：FastAPI 官方文件 chapters 1-5 + 寫 health check API
- **Thu**：Next.js 打 FastAPI、CORS 處理、TypeScript types 對齊
- **Fri**：Docker + docker-compose 把兩個 service 包起來
- **Sat**：把 DDIM 模型載入 backend，做最簡單的 `/generate` endpoint
- **Sun**：retrospective + git tag `v0.1`

#### Deliverable Checklist
- [ ] `docker-compose up` 能同時啟動 frontend + backend
- [ ] 訪問 `localhost:3000` 看到簡陋 UI
- [ ] UI 能 call backend `/health` 顯示 OK
- [ ] backend `/generate` 接受 JSON params 回傳一張圖（base64 即可）
- [ ] README 有「How to run locally」段落

#### Fallback（如果卡住）
- React 不熟 → 改用 Next.js + 完全不寫 client component，用純 server form action
- TypeScript 卡住 → 第一週允許用 JavaScript，Week 6 polish 階段再加上 TS
- Docker 卡住 → 先本地 venv + npm 跑，Week 5 上雲前再容器化

---

### 📌 Week 2：DB 設計 + SQL 紮根

**這週你要回答的問題**：「給你一個 100 萬筆 generations 的表，怎麼設計才不會慢？」

#### 主菜（必做）
- [ ] 起 PostgreSQL（Docker container）
- [ ] SQL 紮實練習（每天一塊）：
  - JOIN 三種類型實作差異
  - GROUP BY + HAVING + window functions
  - 子查詢 vs CTE
  - Index：B-tree、partial index、composite index 順序
  - Transaction、isolation levels（重點：READ COMMITTED vs REPEATABLE READ）
  - `EXPLAIN ANALYZE` 讀懂 query plan
- [ ] LeetCode SQL 30 題（Easy 15 + Medium 15）
- [ ] SQLAlchemy 2.0 ORM（用新版 syntax，不要看到舊文件）
- [ ] Alembic migration：autogenerate + manual edit
- [ ] **設計你的 schema**（建議起點）：

```sql
-- users
id (uuid, pk)
email (text, unique)
created_at (timestamptz)

-- generations
id (uuid, pk)
user_id (fk users)
params (jsonb)              -- 製程參數彈性存
num_steps (int)             -- 記錄當次用幾步
status (enum: pending/running/success/failed)
image_s3_key (text, null)
error_message (text, null)
duration_ms (int, null)
created_at (timestamptz)
completed_at (timestamptz, null)

-- parameter_presets
id (uuid, pk)
user_id (fk users)
name (text)
params (jsonb)
created_at (timestamptz)

-- 必要 index
idx_generations_user_created (user_id, created_at desc)
idx_generations_status (status) WHERE status IN ('pending','running')
gin index on generations.params  -- 之後可以查 params 內容
```

#### 加分菜（Backend 方向）
- [ ] 寫一段「為什麼選 jsonb 不選 JSON」的決策記錄
- [ ] 加 audit log 表 + trigger 自動記錄 generations 變更
- [ ] 用 `EXPLAIN ANALYZE` 對比有沒有 index 的 query 速度，截圖

#### Daily Breakdown
- **Mon**：PostgreSQL 起來 + SQLZoo SELECT/JOIN
- **Tue**：GROUP BY、subquery、CTE + LeetCode SQL Easy 15
- **Wed**：Index 觀念 + EXPLAIN + LeetCode SQL Medium 8
- **Thu**：Transaction、isolation + LeetCode SQL Medium 7
- **Fri**：SQLAlchemy 2.0 + 設計 schema
- **Sat**：寫 migrations + seed script + CRUD endpoints
- **Sun**：retrospective + git tag `v0.2`

#### Deliverable Checklist
- [ ] PostgreSQL 在 docker-compose 裡跑起來
- [ ] Alembic migration 能 up / down 不出錯
- [ ] `/api/generations` GET / POST endpoint 能跑
- [ ] 至少 3 個 query 有對應 index 並用 EXPLAIN 證明
- [ ] LeetCode SQL 30 題完成（截圖證據）

#### Fallback
- SQLAlchemy 太複雜 → 先用 raw SQL + asyncpg，Week 7 再 refactor
- Migration 卡關 → 直接 drop 重建（local 沒差），focus 把 schema 搞對

---

### 📌 Week 3：前端 UI + 模型整合

**這週你要回答的問題**：「不工程師的人能不能 30 秒內理解這個介面在做什麼？」

#### 主菜（必做）
- [ ] Tailwind CSS + shadcn/ui 設定好
- [ ] react-hook-form + zod：型別安全的表單
- [ ] 設計 UI：
  - 左側：參數輸入區（製程參數 + DDIM steps slider）
  - 右側：結果顯示（最新生成 + 歷史 grid）
  - 上方：preset 選擇器
- [ ] 前後端串接：submit → 顯示 loading → 拿到結果
- [ ] 圖片用 base64 暫時 OK（Week 4 再改 S3）

#### 加分菜（Full-stack 方向）
- [ ] 加一個「參數比較模式」：兩組參數並排生成
- [ ] Skeleton loading + 錯誤 toast
- [ ] Dark mode（shadcn/ui 預設支援，免費加分）
- [ ] 用 `next/image` 做圖片 lazy load
- [ ] Lighthouse score > 90 截圖留證

#### Daily Breakdown
- **Mon**：Tailwind 速成 + shadcn/ui 安裝 + 抄 layout
- **Tue**：表單 + zod schema + 前端 validation
- **Wed**：API call + loading state + error handling
- **Thu**：歷史 grid + 從 DB 撈使用者的 generations
- **Fri**：Preset CRUD UI
- **Sat**：UI polish + 響應式 + 跑 Lighthouse
- **Sun**：retrospective + git tag `v0.3`

#### Deliverable Checklist
- [ ] 參數表單可以 submit 並拿到生成圖
- [ ] 歷史記錄顯示出來
- [ ] Preset 可以存 / 載入
- [ ] Mobile view 不會爆版
- [ ] 截圖三張 UI 放進 README（這個很重要，面試官懶得自己跑）

#### Fallback
- 設計能力不夠 → 直接用 [Vercel templates](https://vercel.com/templates) 找一個改
- shadcn 學不動 → 純 Tailwind + 抄 [Tailwind UI](https://tailwindui.com) 免費的 component

---

### 📌 Week 4：Async 架構 + 模型優化

**這週是整個專案的技術核心**。完成這週你的專案就從「toy」進入「production-grade」。

#### 主菜（必做）
- [ ] Redis（Docker）+ Celery worker
- [ ] 改寫 `/generate` 成 async pattern：
  ```
  POST /api/generations  → 回 task_id, status=pending
  GET  /api/generations/:id → poll status
  ```
- [ ] Worker 跑完後上傳到 S3（local 用 MinIO 模擬）
- [ ] 前端改成 polling（每 2 秒）或 SSE
- [ ] **DDIM 減步實驗**：
  - 25 steps benchmark
  - 15 steps benchmark
  - 視覺比較圖
  - 寫進 `docs/benchmarks.md`
- [ ] **畫架構圖**（excalidraw 或 draw.io），包含：
  - User → Vercel → ALB → ECS API → SQS/Redis → ECS Worker → S3
  - DB 連到 API + Worker
  - 把這張圖放進 README 顯眼位置

#### 加分菜（MLE 方向）
- [ ] 加一個 `/api/benchmark` endpoint，給定 params 回傳不同 steps 的 latency 比較
- [ ] Worker 加 retry 機制 + dead letter queue
- [ ] Prometheus metrics（即使不部署，先埋點）

#### Daily Breakdown
- **Mon**：Redis + Celery quickstart
- **Tue**：把 generation 改成 task，前端改 polling
- **Wed**：MinIO 起來 + presigned URL upload
- **Thu**：DDIM 減步實驗 + benchmark
- **Fri**：架構圖 + 文件
- **Sat**：整合測試 + buffer day
- **Sun**：retrospective + git tag `v0.4`

#### Deliverable Checklist
- [ ] 同時送 5 個 generation 不會卡
- [ ] 圖片從 S3/MinIO 取得（不再 base64）
- [ ] benchmarks.md 有 50/25/15 steps 三組數據 + 視覺比較
- [ ] 架構圖存在 `docs/architecture.png`
- [ ] README 顯示架構圖

#### Fallback
- Celery 太複雜 → 用 FastAPI 的 BackgroundTasks（簡單版，後面再升級）
- S3 / MinIO 設定卡住 → 先存 local volume，Week 5 上雲時直接接 S3

---

### 📌 Week 5：AWS 部署（最痛的一週）

**這週你要回答的問題**：「你的 demo URL 在哪？」（從這週起每次面試前都要先確認還活著）

⚠️ **開工前先做**：
1. AWS Billing → 設 alert at $30
2. 開一個獨立 AWS account 給這個專案（不要混用）
3. 用 IAM user 不要用 root

#### 主菜（必做）
- [ ] AWS 基礎概念釐清：IAM、VPC、Security Group、Subnet
- [ ] ECR：把 backend 跟 worker 兩個 image push 上去
- [ ] RDS PostgreSQL（db.t4g.micro，最便宜）+ Alembic 在新環境跑 migration
- [ ] ElastiCache Redis（cache.t4g.micro）或 self-host 在 ECS（省錢）
- [ ] ECS Fargate cluster：
  - Service 1: API（1 task）
  - Service 2: Worker（1 task）
- [ ] ALB + 健康檢查
- [ ] S3 bucket + CloudFront for images
- [ ] Vercel 部署 Next.js（環境變數指向 AWS API）
- [ ] 買網域（Route 53 或 Namecheap），綁 HTTPS

#### 加分菜（Backend 方向）
- [ ] VPC 切 public/private subnet，DB 放 private
- [ ] Secrets Manager 管 DB password
- [ ] CloudWatch alarm：API 5xx > 1% 警報

#### Daily Breakdown
- **Mon**：AWS 基礎 + IAM 設定 + ECR push
- **Tue**：RDS 起來 + 連線測試 + migration
- **Wed**：ECS task definition + service（最容易卡這天）
- **Thu**：ALB + S3 + CloudFront
- **Fri**：Vercel 部署 + 網域 + HTTPS
- **Sat**：整合測試 + buffer day
- **Sun**：retrospective + git tag `v0.5` 🎉 **第一個 public URL**

#### Deliverable Checklist
- [ ] 公開 URL 任何人打開可用
- [ ] HTTPS 正常
- [ ] 從新瀏覽器（無快取）測試完整 flow OK
- [ ] AWS cost 預估 < $30/月（用 Cost Explorer 看）
- [ ] README 加上 "Live Demo" 連結

#### Fallback（你會用上的）
- ECS 設定鬼打牆 > 2 天 → **直接降級用 Render.com**
  - Render 部署 FastAPI + worker：5 分鐘
  - Render PostgreSQL + Redis 內建
  - 把 AWS 的努力寫成 `docs/aws-migration-plan.md`，附架構圖
  - 面試時這樣講：「我先用 Render 快速驗證，AWS migration plan 已經設計好（秀架構圖），等流量起來才需要遷移」
- 模型 image 太大（>2GB）→ 用 multi-stage build + 把 model weight 放 S3，啟動時下載
- GPU 真的需要 → Modal serverless GPU，從 worker call Modal endpoint

---

### 📌 Week 6：CI/CD + 監控 + Auth

**這週你要回答的問題**：「如果半夜壞了你怎麼知道？」

#### 主菜（必做）
- [ ] GitHub Actions pipeline：
  - PR：lint + test
  - Merge to main：build + push ECR + update ECS
- [ ] 寫測試：
  - Backend: pytest 至少 10 個 unit test + 3 個 integration test
  - Frontend: Playwright 1 個 E2E（從填表單到看到結果）
- [ ] CloudWatch logs 集中 + 一個 simple dashboard
- [ ] Sentry 接前後端
- [ ] NextAuth.js + Google OAuth
- [ ] 把 generations 跟 user 關聯（之前可能用 anonymous）

#### 加分菜（Full-stack 方向）
- [ ] 個人化首頁：「歡迎回來，你最近生成了 X 張」
- [ ] 公開 / 私人 generation toggle
- [ ] Share link 功能

#### Daily Breakdown
- **Mon**：GitHub Actions 跑 lint + test
- **Tue**：build + deploy pipeline
- **Wed**：寫 backend test
- **Thu**：Playwright E2E + Sentry
- **Fri**：NextAuth + Google OAuth
- **Sat**：把 user 串進整個 flow + buffer
- **Sun**：retrospective + git tag `v0.6`

#### Deliverable Checklist
- [ ] PR 有自動 CI status
- [ ] Merge 後自動部署，3 分鐘內生效
- [ ] 故意丟一個錯誤，Sentry 收到通知
- [ ] Google login 可用
- [ ] 各環境變數整理在 `.env.example`

#### Fallback
- ECS 自動部署設不起來 → 改用 GitHub Actions SSH 進去手動更新（先求有再求好）
- Auth 卡住 → 先做 magic link（更簡單）或暫時跳過，Week 7 補

---

### 📌 Week 7：性能 + ML 優化 + 收尾

**這週你要回答的問題**：「為什麼是 X 不是 Y？」（每個技術決策都要有答案）

#### 主菜（必做）
- [ ] **Cache layer**：相同 params + same seed → 直接回 cached 圖（Redis，TTL 7 天）
- [ ] **Rate limiting**：每 user 每分鐘最多 10 次（slowapi 或 nginx）
- [ ] **DB index 優化**：用 `pg_stat_statements` 找慢 query，補 index
- [ ] **架構決策記錄（ADR）**至少 5 篇，每篇一頁，主題建議：
  1. 為什麼選 ECS Fargate 不選 K8s
  2. 為什麼選 Celery 不選 SQS + Lambda
  3. 為什麼選 PostgreSQL 不選 MongoDB
  4. 為什麼選 Next.js 不選純 React + Vite
  5. 為什麼用 polling 不用 WebSocket
- [ ] README 大改寫（範本見下方第 4 節）

#### 加分菜（MLE 方向，挑 1–2 個）
- [ ] ONNX export + ONNX Runtime（CPU 推論加速 30-50%）
- [ ] Modal serverless GPU fallback（流量大時切換）
- [ ] Model A/B test：兩個模型 weight 並存，按 user_id hash 分流
- [ ] Generation quality dashboard（SSIM 分布、failure rate）

#### Daily Breakdown
- **Mon**：cache 邏輯
- **Tue**：rate limiting + 慢 query 優化
- **Wed–Thu**：寫 5 篇 ADR
- **Fri**：README 大改寫
- **Sat**：加分菜挑一個
- **Sun**：retrospective + git tag `v1.0` 🎉

#### Deliverable Checklist
- [ ] 重複 params 第二次 < 100ms
- [ ] Rate limit 起作用（手動測試確認）
- [ ] `docs/adr/` 至少 5 篇 markdown
- [ ] README 含：架構圖、tech stack、live demo、benchmark、how to run

---

### 📌 Week 8：求職衝刺

**這週你要回答的問題**：「我要怎麼把這 7 週的努力變成 offer？」

#### 主菜（必做）
- [ ] **Demo 影片（3–5 分鐘）**腳本：
  1. (15s) 問題：DDIM 模型怎麼從 research notebook 變 production？
  2. (30s) Live demo：填參數 → 拿圖 → 看歷史
  3. (60s) 架構圖 walkthrough
  4. (60s) 三個有趣的工程決策（從 ADR 挑）
  5. (30s) Benchmark 數字
  6. (15s) 邀請聯絡
  - 上傳 YouTube unlisted，連結放 README 跟履歷
- [ ] **技術文章一篇**（中文發 Medium，英文發 dev.to）：
  - 標題建議：「我把 diffusion model 部署到 production 學到的 5 件事」
  - 字數 1500–2500，含程式碼片段、架構圖
- [ ] **LinkedIn post**（中英雙語）秀專案
- [ ] **履歷更新**，這個專案放最上面，包含：
  - 一句 elevator pitch
  - 3 個技術 highlight，每個含**數字**：
    - 「將 DDIM 推論 latency 從 50s 降至 12s（76%）透過 step reduction + ONNX」
    - 「設計 async architecture 支援 N concurrent generations，p95 < X ms」
    - 「CI/CD pipeline 部署時間從手動 30 分鐘 → 自動 3 分鐘」
- [ ] **投履歷 30 家**：
  - 台灣：104、CakeResume、LinkedIn
  - 海外遠端：[Hacker News Who's Hiring](https://news.ycombinator.com/jobs)、[Wellfound](https://wellfound.com)
  - 直接 cold email 想去的公司 hiring manager（LinkedIn 找）

#### Daily Breakdown
- **Mon**：錄 demo 影片 + 剪輯
- **Tue–Wed**：寫技術文章
- **Thu**：履歷大改 + LinkedIn 翻新
- **Fri–Sun**：投履歷 + 練 behavioral question

#### Deliverable Checklist
- [ ] YouTube demo 影片 unlisted 連結
- [ ] Medium / dev.to 文章上線
- [ ] LinkedIn post 至少 100 impressions
- [ ] 履歷投出 30 家
- [ ] 至少 5 個面試話術 ready（見第 5 節）

---

## 4. README 範本（Week 7 用）

```markdown
# DDIM Photoelastic Generator

> Production-grade web app that generates photoelastic images from injection
> molding process parameters using a Denoising Diffusion Implicit Model.

🔗 [Live Demo](https://yours.dev) · 🎥 [Video](https://youtu.be/xxx) · 📄 [Paper](link)

![Architecture](docs/architecture.png)

## Why this exists
[3–4 句問題定義 + 你的 contribution]

## Tech Stack
| Layer | Tech | Why |
|-------|------|-----|
| Frontend | Next.js 14, TypeScript, Tailwind, shadcn/ui | ... |
| Backend | FastAPI, SQLAlchemy 2.0, Alembic | ... |
| ML Inference | PyTorch, ONNX Runtime | ... |
| Async | Celery + Redis | ... |
| Database | PostgreSQL 16 | ... |
| Storage | S3 + CloudFront | ... |
| Infra | AWS ECS Fargate, RDS, Vercel | ... |
| Observability | CloudWatch, Sentry | ... |
| CI/CD | GitHub Actions | ... |

## Performance
| Variant | Steps | Latency p50 | SSIM vs baseline |
|---------|-------|-------------|------------------|
| Baseline | 50 | 45s | 1.000 |
| Reduced | 25 | 23s | 0.987 |
| Reduced | 15 | 14s | 0.961 |
| ONNX | 15 | 9s | 0.961 |

## Architecture Decisions
- [ADR-001: Why ECS Fargate over Kubernetes](docs/adr/001.md)
- [ADR-002: Why Celery over SQS+Lambda](docs/adr/002.md)
- ...

## Run Locally
\`\`\`bash
cp .env.example .env
docker-compose up
\`\`\`

## Project Status
Built solo over 8 weeks (Apr–Jun 2026). Open to feedback / collaboration.
```

---

## 5. 面試話術 cheatsheet（Week 7–8 開始記）

### 自我介紹版本（30 秒）
> 「我背景是材料 / 製程相關研究，研究主題是用 DDIM 做射出成形的光彈圖生成。為了把這個研究級的模型變成能讓工廠端真的用的工具，我花 8 週時間把它包成完整的 web application，從前端 React、後端 FastAPI、async architecture 到 AWS 部署都自己做。專案 live URL 在 ___，原始碼在 ___。」

### 「你最 proud 的技術決策」
挑 ADR 裡你最有感的一個，用 STAR：
- Situation：DDIM 50 steps CPU 推論要 45 秒，user 不可能等
- Task：要在不犧牲圖像品質下加速
- Action：實驗 step reduction（50→25→15）、量化 SSIM 對比、用 ONNX Runtime
- Result：latency 降到 9 秒，SSIM 維持 0.96+

### 「你 debug 過最難的問題」
從 retrospective 裡撈一個你卡 > 1 天的問題。

### 「為什麼選 X 不選 Y」
你 ADR 寫過的 5 個決策直接背。

### 「如果讓你重做你會怎麼改」
- 「Worker 用 SQS + Lambda 取代 Celery，serverless cost 更低」
- 「加上 vector DB 搜尋相似 generation」
- 「模型用 distillation 進一步壓到 5 步」

---

## 6. 緊急 Fallback 決策樹

```
進度落後 1 週：
  ├─ 砍加分菜，只做主菜
  └─ Week 7 加分項 ONNX / Modal 全砍

進度落後 2 週：
  ├─ AWS → 改 Render.com（即使 Week 5 已經做一半）
  ├─ E2E test 砍掉，只留 unit test
  └─ ADR 從 5 篇降到 3 篇

進度落後 3 週：
  ├─ Week 8 投履歷照投，專案標 "in progress"
  ├─ Demo 影片改成簡報截圖
  └─ 不要硬撐到 100% 才開始投，70% 完成度也能拿面試

任何時候卡住 > 4 小時：
  └─ 寫進「卡關 log」、跳過、週末再戰
```

---

## 7. 成本預估

| 項目 | 月成本（USD） |
|------|--------------|
| AWS ECS Fargate（2 small tasks）| 15–20 |
| AWS RDS db.t4g.micro | 12–15 |
| AWS S3 + CloudFront | 1–3 |
| Domain（Namecheap）| 1（折算） |
| Vercel hobby | 0 |
| Sentry free tier | 0 |
| GitHub Actions | 0 |
| **Total** | **~$30–40 / 月** |

> 求職期間（Week 8 之後）建議再撐 2 個月維持 demo URL，總投資約 USD $90，這是你拿 offer 最便宜的路徑。

---

## 8. 每週日 Retrospective 模板

複製貼上到 `docs/retro/week-N.md`：

```markdown
# Week N Retrospective

## 完成
- ...

## 沒完成 / 延後
- ...

## 學到的最重要的一件事
（一段話，有點哲學沒關係）

## 卡關紀錄
| 問題 | 卡關時間 | 怎麼解決 |
|------|---------|----------|
| ... | ... | ... |

## 下週調整
- ...

## 心情指數（1–10）跟原因
```

這個檔案到 Week 8 你會有 8 篇——這本身就是面試素材，秀給面試官看「這個人會反思」。

---

## 9. 你的第一個 action（今天就做）

1. 開一個 GitHub repo `ddim-photoelastic`，public
2. 把這份計畫貼進 `docs/plan.md`
3. 把 Week 1 deliverable checklist 變成 GitHub Issues
4. 設好 AWS billing alert
5. 訂今天起每週日晚上 9 點的 retrospective 行事曆提醒

完成這 5 件事，你已經比 90% 「想做專案的人」往前走了。下週我們 Week 1 結束時來檢視進度。

加油 🚀
