# Connect buttons and routes with backend

来源：task_e_690a0a86a0988326ba8caaf2aa52d0dd
类型：Codex 任务 / 未确定日期

### 我
<!-- mid: task_e_690a0a86a0988326ba8caaf2aa52d0dd~usertrn_e_690a0a8801ec8326bc16d35cabbfb7c4 -->
Milestone M0：把按钮/路由与后端打通（今天可动）

目标：能保存交易所 API 密钥，点“验证”有结果。只做 OKX。

M0-01｜按钮与路由梳理与接线

范围：前端所有按钮 → 目标路由/接口映射；移除无效状态。

交付：

《按钮→路由→接口对照清单》落库到 docs/routes-map.md。

修正跳转：

顶栏「Products」→ /products

顶栏「Verify」→ /verify

个人中心「API 设置」→ /settings/api

订单卡「查看」→ /orders/:id，「理赔」→ /claims/new?orderId=:id

状态统一：verifying / verified / failed 只靠 API 结果驱动。

验收：打开每个入口必达目标页面；无 404/空白页；刷新后路由状态不丢。https://github.com/wjz5788/LiqPass

### AI
<!-- mid: task_e_690a0a86a0988326ba8caaf2aa52d0dd~assttrn_e_690a0a8801f88326aff564b877ec1915 -->
> 🛠️ **Git Commit**: `Connect verify flows and routes to backend`
> 📄 **Diff Storage File**: `file_00000000b8e471f79eb89701bdb79acd`

**Summary**
* Added `docs/routes-map.md` to record the latest button, route, and backend API mappings for quick reference.


* Updated the global navigation to surface the new `/verify` entry and rerouted order/claim flows through `/orders/:id` and `/claims/new`, including a new detail view for orders and adjusted buttons that deep-link into those pages.


* Reworked the API 设置 page to fetch and persist accounts via `/api/v1/min/api-accounts`, call `/api/verify` for OKX-only verification, gate actions behind wallet connection, and surface verification progress consistently via server responses.



**Testing**
* ⚠️ `npm run build` *(fails: existing TypeScript errors in other pages)*
