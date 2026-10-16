# log-skimmer-corpus

Log Skimmer 的 L0 回归语料仓 (消费方主仓暂私有) — **公开 CI 日志的滚动快照**, 供其全语料回归门禁 (FN/FP/压缩率) 作可复现 fixture。

## 定位与结构

- `<host>/<owner>/<repo>/<YYYY-MM-DD>/<run_id>-<job_id>-<job_name>-<conclusion>.log` + 同名 `.json` sidecar (`{repo, run_id, job_id, conclusion, workflow_name, created_at, job_name, attempt, ci, fetched_at}`) — 嵌套自然路径, 日期/触发 job/成败一眼可读 (`repo` 为 `host/owner/repo` 完整形态, 元数据 SSOT 在 sidecar)
- `l0-baseline.json`: 压缩率防退化基线 — **随语料一起版本化**, 每次刷新由主仓 master 引擎重算; **管辖域仅本仓语料** (ci=github 的 repo), 内网侧语料的压缩率只进消费方报表不进本基线
- GHA 侧三仓 (sst/opencode · niri-wm/niri · nix-community/home-manager) 槽位: 各 14 failure + 5 success, **滚动刷新** (weekly, 保最新 run)
- Forgejo 侧 (私有内网实例) 语料**永不入本仓** — 主仓 `fetch-forgejo.sh` 按固定清单现场拉取 (隐私红线: 内网拓扑数据不出内网)

## 语料来源与再分发口径

所有日志抓取自**公开仓库的公开 CI run** (无认证 API 即可获取), 此处仅为回归测试目的再分发快照。数据按 `fetched_at` 滚动淘汰, 不积累历史。如来源项目维护者有异议, 请开 issue, 对应样本即时移除。

## 刷新机制 — GHA 全闭环自刷新

本仓自带 `refresh.yml` workflow (weekly cron + engine-sync dispatch): 拉 GHA 侧最新 run 替换滚动槽位 → **仓内镜像引擎**重算压缩率基线 → 单 commit 推回 (基线与语料恒配对版本化)。全程 GitHub 机房执行, 消费方零语料流量。

镜像目录 (`Cargo.toml` + `src/` + `rules/` + `scripts/`, 判定器副本): 由消费方主仓 `corpus-sync` **单向同步** — 勿在本仓手编 (下次同步即覆盖); 消费方引擎/规则变更后会 dispatch 即时刷新, 基线始终跟上最新判定器。

Forgejo 侧快照不参与自动刷新 (凭据内网隔离, 保留手动补抓路径)。

## 消费方式

```bash
git clone --depth 1 https://github.com/luochen1990/log-skimmer-corpus data/logs
```

主仓 CI 与 `just ci-merge` 自动完成此步 (见主仓 `scripts/fetch-corpus.sh`)。
