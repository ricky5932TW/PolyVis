# MoldVis · 每週作戰板（KPI + 技術重點 + 面試考點）

> 配套手冊使用。每週日花 30 分鐘對著這份打勾、打 X，落後就觸發 fallback。

## 怎麼用這份文件

- **KPI 區**：週末自我檢核，達標率 < 70% 就要警覺、< 50% 觸發 fallback
- **技術重點**：每個觀念你都要能用 1-2 分鐘**講出來**（不是看懂，是講出來），對著鏡子練或錄音
- **面試考點**：這週的內容**真的會被問**，括號內是面試官想聽到的關鍵字

---

## Week 1 · JS/TS 急救包 + 後端骨架

### 量化 KPI
- [ ] 不看文件寫出一個有 `useState` + `useEffect` 的 React component（fetch API 並顯示資料）
- [ ] 寫出一個 TypeScript function 用到泛型（例如 `function getFirst<T>(arr: T[]): T | undefined`）
- [ ] FastAPI 寫出 3 個 endpoint（GET、POST with Pydantic body、path parameter）
- [ ] `docker-compose up` 一行起前後端，互通成功
- [ ] GitHub commit 數 ≥ 15（小步提交，message 要清楚）
- [ ] 解釋什麼是 CORS 並能在 FastAPI 設定它

### 技術重點（要能講出來）
1. **JavaScript event loop 與 async/await 機制**：為什麼 JS 是 single-threaded 卻能處理併發？async/await 跟 Promise 的關係？
2. **React 的 reactivity 模型**：state 變更如何觸發 re-render？為什麼不能直接 mutate state？
3. **TypeScript 為什麼有用**：除了「有 type 提示」之外，重構安全、API 契約、泛型重用的價值
4. **Docker 的本質**：為什麼是 image vs container？為什麼解決「在我電腦上可以跑」？跟 VM 差在哪？

### 面試考點
- 🎯 「**JavaScript 的 event loop 是什麼？**」（call stack、task queue、microtask queue、setTimeout(0) vs Promise.resolve）
- 🎯 「**`let` `const` `var` 差在哪？**」（block scope、TDZ、hoisting、為什麼現代不用 var）
- 🎯 「**箭頭函式跟一般 function 的差別？**」（`this` 綁定、不能當 constructor、沒有 `arguments`）
- 🎯 「**Docker 跟 VM 差在哪？**」（共享 kernel、輕量、啟動速度、隔離程度）
- 🎯 「**為什麼用 TypeScript？**」（型別安全、IDE 支援、重構、契約、文件性）

### 紅線
本週**不碰 Redux、不碰 Next.js 進階**。基本功重要，貪多嚼不爛。

---

## Week 2 · PostgreSQL + SQL 紮根

### 量化 KPI
- [ ] LeetCode SQL 完成 Easy 20 + Medium 10（總共 30 題）
- [ ] SQLZoo 全部關卡綠勾
- [ ] 不看文件寫出 3-table JOIN + GROUP BY + HAVING 的查詢
- [ ] 為 generations 表設計 ≥ 2 個 index 並用 `EXPLAIN ANALYZE` 證明效果（截圖存檔）
- [ ] Alembic 寫出至少 3 個 migration（包含一次 schema 變更，例如加欄位）
- [ ] 用 SQLAlchemy 完成 generations 的 CRUD（含 soft delete）

### 技術重點（要能講出來）
1. **Index 為什麼變快、什麼時候不該加 index**：B-tree 結構、寫入成本、low cardinality 欄位的問題
2. **JOIN 的執行原理**：nested loop / hash join / merge join，PostgreSQL 怎麼選
3. **Transaction 的 ACID 與 isolation level**：dirty read、non-repeatable read、phantom read 各是什麼、PostgreSQL 預設哪個 level
4. **Normalization vs Denormalization**：什麼時候該 normalize 到 3NF、什麼時候該為了 read 性能 denormalize
5. **JSONB vs columnar storage**：為什麼 MoldVis 用 JSONB 存參數、什麼情境會後悔

### 面試考點
- 🎯 「**怎麼優化一個慢 query？**」（先 EXPLAIN ANALYZE、看 seq scan、加 index、考慮 covering index、最後考慮 schema 重設計）
- 🎯 「**INNER JOIN vs LEFT JOIN vs OUTER JOIN 差在哪？什麼時候用 LEFT？**」
- 🎯 「**`COUNT(*)` 跟 `COUNT(column)` 差在哪？**」（NULL 值處理）
- 🎯 「**為什麼一張表加 index 之後寫入會變慢？**」（每次 insert/update 都要更新 index）
- 🎯 「**Transaction isolation level 各是什麼？**」（4 種 level + 對應防止的問題）
- 🎯 「**N+1 query 是什麼？怎麼解？**」（ORM 常見陷阱、用 JOIN 或 batch loading）
- 🎯 「**為什麼選 PostgreSQL 而不是 MySQL？**」（JSONB、CTE、窗口函數、PostGIS、type 系統豐富）
- 🎯 「**ACID 是什麼？舉例。**」（Atomicity 轉帳、Consistency 約束、Isolation 並發、Durability 斷電）
- 🎯 系統設計常考：「**設計一個 URL 短網址系統的 schema**」、「**設計 Twitter 的 timeline schema**」

### 紅線
本週**不要碰 Redis、不要碰 NoSQL**。Postgres 不熟就先死磕 Postgres。

---

## Week 3 · 前端核心 + 模型首次接通

### 量化 KPI
- [ ] 用 react-hook-form + zod 寫出至少 6 個欄位的表單，含驗證錯誤顯示
- [ ] 用 shadcn/ui 至少 5 個元件（Button, Input, Select, Card, Dialog）
- [ ] Tailwind 寫的版面在桌機/平板/手機都不爆版（用 DevTools 切換確認）
- [ ] 完成「填參數 → 送出 → 看到生成圖」端到端 flow（同步版）
- [ ] 處理 3 種 UI 狀態：loading / error / empty state
- [ ] 第一支 demo 影片（5 分鐘自錄就好），確認流程順暢

### 技術重點（要能講出來）
1. **Server Component vs Client Component**：Next.js App Router 的核心觀念、什麼時候用 `"use client"`、為什麼這樣設計（bundle size、SEO、初始載入）
2. **Controlled vs Uncontrolled component**：為什麼 react-hook-form 選 uncontrolled（性能）
3. **Form 驗證的層級**：client-side（UX）、server-side（security）、DB constraint（data integrity）——三層都要有
4. **CSS 的盒模型 + Flexbox + Grid**：Tailwind 是 utility，不會 CSS 也是不會
5. **Hydration 是什麼**：SSR 後 client 接管的過程、hydration mismatch 為什麼會發生

### 面試考點
- 🎯 「**React 的 useEffect 什麼時候 trigger？dependency array 空的跟沒寫差在哪？**」（mount、update、unmount、空陣列只跑一次、沒寫每次都跑）
- 🎯 「**為什麼不能在 if 裡面用 hook？**」（hook 順序依賴、Linked List 內部實作）
- 🎯 「**useMemo / useCallback 什麼時候真的需要？**」（不是無腦加，profiling 後再加）
- 🎯 「**React key prop 為什麼重要？用 index 當 key 為什麼有 bug？**」（reconciliation、list 重排序問題）
- 🎯 「**Next.js 為什麼比純 React 快？**」（SSR、SSG、ISR、image optimization、自動 code splitting）
- 🎯 「**SSR vs CSR vs SSG 差在哪？**」（首屏速度、SEO、互動性、適用場景）
- 🎯 「**form 驗證為什麼要前後端都做？**」（前端 UX、後端 security——前端可以被繞過）
- 🎯 「**為什麼前端送出表單後不直接顯示成功，而是等 server 回傳？**」（optimistic update vs pessimistic update 的 trade-off）

### 紅線
本週**不要追求 UI 漂亮**，先求功能完整。Polish 留到 Week 7。

---

## Week 4 · Async 架構 + 模型量化（最重要的一週）

### 量化 KPI
- [ ] 開兩個瀏覽器同時送 5 個請求，所有請求都 < 100ms 拿到 task_id（不會互卡）
- [ ] Celery worker 能水平擴展：跑 1 個 vs 跑 3 個 worker，throughput 線性成長
- [ ] S3 presigned URL 設定 5 分鐘過期，前端能拿到圖
- [ ] 量化前後 benchmark 表格：DDIM 50 步 vs 25 步 vs 10 步、量化前 vs 後，latency 跟主觀品質都記錄
- [ ] 一張完整架構圖（excalidraw），含所有元件 + 資料流向 + 故障點標記
- [ ] 第一個面試金句完整能講：「為什麼這個系統不能用同步 HTTP」

### 技術重點（要能講出來）
1. **同步 vs 非同步的本質差別**：thread blocking、HTTP timeout、worker exhaustion、為什麼 ML inference 必須非同步
2. **Message queue 的角色**：解耦、削峰、retry、durability——為什麼不直接 producer 呼 consumer
3. **Polling vs SSE vs WebSocket**：各自的網路成本、複雜度、適用場景
4. **Presigned URL 為什麼存在**：bandwidth 卸載、安全（不暴露 S3 key）、過期控制
5. **Quantization 的本質**：FP32 → INT8、精度損失 vs 速度收益、dynamic vs static quantization 差別
6. **DDIM 步數壓縮的原理**：DDIM 不是 Markov 過程、可以跳步、為什麼比 DDPM 適合少步推論

### 面試考點（這週的考點面試**一定會被問**）
- 🎯 「**你怎麼處理長時間的 ML 推論請求？**」（task queue、polling、SSE、WebSocket、idempotency key）
- 🎯 「**如果 worker 跑到一半掛掉怎麼辦？**」（task retry、idempotency、dead letter queue、visibility timeout）
- 🎯 「**Celery 的 task result 你存哪？為什麼？**」（Redis 還是 DB、TTL、為什麼不直接 in-memory）
- 🎯 「**為什麼用 S3 而不是直接從 server 回傳圖？**」（bandwidth、CDN、解耦儲存與運算、scalability）
- 🎯 「**Presigned URL 安全嗎？怎麼避免被濫用？**」（短 TTL、IP 限制、access logs、單次使用 token）
- 🎯 「**怎麼避免一個 user 把整個系統打掛？**」（rate limiting、queue priority、dedicated worker pool、circuit breaker）
- 🎯 「**Quantization 為什麼能加速？精度損失多少？**」（INT8 計算、SIMD、cache locality、視任務通常 < 1% accuracy drop）
- 🎯 「**100 個併發請求進來怎麼辦？**」（queue 緩衝、auto-scaling worker、back-pressure、429 rate limit、優雅降級）
- 🎯 系統設計常考：「**設計一個圖片上傳處理服務（壓縮、生成縮圖）**」——架構幾乎跟 MoldVis 一樣，你親手做過

### 紅線
本週是計畫的命脈。**寧可超時 3 天也要做完**。如果真的卡關超過 3 天才換 RQ 取代 Celery，但不要砍掉 async 改造這件事本身。

---

## Week 5 · AWS 上雲

### 量化 KPI
- [ ] 公開 URL 可訪問，HTTPS 綠鎖
- [ ] 從手機打開 → 完整生成流程 < 30 秒
- [ ] AWS 帳單 alert 三層都設好（$10、$30、$50）
- [ ] 能畫出整個 AWS 架構圖含 VPC、subnet、SG、IAM role
- [ ] CloudWatch 能看到 ECS service log、Celery worker log
- [ ] 把 deploy 步驟寫成 runbook（即使現在還是手動）

### 技術重點（要能講出來）
1. **VPC、subnet、security group 的職責劃分**：network-level 隔離、為什麼 public subnet 放 ALB、private subnet 放 RDS
2. **IAM 的 least privilege 原則**：role vs user vs policy、ECS task role 為什麼比 long-lived access key 安全
3. **ECS Fargate vs EC2 vs Lambda**：什麼時候用哪個（trade-off：成本、冷啟動、運維、極限）
4. **Load Balancer 的角色**：health check、SSL termination、blue/green deploy
5. **Multi-stage Docker build 為什麼重要**：image size、安全（不帶 build tools）、deploy 速度

### 面試考點
- 🎯 「**為什麼選 ECS Fargate 而不是 K8s？**」（運維成本、團隊規模、EKS 的學習曲線、Fargate 沒 node 管理——這題可能被追問你知不知道 K8s）
- 🎯 「**ECS Service 跟 Task 差在哪？**」（Service 維持 desired count、Task 是執行單位、Task Definition 是模板）
- 🎯 「**怎麼讓你的 ECS service 不中斷地更新版本？**」（rolling update、deployment circuit breaker、health check grace period）
- 🎯 「**RDS 為什麼要放 private subnet？**」（DB 不該被公網直接訪問、defense in depth）
- 🎯 「**S3 bucket 怎麼設定才安全？**」（block public access、bucket policy、IAM、presigned URL、access log）
- 🎯 「**怎麼控制雲端成本？**」（reserved instance、spot、auto-scaling、scale to zero、CloudWatch alarm、cost explorer）
- 🎯 「**為什麼用 CloudFront？**」（edge caching、SSL、地理分佈、DDoS 緩解）
- 🎯 「**Stateless 跟 Stateful service 差在哪？為什麼 stateless 好 scale？**」（horizontal scale、無 session affinity、failure recovery）

### 紅線
卡關超過 3 天 → 退到 Render.com，不要硬撞。**這是 PM 給你的命令，不是建議**。

---

## Week 6 · CI/CD + 監控 + Auth

### 量化 KPI
- [ ] Push 到 main → 10 分鐘內 production 自動更新
- [ ] CI 的 lint + test 全綠才能 merge（GitHub branch protection）
- [ ] pytest 覆蓋核心 happy path（10 個 test 上下）
- [ ] Playwright E2E 至少 2 個（登入 + 完整生成）
- [ ] 故意丟 exception 30 秒內 Sentry 收到 alert
- [ ] Google OAuth 登入流程順暢，未登入頁面正確 redirect

### 技術重點（要能講出來）
1. **CI 跟 CD 差在哪**：Continuous Integration（merge 前驗證）vs Continuous Delivery（自動部署到 staging）vs Continuous Deployment（自動到 prod）
2. **Test 金字塔**：unit (多) → integration → E2E (少)，為什麼這個比例
3. **OAuth 2.0 的 flow**：authorization code flow、為什麼比 implicit flow 安全、PKCE 的角色
4. **JWT vs Session**：stateless vs stateful、refresh token 的角色、什麼時候用哪個
5. **觀察性的三支柱**：logs、metrics、traces——各自解決什麼問題

### 面試考點
- 🎯 「**你的 CI/CD pipeline 長怎樣？**」（具體階段、平行 job、cache、secret 管理）
- 🎯 「**怎麼確保 main branch 不會被破壞？**」（branch protection、required check、required review）
- 🎯 「**unit test 跟 integration test 差在哪？**」（測試範圍、隔離程度、執行速度、價值）
- 🎯 「**怎麼測一個會打 DB 的 function？**」（mock、test container、in-memory DB、test fixture）
- 🎯 「**為什麼用 OAuth 而不是自己刻密碼系統？**」（password 儲存風險、密碼重設複雜度、user adoption、leverage 大廠安全投資）
- 🎯 「**JWT 的缺點是什麼？怎麼處理 logout？**」（無法主動 invalidate、token blacklist、短 TTL + refresh token）
- 🎯 「**Production 出問題你怎麼 debug？**」（log → metric → trace → reproduce → root cause → fix → postmortem）
- 🎯 「**怎麼設計 alert 才不會 alert fatigue？**」（actionable、SLO-based、降噪、on-call rotation）

### 紅線
測試覆蓋率**不要追數字**。10 個有意義的 test > 100 個 mock 一切的廢 test。

---

## Week 7 · 性能優化 + 文件 + Stretch Goal

### 量化 KPI
- [ ] 至少 1 個 query 透過 index 優化，記錄優化前後執行時間（截圖 EXPLAIN ANALYZE）
- [ ] Redis cache hit rate ≥ 30%（相同參數重複請求的場景）
- [ ] Rate limiting 實作完成，能用 curl 觸發 429
- [ ] 至少 1 篇 ADR 寫完（建議寫 ECS vs K8s 那篇）
- [ ] README 寫到讓陌生人 5 分鐘能理解這專案
- [ ] Stretch goal 至少 1 個完成（ONNX / admin dashboard / A/B test 三選一）

### 技術重點（要能講出來）
1. **Cache 的策略**：cache-aside、write-through、write-behind、TTL 怎麼設、cache invalidation 為什麼是「電腦科學兩大難題」之一
2. **Rate limiting 的演算法**：fixed window 為什麼有 burst 問題、sliding window 怎麼解、token bucket 的彈性
3. **Index 優化的決策樹**：什麼欄位該 index、composite index 的順序原則（最常 filter 的放前面）、covering index 的價值
4. **ONNX 為什麼快**：graph 優化、operator fusion、跨 framework portability、CPU SIMD 利用
5. **ADR 的價值**：未來自己/別人理解「為什麼這樣設計」、不用考古、降低 onboarding 成本

### 面試考點
- 🎯 「**這個 query 慢，你怎麼診斷？**」（EXPLAIN ANALYZE → buffer hit → seq scan → index 選擇 → 是不是 stats 過期）
- 🎯 「**怎麼選擇 cache 的 TTL？**」（資料變動頻率、stale 容忍度、業務需求、staleness vs hit rate trade-off）
- 🎯 「**Cache invalidation 的策略？**」（TTL、event-driven invalidation、versioned key、雙寫一致性問題）
- 🎯 「**怎麼處理 cache stampede？**」（單一請求重建多個 worker 同時打 DB；解法：probabilistic early expiration、lock、refresh-ahead）
- 🎯 「**Rate limit 實作在哪一層？**」（API gateway、reverse proxy、應用層、各自 trade-off）
- 🎯 「**Composite index 的順序為什麼重要？**」（B-tree 從左到右匹配、最常 equality filter 的放前、range 放後）
- 🎯 「**你怎麼證明你的優化有效？**」（before/after benchmark、p50/p95/p99、不是平均、要有負載測試）

### 紅線
**ONNX 卡關不要硬撐**——量化版本已經夠你講故事。Stretch goal 之所以叫 stretch 就是不做也沒關係。

---

## Week 8 · 求職衝刺

### 量化 KPI
- [ ] Demo 影片 3 分鐘 unlisted YouTube 上線
- [ ] README 含架構圖、API 文件、deploy URL、demo gif、quick start
- [ ] 至少 1 篇技術文章發布（中文 Medium 或 iThome、英文 dev.to）
- [ ] LinkedIn post 發出，含 demo gif + 專案連結
- [ ] 履歷改寫完成，MoldVis 在最上面，每個 bullet 都有數字
- [ ] 投履歷 ≥ 30 家
- [ ] Mock interview ≥ 2 場
- [ ] 5 大行為面試題答案準備好（用 STAR + MoldVis 故事）

### 技術重點 — 串完整故事
這週不是學新東西，是**把前 7 週串成一個完整的故事**。練到能在 5 分鐘內講完：
1. **問題**（30 秒）：射出成型 trial-and-error 成本高，DDIM 模型能預測光彈圖加速研發
2. **架構**（90 秒）：Next.js → FastAPI → Celery + Redis → ML worker → S3 → CloudFront
3. **關鍵決策**（90 秒）：為什麼 async、為什麼 ECS、為什麼 PostgreSQL JSONB
4. **量化成果**（60 秒）：latency X → Y、throughput、cost
5. **Trade-off 與下一步**（30 秒）：什麼沒做、為什麼、如果做會怎麼做

### 面試考點（高頻行為題，配上 MoldVis 答案）

🎯 **「介紹一下你最有挑戰的專案」**
> 講 MoldVis 從 research code 到 production system 的歷程，重點放在「我發現什麼問題 → 我怎麼設計 → 結果怎樣」

🎯 **「介紹一下你的系統架構」**
> 開講前先問「你想了解 high-level 還是 deep dive 哪個元件？」（這個問題本身就加分）

🎯 **「遇過最難的 bug？」**
> 準備一個具體的（建議 Week 4 的 async 改造一定會撞到 bug，記錄下來）

🎯 **「如果重做一次你會怎麼改？」**
> 講 ADR 裡寫的 trade-off，誠實說「我用 ECS 是因為 K8s 學習曲線，但團隊大了會 migrate」這種**有思考深度**的回答

🎯 **「為什麼用 X 而不是 Y？」**
> 任何技術選型題都用同一個結構：「我的需求是 A，X 在 B 維度勝出，Y 適合 C 場景但我這裡不是」

🎯 **「scale 100 倍會壞在哪？」**
> 練習指出 MoldVis 的 bottleneck：DB 寫入、ML worker、S3 cost、Redis memory——這題很常考，**你親手做過所以有資格講**

### 投履歷的目標分配
- **必投（高 ROI）**：台灣 AI 新創 + 半導體 AI 部門 → 這個專案最對胃
- **練習投**：海外遠端 → 練英文面試
- **長線投**：大型科技公司 → response 慢但值得放線

### 紅線
**第 8 週不要再寫 code**。重構、優化、加功能都先停。**從第 8 週週一開始，70% 時間在求職、30% 時間在 polish**。

---

## 跨週通用面試考點（任何時候都會被問）

### 系統設計
- 「設計一個 X」：先問需求（QPS、user 量、SLA）→ 估算規模 → high-level 架構 → 深入 1-2 個元件 → bottleneck 分析 → trade-off
- MoldVis 已經是一個系統設計題的答案，**面試前模擬 5 個變形題**：「如果改成影片」「如果加 collaborative editing」「如果要支援 1M users」⋯⋯

### 行為面試（STAR 格式）
- **S**ituation 背景（不要超過 30 秒）
- **T**ask 你的職責（你做了什麼，不是團隊）
- **A**ction 具體行動（這裡占 60% 篇幅）
- **R**esult 結果（**有數字**，沒數字就估一個合理的）

### 必背問題（每場面試最後幾乎都會問你）
- 「你有什麼問題想問我們？」→ 準備 5 個有水準的問題（團隊文化、tech debt、on-call、學習成長、近期挑戰），**絕對不要說沒有**

---

## 進度自檢儀表板

每週日花 30 分鐘填這個（建議用 Notion 表格）：

| 週 | KPI 達成率 | 技術重點能講出幾個 / 總數 | 面試題能答幾題 / 總數 | 預警 |
|---|---|---|---|---|
| W1 | __ % | __ / 4 | __ / 5 | |
| W2 | __ % | __ / 5 | __ / 9 | |
| W3 | __ % | __ / 5 | __ / 8 | |
| W4 | __ % | __ / 6 | __ / 9 | ⚠️ 命脈週 |
| W5 | __ % | __ / 5 | __ / 8 | |
| W6 | __ % | __ / 5 | __ / 8 | |
| W7 | __ % | __ / 5 | __ / 7 | |
| W8 | __ % | n/a | __ / 6 | 🎯 收割週 |

**預警規則：**
- 任一週達成率 < 70% → 隔週減量並 carry over
- 連續兩週 < 70% → 重新評估 timeline，考慮拉長到 10-12 週
- W4 達成率 < 80% → **強烈建議延後 W5、補足 W4**（W4 沒做好後面都白做）

---

## 最後叮嚀

> 這份作戰板的核心不是「全部做完」，是「**任何時候你都知道自己在哪、該做什麼、進度如何**」。
>
> 面試官不關心你這 8 週上了多少教學影片，他關心的是「**遇到 X 問題時你的思考過程**」。技術重點跟面試考點欄位才是真正的金礦——它們把「做專案」轉換成「能講清楚的故事」。
>
> 寫不出來的觀念，就是還沒真的懂。看懂跟講出來中間差了一個 Grand Canyon。
