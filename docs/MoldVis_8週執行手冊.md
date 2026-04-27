# MoldVis · 8 週求職衝刺執行手冊

> 射出成型參數 → 光彈圖視覺化平台
> 從 DDIM research code 到 production-grade full-stack ML system

---

## 0. 專案戰略定位

### 一句話自我介紹（記起來，面試直接講）
> 「我把一個學術級的 DDIM diffusion model 工程化成 production system，使用者輸入射出成型製程參數，後端 async 處理推論並回傳光彈圖。技術棧是 Next.js + FastAPI + PostgreSQL + Celery + AWS ECS，並用 ONNX 量化把推論時間縮短 N%。」

### 三個職缺方向的對應賣點
| 方向 | 你的故事重點 |
|---|---|
| **ML / MLOps Engineer** | 模型 ONNX 化 + 量化 + Celery async + ECS 自動部署 + CloudWatch 監控 |
| **Backend Engineer** | PostgreSQL schema 設計 + Redis cache + rate limiting + presigned URL + ADR 文件 |
| **Full-stack Engineer** | Next.js + TypeScript + Tailwind + react-hook-form + NextAuth + Vercel 部署 |

### 關鍵原則
1. **每週日晚上錄一支 5 分鐘現況 demo 影片**，逼自己有產出（這也是 Week 8 拼接成正式 demo 的素材）
2. **commit message 寫好**，最後 GitHub history 本身就是履歷
3. **卡關超過 1 天**就走 fallback，不要硬撞——時間是最稀缺的資源
4. **每週固定一天「不寫 code 只整理筆記」**（建議週日），整理「我學到什麼、為什麼這樣做」——這就是 ADR（Architecture Decision Records）的雛形

### 預估投入時數
- 全職投入：每週 35-40 小時（每天 6-8 小時）
- 兼職投入：每週 20-25 小時（平日 2-3 小時 + 週末各 5-6 小時）
- 本手冊以**全職投入**為基準設計，兼職者請拉到 12 週

---

## Week 1 · JS/TS 急救包 + 後端骨架

**主題**：把全棧的「最小可運作版本」跑起來

### 學習任務
- [ ] JavaScript ES6+ 核心：解構、spread、async/await、Promise、模組（MDN 跑一輪）
- [ ] TypeScript 基礎：type vs interface、泛型基本用法、utility types（Pick, Omit, Partial）
- [ ] React 核心觀念：useState、useEffect、props、條件/列表渲染、controlled components
- [ ] 跑完 [react.dev 官方 tutorial](https://react.dev/learn)（井字遊戲那個）
- [ ] FastAPI quickstart + Pydantic v2 基礎
- [ ] Docker 基礎：Dockerfile、image vs container、port mapping、volume

### 實作任務
- [ ] 建立 monorepo 結構：`/frontend`（Next.js）、`/backend`（FastAPI）、`/ml`（你現有的模型程式碼）
- [ ] Next.js 14 App Router 專案初始化 + 基本頁面
- [ ] FastAPI 寫一個 `/health` 跟 `/echo` endpoint
- [ ] 寫 `Dockerfile.backend`，能本地用 `docker run` 起來
- [ ] 寫 `docker-compose.yml`（前端 + 後端）

### 週末 Deliverable
✅ 在瀏覽器打開 `localhost:3000`，按一個按鈕能呼叫到 `localhost:8000` 的 API 並顯示回傳結果，整套用 `docker-compose up` 啟動

### 卡關 Fallback
- TypeScript 學不下去 → 先用 JavaScript，TS 等 Week 3 再補
- Docker 卡住 → 直接本地跑（`npm run dev` + `uvicorn`），Docker 移到 Week 5 補
- React 概念混亂 → 看 [Net Ninja 的 React 系列](https://www.youtube.com/c/TheNetNinja)，視覺化學比看文件快

### 預估時數：32-38 小時

---

## Week 2 · PostgreSQL + SQL 紮根

**主題**：DB 是面試必考、業界必用，這週把根紮深

### 學習任務
- [ ] PostgreSQL 安裝（用 Docker 起 `postgres:16` image）
- [ ] SQL 核心：SELECT、JOIN（inner/left/right/full）、GROUP BY、HAVING、子查詢、CTE
- [ ] Index 觀念：B-tree、何時建 index、EXPLAIN ANALYZE 怎麼讀
- [ ] Transaction：ACID、isolation levels（讀懂 read committed vs repeatable read）
- [ ] 完成 [SQLZoo](https://sqlzoo.net) 全部關卡
- [ ] LeetCode SQL 30 題（Easy 20 + Medium 10）
- [ ] SQLAlchemy 2.0 ORM 基礎 + Alembic migration

### 實作任務
- [ ] 設計 schema 並用 Alembic 寫第一個 migration：
  ```sql
  users          (id, email, name, created_at, oauth_provider)
  generations    (id, user_id, params_jsonb, status, image_s3_key,
                  inference_ms, created_at, completed_at)
  parameter_presets (id, user_id, name, params_jsonb, is_public)
  api_usage      (id, user_id, endpoint, count, date)  -- for rate limiting
  ```
- [ ] `params_jsonb` 用 PostgreSQL JSONB 存射出參數（溫度、壓力、保壓時間等）
- [ ] 建好 index：`generations(user_id, created_at DESC)`、`generations(status)`、JSONB GIN index
- [ ] 寫完整 CRUD endpoints：建立生成請求、查詢歷史、刪除、匯出 preset

### 週末 Deliverable
✅ FastAPI 接 PostgreSQL 跑通完整 CRUD，能用 Postman / curl 操作所有 endpoints；至少跑一次 `EXPLAIN ANALYZE` 並截圖記錄查詢計畫

### 卡關 Fallback
- SQLAlchemy ORM 太複雜 → 改用 [Tortoise ORM](https://tortoise.github.io)（更直觀）或直接寫 raw SQL with asyncpg
- Alembic migration 卡住 → 先用 `Base.metadata.create_all()`，migration 移到 Week 7 補

### 面試金句準備
> 「我把參數用 JSONB 存而不是欄位攤平，因為射出參數會隨研究演進，JSONB + GIN index 給我 schema 彈性又不犧牲查詢性能。」

### 預估時數：35-40 小時

---

## Week 3 · 前端核心 + 模型首次接通

**主題**：完整的「填參數 → 送出 → 看結果」走通（local 同步版）

### 學習任務
- [ ] Next.js App Router：server vs client component、layout、loading.tsx、error.tsx
- [ ] Tailwind CSS（不要自己寫 CSS）
- [ ] [shadcn/ui](https://ui.shadcn.com) 安裝跟用法（直接複製貼上元件）
- [ ] react-hook-form + zod schema 驗證
- [ ] fetch / SWR / TanStack Query 二選一處理 client-side data fetching

### 實作任務
- [ ] 設計參數輸入 UI：滑桿、數字輸入、preset 選擇下拉選單
- [ ] 用 zod 寫參數 schema 並做前端驗證（射出溫度範圍、壓力範圍等）
- [ ] 結果顯示頁：歷史紀錄列表 + 點擊看大圖
- [ ] 把 DDIM 模型程式碼包成 FastAPI endpoint：`POST /api/generate` → 同步跑模型 → 回傳 base64 圖
- [ ] 前端送出表單 → 顯示 loading → 顯示生成的光彈圖

### 週末 Deliverable
✅ 本地完整 demo：填參數 → 送出 → 等 N 秒 → 看到生成圖；錄一支 3 分鐘 demo 影片給自己看

### 卡關 Fallback
- shadcn/ui 不會用 → 改用 [Mantine](https://mantine.dev)（開箱即用、不用自己接 Tailwind）
- 模型載入太慢拖慢開發 → 寫一個 `MOCK_MODEL=true` 模式，回傳預先存的範例圖

### 設計原則
這週**不要追求 UI 漂亮**，追求**功能完整**。漂亮留到 Week 7 打磨。

### 預估時數：35-40 小時

---

## Week 4 · Async 架構 + 模型量化（雙主題週，最關鍵的一週）

**主題**：把同步 blocking 架構改成 production-grade async + 第一次碰模型優化

> 這週是面試時被狂問的點，不要省。

### 學習任務 — Async 架構
- [ ] Celery 基本概念：broker、worker、task、result backend
- [ ] Redis 基本操作（順便當 broker + cache）
- [ ] HTTP polling vs Server-Sent Events vs WebSocket（為什麼選哪個）
- [ ] AWS S3：boto3 基本操作、bucket 政策、presigned URL 概念

### 學習任務 — 模型量化
- [ ] PyTorch dynamic quantization 基礎（[官方 tutorial](https://pytorch.org/tutorials/recipes/recipes/dynamic_quantization.html)）
- [ ] DDIM 推論步數壓縮：50 → 25 → 10 步的品質 vs 速度 trade-off
- [ ] 簡單 benchmark：用 `time.perf_counter()` 量推論延遲

### 實作任務 — Async 架構
- [ ] 改寫 `/api/generate`：收到請求 → 建立 DB record (status=pending) → Celery enqueue → 立即回 `{task_id, status_url}`
- [ ] Celery worker：跑模型 → 上傳 S3 → 更新 DB record (status=completed, image_s3_key=...)
- [ ] 新增 `/api/generations/{id}` endpoint：前端 polling 用
- [ ] 前端改成 polling 模式（每 2 秒查一次直到 completed）
- [ ] presigned URL：前端直接從 S3 拿圖，不經過你的 server

### 實作任務 — 模型量化
- [ ] 跑 baseline benchmark：i5-14500k 上 50 步推論的 p50/p95 延遲
- [ ] DDIM 步數實驗：50 / 25 / 10 步分別跑 100 次取平均，記錄品質感官評估
- [ ] 套用 PyTorch dynamic quantization 到模型的 linear layer，再 benchmark
- [ ] 把實驗數據寫成 markdown 表格，放進 repo 的 `experiments/` 資料夾

### 週末 Deliverable
✅ 整套 async pipeline 跑通：開兩個瀏覽器同時送請求不會互卡
✅ 一張清楚的架構圖（用 [excalidraw](https://excalidraw.com) 畫，存成 PNG 放進 README）
✅ 量化前後的 benchmark 表格（這是面試金句的彈藥）

### 卡關 Fallback
- Celery 太複雜 → 用 [RQ (Redis Queue)](https://python-rq.org)（更簡單，原理一樣）
- S3 設定卡住 → 暫時用 local volume + nginx 提供圖檔，S3 移到 Week 5 一起做
- 量化跑出來模型壞掉 → 跳過量化，改做「DDIM 步數壓縮」實驗就好（10 步通常還能用）

### 面試金句準備
> 「同步 HTTP 跑 ML 推論在 production 是不可接受的——一個請求佔用 worker 數秒，併發馬上死。我用 Celery + Redis 把推論非同步化，HTTP 端 < 50ms 回應，圖片透過 S3 presigned URL 直送前端，繞過 backend bandwidth 瓶頸。」
>
> 「我把 DDIM 從 50 步壓到 N 步，加上 PyTorch dynamic quantization，CPU 推論延遲從 X 秒降到 Y 秒，品質感官上沒有顯著差異。」

### 預估時數：40-45 小時（這週可能會超時，沒關係）

---

## Week 5 · AWS 上雲（會痛但值得）

**主題**：把整套東西部署到公開可訪問的 URL

> ⚠️ 開工前先設好 AWS Billing Alert：$10、$30、$50 三個門檻發 email

### 學習任務
- [ ] AWS 帳號開好、開啟 MFA（不要用 root account 做事，建一個 IAM user）
- [ ] 看完 [AWS Cloud Practitioner Essentials](https://aws.amazon.com/training/digital/aws-cloud-practitioner-essentials/)（免費，6 小時）
- [ ] VPC、subnet、security group 概念（不用深入，知道是什麼就好）
- [ ] ECR、ECS Fargate、RDS、S3、CloudFront、Route 53 各自做什麼

### 實作任務
- [ ] ECR：建 repo，把 backend image push 上去
- [ ] RDS：開最便宜的 `db.t4g.micro` PostgreSQL（注意 free tier 12 個月）
- [ ] ElastiCache Redis 或自架 Redis on Fargate（Redis on Fargate 較便宜）
- [ ] S3 bucket：private + presigned URL 模式
- [ ] ECS Fargate：兩個 service（API + Celery worker）
- [ ] Application Load Balancer 接 API
- [ ] Vercel 部署 Next.js（**不要在 AWS 部署前端**，Vercel 免費又快）
- [ ] （可選）買網域 + Route 53 + ACM 憑證做 HTTPS

### 週末 Deliverable
✅ 一個公開 URL（例如 `moldvis.your-domain.dev`），朋友打開就能用
✅ 用手機打開測試，從生成到看到圖整個 < 30 秒

### 卡關 Fallback
- AWS ECS 卡關超過 3 天 → 改用 [Render.com](https://render.com)（5 分鐘部署，每月 $7）或 [Railway](https://railway.app)；README 寫「Future migration plan to AWS ECS」並附架構圖。**面試時 AWS 知識照樣能講，不影響故事。**
- RDS 太貴 → 用 [Neon](https://neon.tech) 或 [Supabase](https://supabase.com) 的免費 PostgreSQL
- 模型 image 太大（PyTorch + checkpoint 可能 > 2GB）→ 用 multi-stage build，model checkpoint 放 S3 啟動時下載

### 成本估算（CPU 推論）
| 項目 | 月費 |
|---|---|
| ECS Fargate（API 0.25 vCPU + Worker 0.5 vCPU 常駐） | $15-20 |
| RDS `db.t4g.micro`（free tier 後） | $13 |
| Redis on Fargate（0.25 vCPU） | $7 |
| S3 + CloudFront（小流量） | $1 以內 |
| Route 53 + 網域 | $1 + 網域年費 |
| **合計** | **約 $35-45/月** |

> 省錢技巧：demo 時才開 Worker service，平時 desired count = 0；或用 EventBridge 排程開關。

### 預估時數：35-45 小時

---

## Week 6 · CI/CD + 監控 + Auth

**主題**：補齊 production 該有的東西

### 學習任務
- [ ] GitHub Actions 基礎：workflow、job、step、secrets
- [ ] pytest 基礎 + FastAPI test client
- [ ] CloudWatch Logs 基礎
- [ ] [Sentry](https://sentry.io) 接前後端（免費 plan 夠用）
- [ ] OAuth 2.0 流程概念（不用刻密碼系統）
- [ ] [NextAuth.js](https://authjs.dev) v5 基礎

### 實作任務
- [ ] GitHub Actions pipeline：
  1. push 到 main → 跑 pytest + ruff lint + mypy
  2. 通過 → build & push image 到 ECR
  3. 觸發 ECS service update
- [ ] 寫 8-12 個 pytest（不追求 coverage，重點是 happy path + auth + 一個 Celery task test）
- [ ] 寫 2 個 [Playwright](https://playwright.dev) E2E（登入流程 + 完整生成流程）
- [ ] CloudWatch Logs 集中所有 service log + 一個簡單的 metric dashboard
- [ ] Sentry：前端後端都接，故意丟一個 exception 看到 alert
- [ ] NextAuth + Google OAuth：登入後才能用、未登入轉導
- [ ] 生成記錄綁 user_id（之前是空的）

### 週末 Deliverable
✅ Push 一個 commit 到 main，10 分鐘後 production 自動部署完成
✅ 故意打壞 API 看 Sentry 收到通知
✅ Google 登入流程完整可用

### 卡關 Fallback
- ECS rolling update 設定複雜 → 先做 manual deploy（Actions 只跑 build & push，部署用手動 CLI），完整 CD 改 Week 7 做
- NextAuth.js v5 在 beta 不穩 → 用 v4 stable 版本

### 面試金句準備
> 「我用 GitHub Actions 做 CI/CD，每個 PR 自動跑 lint + test，merge 到 main 自動部署到 ECS rolling update。Sentry + CloudWatch 在 staging 階段就抓到 N 個 bug。」

### 預估時數：35-40 小時

---

## Week 7 · 性能優化 + 文件 + ONNX Stretch Goal

**主題**：把專案打磨到面試時拿得出手

### 學習任務
- [ ] PostgreSQL 性能：`EXPLAIN ANALYZE` 進階、index scan vs seq scan
- [ ] Redis cache pattern：cache-aside、TTL 策略
- [ ] Rate limiting 演算法：token bucket vs sliding window
- [ ] （Stretch）ONNX 概念 + ONNX Runtime（[官方教學](https://onnxruntime.ai/docs/tutorials/)）

### 實作任務
- [ ] 用 EXPLAIN ANALYZE 找出最慢的 query，加 index 或重寫，記錄優化前後對比
- [ ] Redis cache：相同參數的請求第二次直接從 cache 拿（hash 參數做 key），記錄 hit rate
- [ ] Rate limiting：每個 user 每小時最多 N 次生成（用 Redis INCR + EXPIRE）
- [ ] 寫 3 篇 ADR（Architecture Decision Records）：
  1. 「為什麼選 ECS Fargate 而不是 Kubernetes」
  2. 「為什麼用 Celery 而不是 SQS + Lambda」
  3. 「為什麼選 PostgreSQL JSONB 而不是 NoSQL」
- [ ] README 大改寫：架構圖、技術選型、API 文件（用 FastAPI 自動產生的 `/docs`）、本地啟動步驟、deployed URL、demo gif

### Stretch Goal（挑 1 個做就好）
- [ ] **ONNX 化模型**：PyTorch → ONNX → ONNX Runtime，benchmark 跟 PyTorch 比較
- [ ] **簡易 admin dashboard**：看每日生成數、平均延遲、user 排行
- [ ] **A/B test 框架**：50% 流量走 50 步模型、50% 走 25 步模型，記錄延遲跟 user feedback

### 週末 Deliverable
✅ README 寫到讓陌生人能 5 分鐘理解這專案在做什麼、技術棧、為什麼這樣設計
✅ 至少 1 個 ADR 完成
✅ Stretch goal 至少完成 1 個

### 卡關 Fallback
- ONNX 轉換出問題 → 跳過，PyTorch 直接量化版本就夠了
- 寫 ADR 卡關 → 用模板：背景 → 選項 → 決策 → 後果

### 面試金句準備
> 「我發現 generations 表的 user 歷史查詢慢，EXPLAIN ANALYZE 看到是 seq scan，加了 `(user_id, created_at DESC)` composite index 後從 N ms 降到 M ms。」
> （Stretch）「我把模型用 ONNX Runtime 跑，CPU 推論比 PyTorch 快 X%，因為 ONNX Runtime 對 CPU 有 graph-level 優化。」

### 預估時數：30-35 小時

---

## Week 8 · 求職衝刺

**主題**：把專案變成 offer

### 求職資料準備
- [ ] **Demo 影片**：3 分鐘錄好，YouTube unlisted，放進 README 跟履歷
  - 開場 15 秒講問題（射出成型工程師的痛點）
  - 30 秒 demo 流程（填參數 → 看圖）
  - 90 秒講架構（秀架構圖、講 async + 量化）
  - 30 秒講你的 trade-off（為什麼這樣設計）
  - 15 秒收尾 + repo URL
- [ ] **GitHub README** 最終版
- [ ] **技術部落格文章**（中英雙寫，至少 1 篇）：
  - 中文發 [Medium](https://medium.com) 或 [iThome 鐵人賽](https://ithelp.ithome.com.tw) blog
  - 英文發 [dev.to](https://dev.to)
  - 主題建議：「How I deployed a diffusion model to production with FastAPI + Celery + AWS ECS」
- [ ] **LinkedIn post**：中英雙語，配 demo gif，標 #MLOps #FullStack #DiffusionModels
- [ ] **履歷改寫**：MoldVis 放最上面，3-4 條 bullet，每條都有**數字**：
  - ✅ 「Reduced inference latency from 8s to 2.3s (71%) via ONNX Runtime + DDIM step compression」
  - ✅ 「Designed async architecture supporting N concurrent generations with M ms p95 API latency」
  - ✅ 「Built CI/CD pipeline (GitHub Actions → ECR → ECS) reducing deploy time from manual 30min to automated 4min」
  - ❌ 「Worked on a full-stack ML application」← 這種廢話不要寫

### 投履歷策略
- [ ] **台灣**：104 + LinkedIn 同步投，目標公司：
  - 半導體 AI（TSMC、聯發科 AI 部門、聯詠）
  - AI 新創（Appier、iKala、Vpon、玩美移動、Profet AI、緯創 AI）
  - 大型科技（91APP、KKBOX、KKday、Dcard、SHOPLINE）
- [ ] **海外遠端**：[WeWorkRemotely](https://weworkremotely.com)、[RemoteOK](https://remoteok.com)、[LinkedIn remote filter]
- [ ] **目標數量**：第 8 週投 30 家以上，預期收到回覆 5-8 家、面試 3-5 家、offer 1-2 家

### 面試準備
- [ ] 準備 5 個高頻問題的回答（用 STAR 格式，配合 MoldVis 故事）：
  1. 「介紹一下你最有挑戰性的專案」
  2. 「你怎麼設計這個系統的架構？為什麼這樣選？」
  3. 「遇到過最難解的 bug 是什麼？」
  4. 「你怎麼優化性能？」
  5. 「如果重做一次，你會怎麼改？」
- [ ] 系統設計題練習：把 MoldVis 當案例練「scale to 1M users 怎麼改」
- [ ] LeetCode：每天 1-2 題保持手感（不要本末倒置）
- [ ] Mock interview：找朋友或用 [Pramp](https://pramp.com) 練 2-3 場

### 週末 Deliverable
✅ Demo 影片上線
✅ 至少 1 篇技術文章發布
✅ 履歷改寫完成且投出 30 家以上
✅ 至少 2 場 mock interview

### 預估時數：30 小時（其中 15 小時投履歷 + 準備面試，15 小時持續優化專案）

---

## 工具與資源總覽

### 開發工具
| 工具 | 用途 | 替代方案 |
|---|---|---|
| VSCode + Cursor | IDE | JetBrains, Zed |
| pnpm | Node.js 套件管理 | npm, yarn |
| uv 或 poetry | Python 套件管理 | pip + venv |
| TablePlus 或 DBeaver | DB GUI | psql CLI |
| Postman 或 Bruno | API 測試 | curl, httpie |
| excalidraw | 架構圖 | draw.io, Mermaid |

### 學習資源（精選，不要再到處找）
- **JavaScript/TypeScript**：[javascript.info](https://javascript.info)、[Total TypeScript](https://www.totaltypescript.com)（基礎免費）
- **React/Next.js**：[react.dev](https://react.dev)、[Next.js Learn](https://nextjs.org/learn)
- **FastAPI**：[官方文件](https://fastapi.tiangolo.com)（當教科書讀）
- **PostgreSQL**：[Use The Index, Luke!](https://use-the-index-luke.com) + 《SQL 必知必會》
- **AWS**：[AWS Skill Builder](https://skillbuilder.aws) + [AWS in Plain English](https://aws.plainenglish.io) Medium
- **MLOps**：Chip Huyen《Designing Machine Learning Systems》第 7-9 章
- **系統設計**：[ByteByteGo Newsletter](https://blog.bytebytego.com)（免費部分夠看）

### AI 助手用法
- **Claude / ChatGPT**：寫 code、Debug、技術選型討論、code review
- **Cursor**：IDE 整合 AI，refactor 跟生成 boilerplate 神快
- **不要做的**：不要直接複製貼上不理解的 code。AI 寫一段，你逐行讀一段，看不懂就問為什麼。

---

## 風險與應急

### 高風險項目（事先預防）
1. **AWS 帳單失控** → Week 5 第一天就設 Billing Alert，每週查一次
2. **Week 4-5 卡關**（最容易卡的兩週）→ 嚴格執行「卡 1 天就走 fallback」
3. **完美主義拖延** → 每週日強制 deliverable，做不完就 carry over，不要無限延期
4. **學習與實作失衡** → 60% 時間實作、40% 時間學習，反過來就是浪費

### 進度落後 N 週的應急方案
- **落後 1 週**：壓縮 Week 7 的 stretch goal，照計畫走
- **落後 2 週**：跳過 Week 7，Week 8 只做求職資料 + 投履歷
- **落後 3 週以上**：拉長到 12 週、改兼職節奏，或砍掉 quantization + ONNX，專注全棧故事

---

## 最後，給自己的提醒

> 這份手冊的目標**不是把每件事都學會**，是讓你在 8 週後**可以對著一個 deployed URL 講出一個完整的故事**：你解決了什麼問題、為什麼這樣設計、trade-off 是什麼、學到什麼。
>
> 面試官不會問你 React useEffect 的 dependency 細節，他會問「為什麼選 React」、「如果重做你會怎麼改」、「scale 10 倍會壞在哪」。
>
> **故事 > 技術細節 > LeetCode**。MoldVis 就是你的故事。

---

✅ 開工前 checklist：
- [ ] 讀完整份手冊一遍
- [ ] 設好 AWS Billing Alert（即使這週不會用到）
- [ ] 開好 GitHub repo `moldvis`，Push 一個 README 寫上「Day 1」
- [ ] 在行事曆上排好 8 週的每個週日 deliverable check 時段
- [ ] 找 1-2 個朋友當「進度問責夥伴」，每週日傳 demo 影片給他們

開始吧。
