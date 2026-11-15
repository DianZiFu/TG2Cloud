# v1.5.9 发布与兼容记录

状态：2026-11-15，v1.5.9 已正式发布。GitHub CI、同标签原生双架构检查及正式资源下载核验完成；真实用户 VPS 验收尚未执行，安排在发布成功后。

## 修复与兼容

普通下载与流式取源增加连续 300 秒有效字节无进度保护，覆盖消息读取与附件下载；旧协程真正退出后才按既有次数限制重试。正常慢速大文件没有总时长上限，重复计数不续时，明确监听冷却和流式下游背压单独处理。取消无法完成时保留名额并告警，不启动第二个写入者。

业务就绪增加聚合下载观察，停滞时 queue=failed、ready=false；容器心跳与传输健康分别判断。诊断区分等待下载名额和等待停滞下载回收，不覆盖显式重试、个人额度、预算及磁盘原因。旧快照兼容并把新观察显示为未核实，不输出私人文件名或凭据。

业务 schema 3、原配置、任务、下载和两个 Session 保留，无数据库迁移、新依赖或安装参数变化。一键脚本与独立 Compose 共用 Bot 核心，仍按各自安装、升级、备份与卸载边界操作。两网关提交和远端不确定结果保留策略不变；不更改网关或代理镜像。已核验原 v1.5.8 Tag 提交 `142176eb6a88e152cd2c54591af139fbe6be24fb`，生成可信升级基线，不自动执行下载的历史 Python。

## 本地验证

- `python -B .github/scripts/check_python.py all`：1664 项，1629 通过、35 条件跳过、0 失败；仓库检查、Ruff 和 compileall 通过。
- 下载保护与诊断专项 28 项全部通过，覆盖正常慢速、停滞、重复计数、明确冷却、流式背压、重复取消、有限重试、后续调度、两网关远端保留和脱敏。
- 裸 Python 3.10 专项 215 项，206 通过、9 条件跳过；宿主入口仍只用标准库。
- Bash／ShellCheck、Bandit、源码清单和网站 37 项测试通过。
- 干净公开副本组装两 Edition 一键脚本上下文与两份 format 3 Compose 候选包；各 143 个受管 Python 文件，Bot 核心一致，21 次裸 Python 3.10 入口帮助检查通过。合成镜像引用不等于正式镜像。

本机没有原生 Linux Docker、rclone 和真实网关运行条件；相关条件跳过不算通过，原生检查由 GitHub CI 执行。真实 Telegram、VPS 恢复及长期吞吐尚未执行。

## helper A、入口 B 与发布资产

辅助 A `16a51c86dd948fe897ace17e464e3ab2828ca953` 已公开，80 个 helper 文件的远端 Git blob 和 3 个匿名 raw 样本逐字节核验。入口 B 固定该真实不可变提交，不自指或使用 moving main。

A 的五组 CI 全部通过：[原生两架构](https://github.com/LuoPoJunZi/TG2Cloud/actions/runs/43686325993)、[Python 3.10／3.13 CLI](https://github.com/LuoPoJunZi/TG2Cloud/actions/runs/43686326044)、[仓库](https://github.com/LuoPoJunZi/TG2Cloud/actions/runs/43686326004)、[文档](https://github.com/LuoPoJunZi/TG2Cloud/actions/runs/43686325981)、[网站](https://github.com/LuoPoJunZi/TG2Cloud/actions/runs/43686325987)。[PR #34](https://github.com/LuoPoJunZi/TG2Cloud/pull/34)保留 A/B 审阅历史；入口 B `0bd9ec6c6c5a829a2f98c060c1d9f42a7486bff3` 的五组 CI 与同提交 main 检查通过。

[同标签原生双架构与源码发布](https://github.com/LuoPoJunZi/TG2Cloud/actions/runs/43687312562)、[Compose 原生镜像与部署包发布](https://github.com/LuoPoJunZi/TG2Cloud/actions/runs/43687618216)均成功。正式 Tag 指向 `0bd9ec6c6c5a829a2f98c060c1d9f42a7486bff3`，不移动旧标签或替换历史资产。

下载全部五份正式资产，核对 GitHub 资产 digest、两份 SHA256 清单与实际字节。源码 409 个文件逐字节匹配 Tag Git blob，包含新 watchdog；两份 format 3 包逐文件匹配同标签生成器，helper、模板、版本、提交及镜像一致。匿名读取 GHCR index、两份子 manifest 与配置，核对原生平台、1.5.9 版本和相同 revision。校验和不是数字签名。

共享镜像：`ghcr.io/luopojunzi/tg2cloud@sha256:d66e255bfffdb0c41bc0207b7eca4d1fda87ca6e75de46f701f7200c27c30353`。

| 资产 | SHA256 |
| --- | --- |
| `COMPOSE-SHA256SUMS.txt` | `ef902521ca6367bcb491d699c44b93cded926eb8a5290d69f7e23cf295b63613` |
| `SHA256SUMS.txt` | `4badad6f19d69c869c770b99cc92bf048e341d30f6c5feeff3ea6e3b60de30a1` |
| `TG2Cloud-v1.5.9-compose-clouddrive2.tar.gz` | `db8f8c86e8be6a1d9387f22b759a9c48bd3147ee3cfd7cfa41f70b40b16c40d4` |
| `TG2Cloud-v1.5.9-compose-openlist.tar.gz` | `baac7a894d93c6058ee57cb818667384448afe76e8db307e8ffb459eb99e7f2b` |
| `TG2Cloud-v1.5.9-source.tar.gz` | `453a535c490d855ac861a380db315833dba743606e580484b8cc5ab51a83cf99` |

- Linux AMD64 子镜像：`sha256:2b1e13bbd57dd3e416a1241ed15f559427319c4fbab4c285ca0275aeed346700`。
- Linux ARM64 子镜像：`sha256:9c8113c15efc860e660d7dab65bc7f2ea072ac2df11906386a536591035ebce1`。

以上是自动化与实际发布资产核验，不等于真实 Telegram、用户网关和 VPS 验收。

正式资源包括：

- `TG2Cloud-v1.5.9-source.tar.gz`
- `SHA256SUMS.txt`
- `TG2Cloud-v1.5.9-compose-clouddrive2.tar.gz`
- `TG2Cloud-v1.5.9-compose-openlist.tar.gz`
- `COMPOSE-SHA256SUMS.txt`

## 发布后实机验收顺序

1. 核对正式 Release、源码、镜像与两版包已齐，记录原 Bot／CLI 版本、部署方式和下载进度。先暂停新任务放行；已有卡住任务不直接删除、改库或批量重试。
2. 使用原部署方式导出私密一致性备份，包含配置、数据库与旁文件、两个 Session、必要网关／代理配置。旧协程尚活动时，先检查取消与保留范围；需要停 Bot 时先保存现场，不让同 Token 的两个 Bot 同时运行。
3. 公网脚本实例运行 `bash <(curl -fsSL https://raw.githubusercontent.com/LuoPoJunZi/TG2Cloud/main/install.sh) --edition clouddrive2 --release v1.5.9`；OpenList 更换 Edition，可信内网仍用 install-lan.sh，不切换部署模式。
4. 独立 Compose 下载核验完整本版包，用新包的 `bash manage.sh --root /原Compose目录` 进入“安装与升级”，选择 v1.5.9；保留原数据和配置，不用脚本接管。format 2 与早期 format 1 的限制继续见教程。
5. 核对 Bot、宿主 CLI／离线包均为 1.5.9，数据库仍为 schema 3，原授权、监听来源、上传目录和历史任务保留。业务就绪应有 transfers 聚合观察，正常忙时明确等待下载名额。
6. 用自有小文件分别测试原手动转存和管理员监听，等待从下载到完成，核对网关目录、大小及可读性，再到云盘官方客户端确认。两种网关和两种部署各自记录，未测项目继续列为未验收。
7. 正常慢速、明确 Telegram 冷却与播放器访问不应误判。仅在隔离的模拟来源测试无进度：超过阈值后看到回收／有限重试，旧协程退出后后续任务可开始；不主动破坏真实 Telegram 网络或生产网关制造故障。
8. 异常时收集版本、任务状态、聚合 readiness／diagnostics 和脱敏日志；不得公开文件名、地址、凭据、Session 或数据库。新 Bot 可能写库后失败先保留现场，不自动覆盖新库；回退仅用匹配代码和已验证快照。

没有数据库迁移不等于可以省略备份。实际 WebDAV 大小复验不等于内容哈希或云盘官方最终入库；SDK 不响应取消或下游自身卡住也不能宣称由取源保护强制修复。

[一键脚本](../VPS-INSTALL.md) · [Compose](../COMPOSE.md) · [Compose 管理](../COMPOSE-MANAGEMENT.md) · [任务恢复](../RECOVERY.md) · [备份恢复](../BACKUP-RESTORE.md)
