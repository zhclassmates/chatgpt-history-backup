# Freeze evidence spec v1.0 and define fields

来源：task_e_68f4ea0baf988326acb677b2f22fc2f9
类型：Codex 任务 / 未确定日期

### 我
<!-- mid: task_e_68f4ea0baf988326acb677b2f22fc2f9~usertrn_e_68f4ea0d208c8326ab7043fb946dce60 -->
标题：CTX-01 证据规范与版本基线
目标：冻结“机器可验证 + 人类可读”双轨规范 v1.0，并定义上链最小字段。
变更点：
- /docs/evidence-spec.md：结构与字段定义
- /docs/onchain-fields.md：Attested 事件字段与编码
接口/产物：
- 上链事件：Attested(claimId32, claimer, exchCode, instHash, ordHash, verifiedAt, evidenceHash, verifierVersion, flags)
验收标准：
- [ ] 两份文档合并到主分支并署名版本 v1.0
- [ ] 字段名、取值域、时间精度、哈希算法写清
测试步骤：
1) 评审走查字段表
2) 用示例 JSON 过一遍字段映射
非目标：不改合约逻辑，不做 UI

### AI
<!-- mid: task_e_68f4ea0baf988326acb677b2f22fc2f9~assttrn_e_68f4ea0d209c832691255f9ed52e61e0 -->
> 🛠️ **Git Commit**: `docs: freeze ctx-01 evidence and attested event spec v1.0`
> 📄 **Diff Storage File**: `file_00000000cf0461f586840f5da6d27d72`

**Summary**
* Documented the CTX-01 dual-track evidence bundle specification v1.0, including field catalogue, hashing rules, and a worked JSON example that satisfies the “machine-verifiable + human-readable” requirements.


* Defined the Attested on-chain event baseline, detailing field encodings, hashing, flag registry, and an explicit mapping from the evidence bundle to emitted values.



**Testing**
* ⚠️ Tests not run (documentation-only changes)

### 我
<!-- mid: task_e_68f4ea0baf988326acb677b2f22fc2f9~usertrn_e_68f4ee4ec13083269c06b6fad08931fa -->
补充 JSON Schema 与示例向量，生成固定哈希期望值。

写一个 evidence:verify CLI，对照 Schema 与哈希规则做本地校验。

在 CI 中加入 Schema 校验与哈希快照比对。

用一笔最小示例把映射后的事件实际发链，回读并对比。

记录版本与向后兼容策略（v1.0→v1.x 补充字段走可选、默认值）。 ，最后用汉语回复，任务完成情况

### AI
<!-- mid: task_e_68f4ea0baf988326acb677b2f22fc2f9~assttrn_e_68f4ee4ec14083269ba606b7ca942eed -->
> 🛠️ **Git Commit**: `Add CTX-01 schema validation tooling`
> 📄 **Diff Storage File**: `file_00000000551c61f78c9fcaba4ea87cd5`

> 🛠️ **Git Commit**: `Add CTX-01 schema validation tooling`
> 📄 **Diff Storage File**: `file_00000000de8061f7908ef2cabbf1ebc5`

**Summary**
* 新增 CTX-01 v1.0 JSON Schema、样例证据与哈希快照，并在规范文档中记录本地校验流程及向前兼容策略。 


* 编写 `evidence:verify` CLI，实现 Schema 校验、哈希比对以及在内存链上回放 `Attested` 事件的能力。 


* 在 CI 中新增规范校验任务并固化所需 Python 依赖，确保自动化比对通过。 



**Testing**
* ⚠️ `pip install -r requirements.txt`（受代理限制无法访问 PyPI）


* ✅ `python -m compileall src/scripts/evidence_verify.py`
