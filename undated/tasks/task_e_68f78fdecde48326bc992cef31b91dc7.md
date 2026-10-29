# Add deploy scripts with release and rollback

来源：task_e_68f78fdecde48326bc992cef31b91dc7
类型：Codex 任务 / 未确定日期

### 我
<!-- mid: task_e_68f78fdecde48326bc992cef31b91dc7~usertrn_e_68f78fe17c808326a3f2eed5cb35d676 -->
把下面任务卡-01贴给 Codex，选 wjz5788/leverageguard-reboot@dev 运行。

任务卡-01：部署骨架与一键发布（无 CI/CD）
目标：新增 deploy/ + 本地一键发布与回滚脚本，不改业务代码。
产出：

deploy/US_RELEASES.md、deploy/JP_RELEASES.md（目录规范：US=/srv/web/us/{releases,shared,current}；JP=/srv/app/jp/{releases,shared,current}）

deploy/release.sh（本地：构建→rsync 到 releases/<YYYYMMDD-HHMM>→切 current→US reload Nginx / JP restart 服务→探针→失败自动回滚）

deploy/rollback.sh（回滚上一版本）

deploy/vars.example（US/JP 主机、端口、用户、目标路径、探针 URL）

deploy/VERIFY.md、deploy/CHANGELOG.md
约束：仅 Node LTS 与 Python 3.10；无 Docker/无 Actions；路径全用绝对路径；仅保留最近 3 个版本。
验收：合并后出现以上文件；本地复制 deploy/vars.example 为 deploy/vars 填好变量后，release.sh 能发布到 US/JP，探针 200；rollback.sh 10 秒内恢复。

### AI
<!-- mid: task_e_68f78fdecde48326bc992cef31b91dc7~assttrn_e_68f78fe17c948326b89af26b2bc43f90 -->
> 🛠️ **Git Commit**: `Add deployment skeleton with release and rollback tooling`
> 📄 **Diff Storage File**: `file_000000007a5061f788f846a9fc72e78d`

**Summary**
* Implemented `deploy/release.sh` to build artifacts, sync timestamped releases for US and JP hosts, update symlinks, restart services, verify probes, retain the latest three releases, and auto-rollback on failure.


* Added `deploy/rollback.sh` to switch both regions back to the previous release, rerun service hooks, and confirm probe health before exiting.


* Provided deployment documentation and configuration artifacts covering regional directory layouts, verification steps, changelog tracking, and a configurable vars template for operators.



**Testing**
* ✅ `bash -n deploy/release.sh` 


* ✅ `bash -n deploy/rollback.sh`
