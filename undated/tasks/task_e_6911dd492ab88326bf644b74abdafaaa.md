# Conduct a read-only end-to-end audit

来源：task_e_6911dd492ab88326bf644b74abdafaaa
类型：Codex 任务 / 未确定日期

### 我
<!-- mid: task_e_6911dd492ab88326bf644b74abdafaaa~usertrn_e_6911dd4acfd88326a3a50d46f8b7cf3c -->
https://github.com/wjz5788/LiqPass # 项目诊断提示词模板 v1（LiqPass 适配）

> 用法：把以下块状提示词复制到任意 AI（通用大模型、代码助手、审计工具），替换花括号变量即可开跑。所有模板都 **只读诊断、不改代码**，输出为 **Markdown**。若结论缺证据，必须标注“**假设**”。

---

## A. 总控体检（一次性拉通 90 分钟）

**粘贴即用（替换花括号）**：

你是一个“全链路技术体检官”。请对我的项目做 **一次性端到端体检**，仅输出 Markdown 报告，不生成任何代码。

**上下文**：

* 仓库：{{REPO_URL}}
* 分支：{{BRANCH|main}}
* 主要目录：apps/us-backend（Express、SQLite）、apps/us-frontend（React/Vite/Tailwind）、apps/jp-verify（FastAPI）、apps/chain-listener（事件监听）、packages/abi（ABI与地址）
* 运行架构：US 服务器（前后端），JP 服务器（验证/证据），Base 主网（chainId=8453），结算币种 USDC。
* 产品要点（只作校验维度，不要求你给方案）：SKU=24h 当日保 / 8h 时段保 / 月度回撤保 / 无爆仓返现；只认 **PremiumPaid 事件 + 交易所只读数据**；证据→摘要（evidenceRoot/Merkle）→可复核。

**硬性规则（必须遵守）**：

1. 只读诊断，不运行破坏性命令；不请求任何密钥、私钥或写权限。
2. 发现问题必须给出 **证据→风险→影响→修复建议（不写代码）→验证方法→受影响范围→优先级(P0/P1/P2/P3)**。
3. 结论缺证据时，标注“假设”；对关键断言给出引用（文件路径与行号/接口名/事件名）。
4. 输出结构固定如下：

   * `# 概览`
   * `# 关键路径体检（下单→验证→赔付）`
   * `# 模块诊断`（合约 / us-backend / jp-verify / chain-listener / 前端 / 数据库 / 运维）
   * `# 一致性检查`（接口契约、字段/单位、事件字段、环境变量样例 vs 校验器）
   * `# 安全与合规`（鉴权、签名/EIP‑712、重放/nonce、最小权限、限额、等待期、黑名单、审计日志）
   * `# Top 10 阻断项（P0/P1）`
   * `# 7 天落地包`（按天排期、负责人、交付标准DoD）

**体检重点（逐条检查）：**

* 关键字段与单位是否一致：`premiumUSDC` 是否采用“六位小数的整数”（1e‑6 USDC）。
* 合约：`CheckoutUSDC.sol` 是否使用 `SafeERC20`、`Pausable`、`ReentrancyGuard`；事件 `PremiumPaid` 字段是否足以唯一对账（payer、policyId/quoteId、amount、token、chainId、txHash等）；签名是否防重放（nonce、deadline、chainId、attestor分离）。
* us-backend：订单创建是否一律 `pending`；鉴权（JWT/HMAC）与速率限制；幂等性键；外部依赖超时/重试；日志可审计性（traceId）。
* jp-verify：只读 API（OKX/Binance）拉取证据，是否有等待期/限赔/黑名单；evidenceRoot 生成是否有可复核脚本与存档。
* chain-listener：是否可从 `PremiumPaid` 构建事实账本；断点续扫与重放保护。
* 前端：金额/单位转换一致；全局 Authorization 注入；错误提示与回退路径。
* 环境变量：`.env.sample` 与 `EnvValidator` 一致；缺失即 fail‑fast。
* DevOps/CI：固定 Node/PNPM 版本；构建可重现；Secrets 管理；备份与回滚；健康检查与告警。

**评分与交付**：

* 每模块给出 **一致性收敛分**（0–100）与 **风险热图**。
* 报告末尾输出一张表：`问题|证据|优先级|影响|负责人|建议工期|验证方法`。

---

## B. “阻断项→落地包”提示词（把问题转成7天执行计划）

基于上次体检报告（粘贴其“Top 10 阻断项”列表），请把每一项转化为 **7 天落地包**：

* 输出格式：按天列出 Day1…Day7；每天给“目标、步骤、产物（PR/文档/配置）、风险回避、回滚方案、验收标准DoD”。
* 约束：不写具体代码；产物以 **PR 标题/文档标题/脚本名** 级别描述；每项标注负责人与预计时长（h）。
* 收敛要求：优先解决 P0→P1；任何需要密钥/生产访问的步骤必须提供 **安全替代路径（本地模拟/只读Mock）**。

---

## C. 模块深挖模板（四选其一反复使用）

**C‑1 合约（CheckoutUSDC / 赔付合约）**
请仅审查合约安全与可运维性：

* 证据：列出 `contracts/*.sol` 的关键函数与事件，逐一给出行号引用。
* 风险清单：重入、签名可塑性、权限边界、暂停/紧急开关、allowance/Permit2 差异、金额单位、事件字段不全、可升级性（如有）。
* 建议：给出 **无代码** 的修复要点与测试场景（主网回归/本地fork）。

**C‑2 us‑backend（Express/SQLite）**

* 入口路由、鉴权、中间件链路图；速率限制与幂等。
* `.env.sample` vs `envValidator` 差异；缺失即启动失败的策略检查。
* 订单状态机：`pending→paid→verified→claimable→paidOut/rejected` 是否闭环；每一步日志与追溯证据是否充分。

**C‑3 jp‑verify（FastAPI）**

* 外部 API 的失败矩阵（超时、限流、5xx）；重试与退避；证据快照与归档格式（原始/归一化JSON、Merkle root）。

**C‑4 chain‑listener（事件监听）**

* 断点续扫策略、确认数（confirmations）、重放保护；与后端账本一致性；单写原则。

每个子模块输出：`问题→证据→风险→建议→验证`，并打 P0/P1/P2。

---

## D. 证据收集 / 一致性对照模板

请严格按下列“对照表”逐项比对，并把不一致项输出为列表：

* **字段与单位**：`premiumUSDC` 是否在前后端/合约/数据库统一为“六位小数整数”。
* **接口契约**：OpenAPI（若有）与实现是否一致；必填/可选字段；错误码枚举。
* **事件契约**：合约事件字段是否足够做唯一对账；监听与入库字段是否丢失。
* **环境变量**：`.env.sample`、`README`、启动校验器的键集合是否一致；是否存在“口头约定”。
* **地址与网络**：`chainId`、USDC 地址、合约地址是否在 `packages/abi/addresses.json` 中唯一事实来源（SSOT）。

输出一张“**一致性收敛表**”：`对象|事实来源A|事实来源B|差异|风险|建议`。

---

## E. 报告格式（强约束）

在任何诊断中，请使用以下固定块：

```
# 概览
一句话结论 + 总分（0–100）

# 关键发现（最多10条，按P0→P1排序）
- [P0] 标题 —— 证据(路径:行号) —— 影响 —— 建议
...

# 模块评分
| 模块 | 分数 | 主要风险 |
| --- | --- | --- |
| 合约 |  |  |
| us-backend |  |  |
| jp-verify |  |  |
| chain-listener |  |  |
| 前端 |  |  |
| 数据库/运维 |  |  |

# 不确定/假设
- 假设点1（待证据）

# 7天落地包（骨架）
- Day1: ...
- ...
```

另外附一个 **JSON 片段** 便于我导入任务系统：

```json
{
  "issues": [
    {
      "title": "",
      "priority": "P0",
      "evidence": ["apps/.../file.ts:120-140"],
      "impact": "",
      "suggestion": "高层级操作要点（不写代码）",
      "owner": "",
      "eta_hours": 4,
      "verify": "如何验收"
    }
  ]
}
```

---

## F. 时间箱与工作流

* 初次体检时间箱：90 分钟；每 30 分钟给一次中间态小结（增量发现）。
* 输出顺序：先 `关键路径体检` → 再 `Top 10 阻断项` → 最后 `7天落地包`。
* 对未取证的结论，一律放入“**不确定/假设**”。

---

## G. 安全与边界（对 AI 的硬性限制）

* 禁止：写入仓库、执行交易、请求密钥/私钥、扫描内网；生产环境只读。
* 允许：读取公开仓库、静态分析、生成报告与任务清单、提出验证脚本思路（无代码）。

---

## H. 快速起手词（超短版）

对 {{REPO_URL}}@{{BRANCH}} 做 **只读端到端体检**。固定输出：概览、关键发现(≤10)、模块评分、Top 10 阻断项、7天落地包。每条必须含证据(路径:行号)与优先级(P0/P1/P2)。重点校验：`premiumUSDC` 单位一致、`PremiumPaid` 事件字段充分性、鉴权/速率限制/幂等、evidenceRoot 流程、.env.sample 与校验器一致、地址与链Id 的 SSOT。**不写代码**，缺证据标“假设”。不写代码，只是做审核，最后生成md 。汉语输出

### AI
<!-- mid: task_e_6911dd492ab88326bf644b74abdafaaa~assttrn_e_6911dd4acff08326a5ca04030c93d189 -->
# 概览
发现 4 个 P0 与多项 P1 隐患，关键链路存在认证与对账断点，当前端到端健康度约 **38/100**。

# 关键发现（最多10条，按P0→P1排序）
- [P0] 热钱包登录未校验签名 —— `verifyLogin` 仅检查字段存在即颁发会话令牌，无任何签名/nonce校验，任意地址可冒充高净值用户。

 —— 影响：认证面完全失守，风控与下单鉴权形同虚设。—— 建议：接入 EIP‑4361/siwe 等签名校验、绑定一次性 nonce，并在数据库保存验签摘要供审计。
- [P0] 服务启动即崩溃：缺失 `middleware/requestLogger` 实现 —— `app.ts` 引入并调用该中间件但目录下不存在文件，Node 在冷启动时报错退出。

 —— 影响：us-backend 无法上线，所有接口不可用。—— 建议：补齐 requestLogger 模块或在启动前移除引用，采用已有 `requestId` 中间件输出可审计日志。
- [P0] `PremiumPaid` 事件字段不足，链上账本无法复核 —— 合约仅输出 `orderId/buyer/amount/quoteHash/timestamp`，监听器却期望 `token/treasury` 等字段并写入数据库，结果持久化 NULL，无法保证唯一对账与重放安全。

 —— 影响：支付事实账本失真，赔付 & 风控链路无法达成。—— 建议：扩展事件字段（token/treasury/chainId/paymentTx 等）并同步监听服务、后端模型。
- [P0] 前端 API 基址环境变量拼写错误 —— API 服务构造函数读取 `VITE_API_BASE_URL`，但校验器仅暴露 `VITE_API_BASE`，最终拼出 `undefined/orders` 导致所有 API 请求失败。

 —— 影响：终端用户无法下单、查询或支付。—— 建议：统一变量命名或在 env 校验器中补充别名，新增 e2e 冒烟测试覆盖。
- [P1] 金额单位多处不一致 —— 产品/报价模型仍按浮点数计算，早期迁移脚本存 DECIMAL(20,8)，监听器又存 TEXT，已与订单服务的 6 位整数契约背离。

 —— 影响：金额换算、对账与风控难以统一。—— 建议：制定单一 1e‑6 USDC 契约并补齐迁移脚本、类型定义、监听表结构。
- [P1] 监听服务缺乏可靠断点/字段校验 —— 账本更新依赖缺失字段的事件、以最新区块号计算确认数且未经多节点比对，风险日志只是 `console.warn`。

 —— 影响：遇到分叉或字段缺失时会把订单错误标记为已付。—— 建议：引入确认区块缓存、字段空值保护与重放审计表。
- [P1] JP 验证服务允许内联高敏密钥直连交易所，缺少等待期/赔付上限/黑名单控制 —— 接口直接把用户密钥透传到 OKX，无额外风控策略与分离 attestor。

 —— 影响：合规与密钥管理风险，且赔付校验无法落地。—— 建议：改为只读只需要的公开接口，或引入密钥托管/白名单与等待期参数。
- [P1] OpenAPI 契约陈旧 —— 仍描述旧版 `/pricing/quote` 接口、`skuId` 为整数，与实际 `/orders/preview` 等现状严重不符。

 —— 影响：集成方误解接口、自动生成 SDK 失败。—— 建议：用真实路由重新生成 OpenAPI，并纳入 CI。
- [P1] 链上地址 SSOT 为空白 —— `packages/abi/addresses.json` 各网络 `CheckoutUSDC` 均为空字符串，团队成员无法确认主网/测试网地址。

 —— 影响：部署、监听、前端配置容易跑错地址。—— 建议：补齐地址并约定唯一来源，结合 EnvValidator 校验。
- [P1] 监听器 HTTP API 无鉴权 —— `/orders/:id`、`/premium-paid` 等接口公开暴露链上对账数据，无 JWT/API key 防护。

 —— 影响：外部可探测内部账本、推断用户资产活动。—— 建议：复用 us-backend API key 中间件或至少加 Basic Auth + IP 白名单。

# 模块评分
| 模块 | 分数 | 主要风险 |
| --- | --- | --- |
| 合约 | 40 | `PremiumPaid` 事件字段缺失导致无法对账。

 |
| us-backend | 30 | 登录验签缺失、启动即崩溃。

 |
| jp-verify | 55 | 内联密钥、缺少赔付策略控制。

 |
| chain-listener | 45 | 事件字段 NULL、无鉴权 API。

 |
| 前端 | 35 | API 基址拼写错误导致全站瘫痪。

 |
| 数据库/运维 | 45 | 金额单位混乱、地址 SSOT 空缺。

 |

# 关键路径体检（下单→验证→赔付）
- 下单/鉴权：钱包登录未验签，攻击者可直接获取 JWT；缺失的 `requestLogger` 使服务无法启动，也无法生成审计日志。


- 订单创建：内存订单服务使用 6 位整数，但数据库/监听仍有浮点/文本字段，金额一致性尚未闭环。


- 支付监听：合约事件缺少 token/treasury，监听器写入 NULL 并继续标记订单已付，且 HTTP API 无鉴权曝光账本。


- 证据验证：jp-verify 直接接受用户提供的 API 密钥，缺乏等待期/赔付上限/黑名单策略，只生成证据文件。


- 赔付闭环：订单状态仅靠监听器 `markPaid` 更新，无多节点确认或重放保护，且链上地址 SSOT 缺位，难以完成赔付复核。



# 模块诊断
## 合约（CheckoutUSDC）
- 问题→事件缺少 token/treasury/chainId；`PremiumPaid` 无法唯一对账。


- 证据→监听器代码要求这些字段并写数据库。


- 风险→账本失真、无法对账，赔付流程中断。P0
- 建议→扩展事件、同步 ABI、更新监听/后端模型。
- 验证→部署新合约到测试网，回放支付交易核对数据库字段。

## us-backend（Express/SQLite）
- 问题→钱包登录未验签；requestLogger 缺失导致启动失败；金额单位混用；OpenAPI 陈旧。


- 风险→P0 认证绕过，服务无法上线，文档误导集成。P0/P1
- 建议→落地 SIWE、补齐中间件、统一金额单位、重生成 OpenAPI。
- 验证→编写登录验签单测、启动自检、OpenAPI 与路由快照比对。

## jp-verify（FastAPI）
- 问题→接收内联 API 密钥直接请求交易所，无等待期/赔付限额/黑名单；仅文件存证。


- 风险→密钥泄露、赔付策略无法执行。P1
- 建议→改用只读端点、引入 key alias/白名单、增加等待期和限赔参数。
- 验证→模拟无密钥请求、黑名单账号，确保被拒绝并生成审计日志。

## chain-listener（事件监听）
- 问题→依赖缺失字段、确认数基于单节点、HTTP API 无鉴权、DB 列类型 TEXT。


- 风险→订单错误标记已付、账本暴露。P0/P1
- 建议→补字段校验、引入多节点确认与断点续扫、API 增加鉴权、列类型改为 INTEGER。
- 验证→Fork 环境重放事件、执行 reorg 测试。

## 前端
- 问题→API 基址变量拼写错误导致请求失败；金额展示混用 `premium`/`premiumUSDC`。


- 风险→用户无法下单或看到错误金额。P0/P1
- 建议→统一 env 命名、加入集成测试；页面统一使用 USDC6d 工具。
- 验证→E2E 下单流程、比对金额显示。

## 数据库/运维
- 问题→旧迁移仍存浮点金额、新迁移未清理；地址 SSOT 空白；监听 API 未受保护。


- 风险→备份/回滚困难、配置漂移。P1
- 建议→补全迁移、维护地址清单、为监听 API 配置鉴权。
- 验证→数据库一致性检查、配置变更审计。

# 一致性检查
| 对象 | 事实来源A | 事实来源B | 差异 | 风险 | 建议 |
| --- | --- | --- | --- | --- | --- |
| `premiumUSDC` 单位 | 订单类型定义（6d）

 | 旧迁移/监听（DECIMAL/TEXT）

 | 单位不一致 | 对账失败 | 全链统一 1e‑6 USDC，并迁移旧表 |
| API 基址 | 前端 env 校验 `VITE_API_BASE`

 | API 服务读取 `VITE_API_BASE_URL`

 | 变量名不匹配 | API 调用全失败 | 统一命名或提供别名 |
| 事件字段 | 合约事件

 | 监听器解析需求

 | token/treasury 缺失 | 无法对账 | 扩展事件/监听模型 |
| 环境变量样例 | `.env.sample` 新键齐备

 | `EnvValidator` 校验覆盖相同键

 | 一致 | 低 | 保持 |
| 地址与网络 | `packages/abi/addresses.json`

 | 其他模块引用 ENV | SSOT 为空 | 部署出错 | 补填地址并引用该文件 |

# 安全与合规
- 鉴权：钱包登录未验签；监听 API 全公开；JP 验证内联密钥缺少权限隔离。


- 签名/Nonce：无 EIP‑712/EIP‑4361 支持；quoteHash 注册未关联链下签名复核。


- 重放防护：监听器直接信任单节点确认，缺少 nonce/多节点比对。


- 最小权限：地址 SSOT 缺失、API 无鉴权，违反最小权限原则。


- 限额/等待期：JP 验证无等待期、赔付上限逻辑。


- 黑名单/审计：缺少钱包/账户黑名单策略，requestLogger 缺失导致审计日志空缺。



# Top 10 阻断项（P0/P1）
| 问题 | 证据 | 优先级 | 影响 | 负责人 | 建议工期 | 验证方法 |
| --- | --- | --- | --- | --- | --- | --- |
| 钱包登录未验签 | 

 | P0 | 任意人可冒充用户 | 待指派 | 8h | 回归签名验签单测 + 手工登录冒烟 |
| 缺失 requestLogger 导致启动失败 | 

 | P0 | 后端无法启动 | 待指派 | 4h | npm start 验证、日志输出检查 |
| PremiumPaid 事件字段不足 | 

 | P0 | 无法对账/赔付 | 待指派 | 16h | 本地 fork 支付→数据库字段校验 |
| 前端 API 基址变量错误 | 

 | P0 | 所有前端请求失败 | 待指派 | 6h | Vite 本地启动 + 下单流程 |
| 金额单位不一致 | 

 | P1 | 对账混乱 | 待指派 | 12h | SQL 差异扫描 + 对账脚本 |
| 监听器字段/确认逻辑薄弱 | 

 | P1 | 误标 paid | 待指派 | 14h | Fork 重放 + reorg 测试 |
| JP 验证缺少密钥治理 | 

 | P1 | 合规/密钥风险 | 待指派 | 12h | Mock 密钥请求 + 风控用例 |
| OpenAPI 文档过期 | 

 | P1 | 集成方误用 | 待指派 | 6h | 自动生成 OpenAPI 与 SDK smoke |
| 合约地址 SSOT 为空 | 

 | P1 | 部署/监听错链 | 待指派 | 4h | 补录地址 + EnvValidator 校验 |
| 监听器 API 无鉴权 | 

 | P1 | 数据泄露风险 | 待指派 | 6h | 新增鉴权 + 401 冒烟 |

# 不确定/假设
- 假设链上合约尚未与生产支付绑定（仓库地址为空），需上线前再次核对真实部署信息。


- 假设 requestLogger 缺失是遗漏而非自动生成文件，未在其他分支找到实现。

# 7天落地包（骨架）
- Day1: 建目标与验收 —— 梳理 auth/支付链路 P0 需求；召开 Kick-off，确定签名规范与事件字段列表。产物：会议纪要+需求确认表。风险回避：同步全员时间表；回滚：若范围过大，拆分为子任务。DoD：产品/技术双签。
- Day2: 修复登录与启动阻断 —— 落地 SIWE 验签、补齐 requestLogger；联调 JWT 生成。产物：`feat: enforce wallet signature` PR、`chore: add request logger middleware` PR。风险：兼容旧 Token；回滚：保留旧接口 behind flag。DoD：单测+本地冒烟通过。
- Day3: 扩展合约事件与 ABI —— 设计新事件字段、编写迁移脚本、更新监听器解析。产物：`contracts: PremiumPaid v2` PR、监听器更新 PR。风险：合约升级窗口；回滚：保留旧事件监听通道。DoD：fork 网络回放成功。
- Day4: 前端环境与 API 验收 —— 统一 env 变量、补充集成测试、验证金额显示。产物：`fix: api base env` PR、Playwright 测试报告。风险：多环境差异；回滚：feature flag 切换。DoD：CI + 本地验收。
- Day5: 金额单位迁移 —— 编写 SQL/数据迁移脚本、更新模型/DAO/监听表为 INTEGER 6d。产物：`migration: usdc6d unification` PR、回滚脚本。风险：历史数据精度丢失；回滚：备份数据库。DoD：对账脚本通过。
- Day6: JP verify 风控治理 —— 引入 API key alias/白名单、等待期与限赔参数，更新文档。产物：`chore: jp-verify security controls` PR、运行手册。风险：交易所限流；回滚：保留原薄代理。DoD：Mock 测试、文档签字。
- Day7: 文档 & 运维收尾 —— 更新 OpenAPI、填充地址 SSOT、为监听 API 加鉴权，完成全链路回归。产物：OpenAPI 版本发布、`addresses.json` 更新、运维手册。风险：CI 失败；回滚：保留旧文档标签。DoD：全链路 checklist 签字。

```json
{
  "issues": [
    {
      "title": "Wallet login accepts unsigned payloads",
      "priority": "P0",
      "evidence": ["apps/us-backend/src/services/authService.ts:82-236"],
      "impact": "Attackers can impersonate any wallet and place orders.",
      "suggestion": "Introduce SIWE/EIP-4361 verification with nonce storage and audit logging.",
      "owner": "",
      "eta_hours": 8,
      "verify": "Unit test signature validation and attempt login with forged payload."
    },
    {
      "title": "Missing requestLogger crashes backend on start",
      "priority": "P0",
      "evidence": ["apps/us-backend/src/app.ts:1-77"],
      "impact": "API service cannot boot.",
      "suggestion": "Restore the middleware implementation or remove the import while adding request ID logging.",
      "owner": "",
      "eta_hours": 4,
      "verify": "npm start and ensure logs emit request IDs."
    },
    {
      "title": "PremiumPaid event lacks reconciliation fields",
      "priority": "P0",
      "evidence": ["contracts/CheckoutUSDC.sol:35-118","apps/chain-listener/src/watchers/checkoutUSDC.mjs:73-187"],
      "impact": "Ledger entries miss token/treasury data, breaking payout audits.",
      "suggestion": "Emit token/treasury/chainId in events and update listener/database schema accordingly.",
      "owner": "",
      "eta_hours": 16,
      "verify": "Replay payment on fork network and inspect premium_paid table fields."
    },
    {
      "title": "Front-end API base misconfigured",
      "priority": "P0",
      "evidence": ["apps/us-frontend/src/services/api.ts:148-198","apps/us-frontend/src/env.ts:6-110"],
      "impact": "All client API calls target undefined host.",
      "suggestion": "Align env key names or expose aliases and add integration smoke tests.",
      "owner": "",
      "eta_hours": 6,
      "verify": "Run Vite app and complete order flow with corrected base URL."
    }
  ]
}
```

### 我
<!-- mid: task_e_6911dd492ab88326bf644b74abdafaaa~usertrn_e_6911e04acc808326921885f60c5c9f6b -->
你先修复，4个p0 , 有 什么，信息需要我提供，

### AI
<!-- mid: task_e_6911dd492ab88326bf644b74abdafaaa~assttrn_e_6911e04acca0832684dbd1d522d28867 -->
> 🛠️ **Git Commit**: `Secure wallet login and enrich PremiumPaid event`
> 📄 **Diff Storage File**: `file_00000000974c71f782592df3e226ac46`

**Summary**
* Added wallet challenge issuance, strict signature validation, and audit logging to the wallet login flow while introducing a dedicated challenges table and registering its migration.


* Introduced a structured request logger tied to request IDs and wired it into the Express middleware stack for traceable auditing.


* Enriched the PremiumPaid event/ABI with token, treasury, and chainId metadata, updated listeners and contract handlers accordingly, and unified the front-end API base configuration with an alias-friendly validator.



**Testing**
* ⚠️ `pnpm --filter us-backend test` (fails: `vitest` binary missing in container)


* ⚠️ `npm --prefix apps/us-backend test` (fails: `vitest` binary missing in container)


* ⚠️ `npm --prefix apps/us-backend run build` (fails: TypeScript build blocked by absent runtime dependencies/typings in environment)
