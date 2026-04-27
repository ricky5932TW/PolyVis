# 8 週執行手冊：DDIM 光彈圖生成系統 → 求職專案

> **目標**：將 DDIM 模型工程化成 production-grade 系統，同時建立 MLE / Full-stack / Backend 三方向的面試素材。
>
> **路線**：平均路線（不偏廢任一方向）
> **推論硬體**：CPU（含優化 track）
> **總時長**：8 週，每週預估投入 25–35 小時

---

## 一、戰略定位

**三種面試敘事，同一個 codebase**：

| 投遞方向 | 面試時的核心敘事 | 主要 talking point |
|---|---|---|
| MLE / MLOps | 把研究模型優化並部署上 production | Inference 延遲從 X 砍到 Y、async 架構、模型監控 |
| Full-stack | 從零打造 ML-powered web product | Next.js + TypeScript、auth、UX 設計、E2E 測試 |
| Backend | 設計可擴展的 ML serving 系統 | 系統架構、DB schema、佇列、cache、rate limit |

**履歷上同一個專案，cover letter 描述微調即可。**

---

## 二、技術選型（已敲定）

```
Frontend   : Next.js 14 (App Router) + TypeScript + Tailwind + shadcn/ui
Backend    : FastAPI + Pydantic v2 + SQLAlchemy 2.0 + Alembic
Queue      : Celery + Redis
DB         : PostgreSQL 16
Storage    : AWS S3 (圖片) + CloudFront (CDN)
Auth       : NextAuth.js + Google OAuth
ML         : PyTorch → ONNX Runtime（CPU 量化版）
Container  : Docker + docker-compose
Cloud      : AWS ECS Fargate + RDS + ElastiCache + S3
CI/CD      : GitHub Actions
監控       : CloudWatch + Sentry
前端部署   : Vercel（免費、快、跟 Next.js 完美整合）
```

---

## 三、整體里程碑

```
Week 1 ────── 開發環境就緒
Week 2 ────── DB schema 完成，後端 CRUD 通
Week 3 ────── 前端 UI + 模型本地接通
Week 4 ────── Async 架構 + 推論優化（關鍵週）
Week 5 ────── 部署到 AWS（公開 URL）
Week 6 ────── CI/CD + Auth + 監控
Week 7 ────── 系統打磨 + 文件
Week 8 ────── 履歷、demo 影片、投遞
```

---

# 各週詳細計畫

## Week 1：環境就緒 + 全棧 Hello World

### 目標
能用 docker-compose 起一個 Next.js + FastAPI + PostgreSQL 的本地環境，前端能打到後端。

### 學習任務
- [ ] JavaScript ES6+：解構、spread、async/await、模組（4 小時）
- [ ] TypeScript 基礎：type vs interface、generics、utility types（3 小時）
- [ ] React 核心觀念：useState、useEffect、props drilling、條件/列表渲染（react.dev tutorial 全做完）
- [ ] FastAPI quickstart：path params、Pydantic models、dependency injection（官方文件）
- [ ] Docker：Dockerfile 寫法、image vs container、volumes、networks
- [ ] docker-compose 多服務串接

### 實作 Deliverables
- [ ] `docker-compose.yml` 起 Next.js + FastAPI + Postgres
- [ ] `/api/health` endpoint 回傳 `{ "status": "ok", "db": "connected" }`
- [ ] Next.js 頁面打 `/api/health` 顯示結果
- [ ] Repo 推上 GitHub，README 寫好啟動指令

### 三方向 Talking Point 種子
- **Full-stack**: 「我用 docker-compose 把整個開發環境容器化，新成員 5 分鐘上手」
- **Backend**: 「FastAPI 的 dependency injection 讓我能乾淨地處理 DB session lifecycle」
- **MLE**: 暫無

### 卡關 Fallback
- TypeScript 撞牆 → 暫時用 JavaScript，第 6 週再改 TS
- Docker 一直跑不起來 → 用 native install 先做，第 5 週上雲前再補 Docker

---

## Week 2：SQL 紮根 + DB 層完成

### 目標
PostgreSQL 操作流暢、Schema 設計合理、能寫出 production-quality 的 SQL query。

### 學習任務
- [ ] **SQL 紮實練習**（這週重點）：
  - SQLZoo 全部章節（約 8 小時）
  - LeetCode SQL 50 題（Easy 30 + Medium 20）
  - 重點主題：JOIN（LEFT/INNER/SELF）、GROUP BY + HAVING、子查詢、CTE、window functions、index、transaction isolation level
- [ ] PostgreSQL 特色：JSONB、array type、`EXPLAIN ANALYZE`
- [ ] SQLAlchemy 2.0 ORM（async 版本）+ Alembic migration

### Schema 設計
```sql
users (
  id UUID PRIMARY KEY,
  email VARCHAR UNIQUE,
  google_id VARCHAR UNIQUE,
  created_at TIMESTAMP
)

generations (
  id UUID PRIMARY KEY,
  user_id UUID FK,
  params JSONB,           -- 製程參數
  status VARCHAR,          -- queued/processing/done/failed
  image_s3_key VARCHAR,
  inference_ms INT,        -- 推論延遲（之後拿來做 metrics）
  ddim_steps INT,
  model_version VARCHAR,   -- v1_pytorch_fp32 / v2_onnx_int8
  created_at TIMESTAMP,
  completed_at TIMESTAMP
)

parameter_presets (
  id UUID PRIMARY KEY,
  user_id UUID FK,
  name VARCHAR,
  params JSONB,
  is_public BOOLEAN
)

CREATE INDEX idx_generations_user_created ON generations(user_id, created_at DESC);
CREATE INDEX idx_generations_status ON generations(status) WHERE status IN ('queued', 'processing');
```

### 實作 Deliverables
- [ ] Alembic migration 全套
- [ ] FastAPI CRUD endpoints（POST/GET/LIST generations、CRUD presets）
- [ ] Postman / Bruno collection 全 endpoint 可測
- [ ] pytest 跑 happy path 測試

### 三方向 Talking Point 種子
- **Backend**: 「Generations 表我加了 partial index 只 cover queued/processing 狀態，把 worker 的 polling query 從 full table scan 變 index scan」
- **Full-stack**: 暫無
- **MLE**: 「DB 把 model_version 跟 inference_ms 存起來，後面做 A/B test 跟監控就直接 SQL 查」

### 卡關 Fallback
- ORM 太燒腦 → 直接寫 raw SQL + `asyncpg`，業界也很多人這樣做
- Alembic 設定不順 → 先手動 migration 跑通再說

---

## Week 3：前端打磨 + 模型本地接通

### 目標
有一個看起來像樣的 UI（不是 default 黑白），且能填參數送出後跑出圖（同步阻塞版本，下週改 async）。

### 學習任務
- [ ] Next.js App Router：Server Components vs Client Components 差別
- [ ] react-hook-form + zod：表單驗證業界標配
- [ ] Tailwind CSS（不要自己寫 CSS）
- [ ] shadcn/ui：直接 `npx shadcn-ui add` 拿元件
- [ ] PyTorch 模型 → FastAPI endpoint 包裝

### 實作 Deliverables
- [ ] **首頁**：簡介 + 範例圖 + CTA
- [ ] **生成頁**：
  - 表單（每個參數都有 label、單位、合理範圍 validation）
  - "Use preset" dropdown
  - "Generate" 按鈕（loading state）
  - 結果區（圖片 + 用了什麼參數 + 推論時間）
- [ ] **歷史頁**：列出過去的 generations，點擊看 detail
- [ ] FastAPI `/api/generations` POST endpoint，內部直接呼叫 PyTorch 模型回圖（同步、會卡，下週解決）
- [ ] Tailwind + shadcn 弄一個你不會嫌醜的設計

### 三方向 Talking Point 種子
- **Full-stack**: 「我用 react-hook-form + zod 做 client-side 驗證，FastAPI 用同一份 schema (Pydantic) 做 server-side，type-safe across the stack」
- **Backend**: 暫無
- **MLE**: 「先做 baseline 同步版本，量到 p95 latency 是 X 秒，下週要解掉」

### 卡關 Fallback
- 設計太醜 → 找一個 Tailwind 模板（如 [Tailwind UI](https://tailwindui.com) 免費部分、[Vercel templates](https://vercel.com/templates)）抄
- 模型載入慢 → 先 mock 一張固定圖回傳，先把 flow 通了再說

---

## Week 4：Async 架構 + 推論優化（最關鍵的一週）

### 目標
把同步阻塞改成 production-grade async，**且**完成一輪 CPU 推論優化，留下 benchmark 數據。

### Part A：Async 架構（前 4 天）

**架構圖（一定要畫出來，README 要用）**：
```
[Browser]
    │ 1. POST /api/generations { params }
    ▼
[Next.js API Route] ──► [FastAPI] ──► [DB: insert generation, status=queued]
                                  │
                                  ├──► [Redis: enqueue task]
                                  │
                                  ▼  立即回 { generation_id }
[Browser]
    │ 2. polling GET /api/generations/{id} (or SSE)

[Celery Worker] (常駐)
    │ pick task
    ▼
[ONNX Runtime: 跑模型]
    │
    ▼
[S3: 上傳圖片] ──► [DB: update status=done, image_s3_key, inference_ms]
    │
    ▼
[Browser 下次 polling 拿到 done + presigned URL]
```

- [ ] Celery + Redis 設定，worker 跟 API 拆成兩個 service
- [ ] S3 boto3 上傳邏輯
- [ ] Presigned URL 邏輯（前端直接從 S3 抓圖，不過你的 server）
- [ ] 前端改 polling 機制（每 2 秒查一次直到 done）

### Part B：推論優化 Track（後 3 天，這是 MLE 黃金素材）

**先建立 baseline benchmark script**：
```python
# benchmark.py
# 跑 100 次推論，記錄 p50/p95/p99 latency、peak memory、輸出品質
# 輸出品質可用：跟 ground truth 的 SSIM、或人眼可辨的對比圖
```

| Stage | 優化手段 | 預期 speedup | 風險 |
|---|---|---|---|
| 0 baseline | PyTorch fp32, DDIM 50 步 | 1.0x | — |
| 1 | DDIM 步數 50 → 20 | 2.5x | 品質可能微降，要量 SSIM |
| 2 | DDIM 步數 20 → 10 | 5x | 品質降更多，可能要妥協 |
| 3 | torch.compile()（PyTorch 2.x） | +20–50% | PyTorch 版本要夠新 |
| 4 | export to ONNX + ONNX Runtime | +50–150% | export 過程可能踩坑 |
| 5 | ONNX dynamic int8 量化 | +50–100% | 品質可能再降 |

**最後輸出 benchmark 報告**（這份報告貼在 README，超有殺傷力）：
```
Model size: 234 MB → 62 MB
p95 latency: 18.3s → 2.1s (8.7x faster)
SSIM vs baseline: 1.000 → 0.982 (perceptually identical)
```

### 實作 Deliverables
- [ ] 完整 async 架構運作
- [ ] 架構圖（excalidraw）
- [ ] benchmark.py + 結果報告
- [ ] DB 紀錄每次推論的 model_version 跟 inference_ms（為下週監控做準備）

### 三方向 Talking Point 種子
- **MLE**: 「我把 inference 從 18 秒降到 2 秒，主要靠 step reduction + ONNX + int8 量化，過程中用 SSIM 監控品質沒掉」
- **Backend**: 「我用 Celery + Redis 做 async task queue，配合 DB status field 做 idempotent retry」
- **Full-stack**: 「前端用 polling + optimistic UI 處理長時間任務，generation 卡片有清楚的 loading/done/failed 狀態」

### 卡關 Fallback
- ONNX export 死活不過 → 跳過，做完 step reduction + torch.compile 已經很夠用
- Celery 一直連不上 Redis → 用 RQ（更簡單）替代，或先用 FastAPI BackgroundTasks（簡陋但能跑）

---

## Week 5：AWS 部署上雲

### 目標
有一個 `https://your-domain.com` 朋友打開就能用的公開系統。

### 設定 AWS Billing Alert（**第一件事**）
- [ ] AWS Console → Billing → 設 $20 USD 警報
- [ ] 設 $50 USD 強制警報

### 學習任務
- [ ] AWS 核心概念：IAM users/roles、VPC（不深入，知道是什麼就好）
- [ ] ECR：build → tag → push image
- [ ] ECS Fargate：task definition、service、task role
- [ ] RDS：建 PostgreSQL，記得設 publicly accessible = false
- [ ] ElastiCache：Redis 託管版
- [ ] S3 + CloudFront：bucket policy、CORS、CDN 設定
- [ ] Route 53：買網域 + DNS 設定（買 `.dev` 或 `.app` 大約 NT$300/年）

### AWS 服務拓撲
```
Internet
   │
   ├──► Vercel (Next.js)
   │       │
   │       ▼
   ├──► CloudFront ──► API ALB ──► ECS Fargate (FastAPI)
   │                                     │
   │                                     ├──► RDS PostgreSQL
   │                                     ├──► ElastiCache Redis
   │                                     └──► S3 (presigned URLs)
   │
   └──► CloudFront ──► S3 (生成圖片)

ECS Fargate (Celery Worker) — 跟 API 同 VPC，連同樣的 Redis 跟 RDS
```

### 實作 Deliverables
- [ ] FastAPI image 推到 ECR
- [ ] Celery worker image 推到 ECR
- [ ] ECS Cluster + 兩個 service（api、worker）跑起來
- [ ] RDS 連得上、migration 跑過
- [ ] S3 bucket 設好 CORS、CloudFront distribution
- [ ] Vercel 部署 Next.js，環境變數指向 AWS API
- [ ] 自訂網域生效

### 三方向 Talking Point 種子
- **MLOps**: 「整套架構 ECS Fargate + RDS + ElastiCache，用 Vercel + CloudFront 加速前端與圖片 CDN」
- **Backend**: 「我特別把 RDS 放在 private subnet 不對外開放，只透過 ECS task role 連線，符合 least privilege 原則」
- **Full-stack**: 「前端走 Vercel 邊緣節點，靜態資源毫秒級回應」

### 成本控制（CPU 推論版本）
| 服務 | 規格 | 月成本（約） |
|---|---|---|
| ECS Fargate API | 0.5 vCPU, 1GB | $5 |
| ECS Fargate Worker | 1 vCPU, 2GB | $10 |
| RDS db.t4g.micro | — | $13（free tier 1 年免費） |
| ElastiCache cache.t4g.micro | — | $13 |
| S3 + CloudFront | 低流量 | $1 |
| Route 53 | hosted zone | $0.5 |
| **總計** | | **~$15–40/月** |

> 不 demo 時把 ECS service desired count 設 0，月成本能再砍一半以上。

### 卡關 Fallback
- AWS 卡超過 3 天 → **直接用 [Render.com](https://render.com)**（5 分鐘部署，免費 tier 就能跑），AWS 的部分變成 README 裡的「Future Migration Plan」+ 你的架構圖。面試一樣能講「I designed an AWS-ready architecture, currently deployed on Render for cost reasons, with migration plan documented」。
- RDS 連不上 → 99% 是 security group 設定，檢查 inbound rule from ECS SG

---

## Week 6：CI/CD + Auth + 監控

### 目標
程式 push 到 main 自動部署、錯誤會被 capture、有 user 系統。

### 學習任務
- [ ] GitHub Actions：workflow syntax、secrets 管理、matrix builds
- [ ] NextAuth.js + Google OAuth provider
- [ ] Sentry：前後端 SDK
- [ ] CloudWatch Logs Insights 基本查詢

### 實作 Deliverables

**CI/CD pipeline**（`.github/workflows/`）：
- [ ] `ci.yml`：PR 觸發 → lint + test（前後端都要）
- [ ] `deploy-api.yml`：main 推送 → build → push ECR → 更新 ECS service
- [ ] `deploy-worker.yml`：類似 api
- [ ] Vercel 自己會處理前端 CI/CD

**測試**：
- [ ] 後端 pytest：API endpoints happy path + 一個 error case
- [ ] 前端 Playwright：填表單 → 送出 → 看到 generation 出現在歷史（一個 E2E 就夠）

**Auth**：
- [ ] NextAuth.js + Google provider 設好
- [ ] FastAPI middleware 驗 JWT
- [ ] Rate limit：未登入 IP 一天 5 次、登入 user 一天 50 次（用 Redis 計數）

**監控**：
- [ ] Sentry 接好前後端，故意 throw 一個 error 確認有收到
- [ ] CloudWatch Dashboard：API latency、error rate、Celery queue depth、推論時間 p95

### 三方向 Talking Point 種子
- **MLOps**: 「我有 dashboard 監控 inference latency p95 跟 queue depth，超過 threshold 會在 Sentry 警報」
- **Full-stack**: 「Auth 用 NextAuth.js + Google OAuth，session 用 JWT 跟後端共享」
- **Backend**: 「Rate limit 用 Redis sliding window，分 anonymous 跟 authenticated 兩個 tier」

### 卡關 Fallback
- GitHub Actions 跑不起來 → 先用手動 script，CI 留到第 7 週
- Auth 太燒腦 → 用 [Clerk](https://clerk.com) 託管 auth，5 分鐘搞定

---

## Week 7：系統打磨 + 文件

### 目標
專案到「拿出來給別人看不會丟臉」的程度，README 完整到讓面試官看完就想找你聊。

### 實作 Deliverables

**性能優化**（挑 2 個做）：
- [ ] DB index 優化，附 `EXPLAIN ANALYZE` 前後對比
- [ ] Redis cache：相同參數的請求 24 小時內直接回 cache
- [ ] 前端用 React.lazy 做 code splitting
- [ ] 圖片 lazy loading + Next.js Image 優化

**文件**（這部分超重要）：
- [ ] **README.md**：
  - 專案 hero image / GIF
  - Live demo URL
  - 一句話 elevator pitch
  - 架構圖（excalidraw 那張）
  - 技術棧 badges
  - Features 列表
  - 本地啟動步驟
  - 部署步驟（簡述）
  - Roadmap / Future work
- [ ] **ARCHITECTURE.md**：每個元件選擇的理由（這是面試準備材料）
- [ ] **ADR（Architecture Decision Records）**：3–5 個關鍵決策的記錄，例如：
  - 「為什麼選 Celery 而不是 SQS + Lambda」
  - 「為什麼選 ECS Fargate 而不是 EKS」
  - 「為什麼推論用 ONNX Runtime 而不是 TorchServe」
- [ ] **BENCHMARK.md**：第 4 週的優化結果完整版

**加分項**（時間夠才做）：
- [ ] OpenAPI / Swagger UI 對外公開
- [ ] Storybook 給前端 component
- [ ] 簡易 admin dashboard：今日生成數、平均 latency、錯誤率

### 三方向 Talking Point 種子
- 「我寫了 ADR 記錄關鍵架構決策，每個選擇都有 trade-off 分析」
- 此週主要是把前面所有東西的 talking point 整理成可講的故事

---

## Week 8：求職衝刺

### 目標
這個專案開始幫你拿到面試。

### 實作 Deliverables

**Demo 影片**（3 分鐘，超重要）：
- [ ] 30 秒：問題陳述 + 你解決什麼
- [ ] 90 秒：產品 demo（填參數 → 生成 → 看結果 → 看歷史）
- [ ] 60 秒：架構圖 + 三個技術亮點
- [ ] 上 YouTube unlisted，連結放 README + LinkedIn + 履歷

**技術文章**（中英各一篇）：
- [ ] 中文發 Medium / iThome：「我如何把 DDIM 模型部署到 production」
- [ ] 英文發 dev.to：「Reducing diffusion model inference latency by 8x with ONNX + quantization」
- [ ] 兩篇都連到 GitHub repo

**履歷改寫**：
- [ ] 專案放在最上面，三行描述（用 STAR + 數字）：
  ```
  Designed and deployed a diffusion-model-based photoelastic image generator.
  - Reduced p95 inference latency by 8.7x (18.3s → 2.1s) via DDIM step reduction,
    ONNX Runtime conversion, and int8 quantization, maintaining SSIM ≥ 0.98
  - Architected async serving system with FastAPI + Celery + Redis on AWS ECS,
    handling N concurrent generations with full observability via Sentry + CloudWatch
  - Built Next.js frontend with TypeScript, Tailwind, NextAuth Google OAuth,
    deployed to Vercel with E2E tests via Playwright
  ```
- [ ] 每個 bullet 包含**動詞 + 做法 + 結果（含數字）**

**LinkedIn 更新**：
- [ ] Headline 加上「Built ML serving systems on AWS」
- [ ] About 段落提到專案
- [ ] 一篇 post 介紹專案 + 連結

**投遞策略**（一週至少 30 家）：
- [ ] 104：搜「MLOps」「機器學習工程師」「全端工程師」
- [ ] LinkedIn：開啟 Open to Work（recruiters only），主動投「ML Engineer」「Full-stack Engineer」職缺
- [ ] 海外 remote：[Wellfound](https://wellfound.com)（前 AngelList）、[Remote OK](https://remoteok.com)
- [ ] 鎖定的台灣公司：Appier、iKala、玉山金 AI、聯發科 AI 部門、TSMC AI、Gogolook、KKBox、Dcard、17LIVE、Pinkoi
- [ ] 找內推：LinkedIn 搜目標公司，找你校友/前同事

---

## 四、每週時間分配建議

```
週一 18:00–22:00   學習新概念（4h）
週二 18:00–22:00   實作（4h）
週三 18:00–22:00   實作（4h）
週四 18:00–22:00   實作 + debug（4h）
週五 休息
週六 09:00–17:00   集中實作（8h）
週日 09:00–13:00   完成週末 deliverable + 寫週報自評（4h）
                                             總計：28h/週
```

> **每週日晚上自評**：3 個本週 talking point 各用 60 秒講出來錄音，講不順代表沒真的搞懂。

---

## 五、整體風險與緩解

| 風險 | 機率 | 影響 | 緩解策略 |
|---|---|---|---|
| AWS 部署卡關 | 高 | 高 | Week 5 卡 3 天就跳 Render.com |
| 前端框架學習曲線 | 中 | 中 | shadcn/ui + 抄模板，不重造輪子 |
| 模型優化品質掉太多 | 中 | 中 | 設 SSIM 門檻 0.95，不過就退回上一階段 |
| 推論還是太慢 | 低 | 中 | 退回更少步數 + 更小 batch；最差直接 mock |
| 個人時間不夠 | 中 | 高 | 每週砍 1–2 個非核心任務，保住主軸 |
| AWS 帳單失控 | 低 | 高 | Billing alert + 不 demo 時關掉 service |

---

## 六、成功指標（8 週後檢核）

打勾數越多，求職競爭力越強：

**技術面**：
- [ ] Public URL 可訪問且能跑通完整流程
- [ ] GitHub repo star 數 ≥ 5（朋友幫忙）、commit 數 ≥ 200
- [ ] README 完整含架構圖、benchmark、live demo 連結
- [ ] CI/CD pipeline 跑得起來
- [ ] 至少一篇技術文章發布

**求職面**：
- [ ] 履歷專案 section 含 3 個量化 bullet
- [ ] LinkedIn 個人檔案更新
- [ ] 投遞 ≥ 30 家
- [ ] 拿到 ≥ 5 個面試
- [ ] 至少 1 個 final round

---

## 七、長期心法（一週看一次）

1. **完成 > 完美**。Week 4 的 ONNX 優化做不出來不會死，但 Week 5 部署沒成功就沒東西可 demo。
2. **每個 commit 都假設未來的你會看**。寫好 commit message，這就是你工程素養的證據。
3. **遇到不會的先 timebox 1 小時自己查**，1 小時還沒進展就找 Claude / ChatGPT / Stack Overflow。
4. **進度落後時砍 scope 不是失敗**，把砍掉的部分寫進 Roadmap，面試時還能講「我有意識地做 trade-off」。
5. **這個專案是你的求職主菜**。八週內任何工作邀約如果讓你做不完這專案，思考清楚再答應。
