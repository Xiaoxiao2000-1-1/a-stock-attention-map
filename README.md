# A股资金动向地图

**机构在买什么 · 哪里估值便宜 · 钱有多集中**

单文件 HTML 看板：31 个申万一级行业的机构资金方向、估值位置、涨跌与分红吸引力。
不预测涨跌，只做公开数据的客观呈现。

## 在线查看（手机可用）

**https://xiaoxiao2000-1-1.github.io/a-stock-attention-map/**

托管在 GitHub Pages（仓库 `Xiaoxiao2000-1-1/a-stock-attention-map`，公开）。
工作日 18:30 自动发布，Pages 重建约 1 分钟后生效。手机浏览器直接打开或存书签即可，不受网络环境限制。

仓库每次发布留一个 commit，等于按日留档——想看某天的状态可以翻历史。

## 定时更新

**每个工作日 18:30**（周一至周五）自动生成，输出 `dashboard.html`。

- launchd 任务：`~/Library/LaunchAgents/com.codex.attention-index.daily.plist`
- 启动脚本：**`~/.codex/attention-index/run_dashboard.command`**（英文路径，见下方"两个坑"）
- 发布目录：`~/.codex/attention-index/gh-pages/`（GitHub Pages 仓库的本地克隆）
- 运行日志：`logs/dashboard.log`

**执行顺序**：抓数 → 生成 `dashboard.html` → 跑 `audit.py` → **审计通过才推送** →
`git push` → GitHub Pages 自动重建。审计不通过时页面照常生成，但**不会发布**，
日志里留 `publish: 跳过（审计未通过）`。

## 手动运行

```bash
~/.codex/attention-index/run_dashboard.command
```

跑完看 `logs/dashboard.log` 尾部的 `===== done ... =====`。

## 两个 macOS 坑（都踩过，别再踩）

1. **启动脚本必须放英文路径。** launchd 打开含中文的脚本路径时会编码错乱，
   报 `can't open input file: /Users/.../投研分析/...`，任务准时触发但从不执行。
   所以脚本在 `~/.codex/attention-index/`，而项目代码留在中文目录——脚本内部用
   UTF-8 字面量引用项目路径没问题。

2. **必须走 `/usr/bin/open` 而不是直接跑脚本。** launchd 进程没有 TCC 授权访问
   `~/Documents`，直接执行会报 `operation not permitted`。
   plist 里用 `/usr/bin/open` 交给 Terminal 执行，Terminal 有 Documents 授权。
   这与已有的 `com.codex.weekly-note.cli.sunday` 是同一套方案。

对应地：**改定时行为要改 `~/Library/LaunchAgents/` 里的 plist，不是项目目录里的。**
项目 `launchd/` 下那份只是模板，改完要 `cp` 过去再 `launchctl unload && load`。

## 数据源与更新频率

| 数据 | 来源 | 频率 | 滞后 |
|---|---|---|---|
| 行业指数（价/量） | 申万宏源 `index_hist_sw` | 交易日 | 当日盘后 |
| 行业估值（PE/PB/股息率） | 申万宏源 `index_analysis_daily/monthly_sw` | 日 + 月 | 日度当日、月度约 1 个月 |
| 龙虎榜机构净买 | 东财 `stock_lhb_jgmmtj_em` | 交易日 | 当日盘后 |
| 个股快照（成交额/涨跌幅） | 新浪 `stock_zh_a_spot` | 实时 | — |
| 10Y 国债 | 中债 `bond_china_yield` | 交易日 | 当日 |
| 两融（全市场情绪） | 中登 `stock_margin_account_info` | 交易日 | 约 1 周 |

## 文件

| 文件 | 作用 |
|---|---|
| `build_dashboard_v9.py` | 主生成器，输出 `dashboard.html` |
| `factor_lib.py` | 面板加载（`build_factors` 当前未被调用，留作拥挤度备用） |
| `refresh_panel.py` | 刷新 31 个行业指数缓存（不刷新数据会永久停在首次抓取日） |
| `fetch_valuation.py` / `fetch_valuation_daily.py` | 月度 / 日度行业估值 |
| `fetch_bond.py` | 10Y 国债 |
| `audit.py` | 结构与数值自检，`exit 0` 表示全过 |
| `archive/` | 历史版本与一次性探测脚本，不参与运行 |

发布不在项目目录内操作：`plist`、启动脚本、Pages 仓库克隆都在 `~/.codex/attention-index/` 下。

## 审计

```bash
python3 audit.py     # 0 失败即通过
```

覆盖：表格行列一致、标签闭合、资金面板合计、行业成交额占比合计（按取整误差设容忍）、
估值范围、数据新鲜度（按各源实际发布频率设阈值）、估值口径合法性。

## 已知限制

- 估值分位是「相对该行业自身历史」，**不能跨行业比大小**
- 煤炭/石油石化/美容护理/环保的估值历史不足 80 个月（申万 2021-12 分类改版），页面标 `*`
- 机构净买只统计上过龙虎榜的个股，银行这类很少异动的大盘行业显示 `—`
- 历史对照是相似度排序的结果，每多一个交易日数据就可能微调，不是稳定结论
