# log-skimmer-corpus

Log Skimmer 的 L0 回归语料仓 (消费方主仓暂私有) — **公开 CI 日志的滚动快照**, 供其全语料回归门禁 (FN/FP/压缩率) 作可复现 fixture。

## 定位与结构

- 样本路径 `<host>/<owner>/<repo>/<YYYY-MM-DD>/<run_id>-<job_id>-<job_name>-<conclusion>.log.gz` (gzip 压缩, 证据原样不截断), 配对 sidecar **`X.log.gz ↔ X.log.json`** (同名去 `.gz` 加 `.json`, 一一对应) — 字段 `{repo, run_id, job_id, conclusion, workflow_name, created_at, job_name, attempt, ci, gate_exempt, fetched_at}`, 元数据 SSOT 在 sidecar, 路径仅为可读性投影 (`repo` 为 `host/owner/repo` 完整形态)
- 示范项目集 **124 项跨 25 栈标签** (rust/go/ts/python/java/cpp/... 主流 fmt/lint/typecheck/test 工具族), 清单 SSOT: 仓根 [`projects.yaml`](projects.yaml); 槽位 8 failure + 3 success per repo (**既有 3 仓 niri-wm/niri · anomalyco/opencode · nix-community/home-manager 为 14F+5S**, 语料连续性), **weekly 滚动刷新保最新 run**
- `gate_exempt: true` (清单 `exempt` 字段驱动): 该项目工具暂无覆盖规则, 消费方 L0 门禁豁免其 FN/FP/零块断言的**过渡通道**, 规则适配达标后移出 — 见上方「语料来源与再分发口径」, 样本本身与其他样本同源同权

## 语料来源与再分发口径

所有日志 (含 `gate_exempt` 样本) 抓取自**公开仓库的公开 CI run** (无认证 API 即可获取), 此处仅为回归测试目的再分发快照。数据按 `fetched_at` 滚动淘汰, 不积累历史。如来源项目维护者有异议, 请开 issue, 对应样本即时移除。

## 刷新机制 — GHA 自刷 (orphan 单提交, 源头自包含)

本仓自带 `refresh.yml` workflow (weekly cron + manual dispatch): 按 `projects.yaml` **全量重抓替换**旧样本 → 读树重建**单个 orphan commit** force-push master — 仓体积恒定, 历史永不积累。**clone 者须知**: master 历史随时被强推重写, 更新用 `git fetch + git reset --hard origin/master`, **不要 `git pull`** (必然分叉报错)。`scripts/fetch-logs.sh` 是自带的槽位抓取执行器 (源文件在 log-skimmer dev 仓, 随发布面节奏手动同步; `--projects projects.yaml` 驱动)。语料是源头材料: **不算基线、不构建引擎、不依赖任何其他仓** (压缩率防退化由消费方 dev 仓 CI 的双测对比承担)。

## 消费方式

```bash
git clone --depth 1 https://github.com/luochen1990/log-skimmer-corpus data/logs
```

主仓 CI 与 `just ci-merge` 自动完成此步 (见主仓 `scripts/fetch-corpus.sh`, 已适配 orphan 强推历史的 `fetch + reset --hard` 更新语义)。

## License

仓内编排与元数据以 CC0 1.0 发布 (见 [LICENSE](LICENSE)); 日志样本内容版权归原作者所有, 再分发口径见上文「语料来源与再分发口径」。
