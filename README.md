# 灵感官 · Inspiration Officer

> 一个会**主动找事做**的科研助手：你不需要任何详细的指令。它自己会去读新论文、扫描你本地的代码和数据集，然后值得做的事整理成可验证的卡片，送到你本机的评价页。只需要输入一句话。

当前状态：`1.5.9-test` · 平台：[SciencePal](https://sciencepal.ai)

---

## 为什么做这个

现在的 AI 科研助手基本都是你提出需求，它马上执行。我不继续监督和反馈，它就会停在那里。

但人和 AI 的能力结构其实是不对称的：

**人的判断力强，但一天能投入的时间有限；**

**AI 的判断力还不够，但它可以一直读、一直算。**

所以合理的分工是**让它用能一直工作的长板，去帮我主动从海量信息当中发掘我真正需要的灵感**：
不需要人类一直反馈指令，它自己去找值得做的事，整理成建议，来找我申请。

> 它不替我做科研，它只是在我的许可下，自己去找该做的事；
> 每一条建议都是一次申请，我的每一次评价都让它更具有科研品味。

---

## 它做什么

| 能力 | 说明 |
|---|---|
| **主动探索论文** | 按你的方向抓 arXiv → 去重 → 读摘要（或全文）→ 打分 → 生成候选卡片 |
| **本地观察** | 扫你本机的代码 / 笔记 / 论文库 / 数据 / 日志，找出函数的参数太多、笔记里说过想试某个新方法这类事 |
| **四类卡片** | A 优化建议 · B 知识建议 · C 假说 · D 流程改进，各自独立配额 |
| **可审计卡片** | 每张卡都有来源、逐字引文、为什么和你有关、最小验证、判据、**推翻条件**、成本估计 |
| **评价与学习** | 采纳 / 否决 / 已做过 / 不重要 / 追问 / 立刻动手 / 改写；权重按你的反馈保守调整，并给你周报 |
| **立刻动手** | 按 6 先弹「动手单」表单（场景/参数/方法/产出/对照/判据/交付），批准后才执行，产出回执含产出实测、对照实测、八项检查 |
| **本机评价页** | `http://127.0.0.1:8765/`，卡片一到达就弹 Windows 通知 |

---

## 工作流

```text
你在网页里说"主动探索灵感"
        │
        ▼
prepare ── 抓 arXiv / 读本地观察 / 去重 / 打分 / 入围
        │    （论文卡：一次读 triage_shortlisted.json → 一次写 cards.json）
        ▼
insp-card ── 生成卡片（论文卡 cards.json；观察卡 cards/NN.json）
        │
        ▼
finalize ── 校验 + 配额裁剪 + 输出 INSP_BUNDLE_JSON
        │
        ▼
本机收货员 ── 入库 → 评价页 + 通知
        │
        ▼
你的评价 ── verdicts.jsonl → 权重 / 周报 / 负面清单
```

---

## 快速开始

```powershell
# 1) 本地配置 + 状态
copy platform\config.example.json config.json
xcopy /E /I insp-init\assets\灵感状态模板\inspiration inspiration

# 2) 导入技能、建灵感官（需要 SciencePal 会话令牌）
python platform\setup_platform.py --config config.json --replace
python platform\setup_platform.py --config config.json --check

# 3) 启动本机评价页
powershell -ExecutionPolicy Bypass -File platform\windows\start_review_server.ps1
```

然后打开 <http://127.0.0.1:8765/>，在 SciencePal 网页里对「灵感官」说 **“建立画像”**，
再新开一个对话说 **“主动探索灵感”**。

**完整安装步骤（含令牌、故障排查、升级、卸载）见 [INSTALL.md](INSTALL.md)。**

---

## 目录结构

```text
insp-nightly/ insp-card/ insp-profile/ …   7 个技能：导入到 SciencePal，在云端沙箱里运行
platform/                                   本机侧：安装、receiver、评价页、周报、立刻动手、本地观察
insp-init/                                  初始化模板（状态模板、画像问题）
scripts/                                    vendor 同步、打包
tests/                                      离线测试（不连平台、不花 token）
docs/                                       开发文档、发布门槛、验收节点
library/（需要用户自己建，不进仓库）             你的资料库：code / notes / papers / data / logs
```

---

## 设计原则

1. **它不替我做决定，它替我找选项。** 所有建议都是申请，批准才执行。
2. **它是来找我聊天的，不是来刷屏的。** 三层去重，宁缺毋滥。
3. **它知道，不等于它会。** 每条判断都必须有来源、可追溯、可被一键删除。

---

## 隐私

- 你的**原始文件不出本机**：本地观察只上传脱敏后的片段（可用 `observe.upload_code_snippets=false` 只传指标）；
- 上传到沙箱的只有技能 + 一份本地状态快照（`state.json` / `observed.jsonl` / 画像 / taste规则）」；
- 令牌保存在 `%USERPROFILE%\.sciencepal\`，不写进仓库、不写进日志；
- 本仓库不含任何个人配置或历史数据。

---

## 已知限制（Beta）

- 论文卡的入围数量取决于检索式宽度和 `max_results`（默认 60；300–500 更稳，3000 会让 arXiv 返回 500）；
- `state.json` 里的 `seen_ids` 会记住已推送论文，重复跑同一方向可能没有新论文；
- 观察型卡一张一个文件，一轮 6 条候选大约多 10–12 步；
- 通知与桌面集成只在 Windows 完整验证；Linux/macOS 可用但未做完整测试。

## 参与

- 想反馈使用体验：`python platform\collect_feedback.py --config config.json --test <节点> --result pass|fail|unsure --note "..."`，
  会在 `feedback/` 生成一个可上传的反馈包。

## 许可

[MIT](LICENSE)

# 用户手册

## 

## 网页交互：每个功能的操作清单

> 通用规则：所有网页操作都是在 **SciencePal → 智能体 → 灵感官 → 新建会话** 里进行。右侧的“计算机”面板会实时显示它执行的命令、终端输出和退出码，这是判断“它真的做了”的直接证据，不要只看它在对话里怎么说。

### F1 建立画像（第一次必做，约 5 分钟）

| 步   | 你做什么                                                     | 你应该看到什么                                               |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | 确认评价页服务开着（收货员内置在里面）                       | 评价页状态行正常                                             |
| 2    | 网页里说：**“建立画像”**                                     | 计算机面板出现 `load_skill insp-profile`，它先复述 3 条硬纪律 |
| 3    | 它**用一条消息**问 7 个问题（身份与项目、关注方向、擅长 + 例子、不熟领域、表达偏好、arXiv 分类与关键词、不要假设我会） | 不会弹表单；如果弹了表单，说明纪律被违反，记下来反馈         |
| 4    | 简短回答。擅长的东西**要举例**，否则会被降级为“了解”；不确定分类就说“你帮我定” | 缺必填项时，它最多再追问一轮                                 |
| 5    | 它给出画像**预览**                                           | 这一步还没写任何文件                                         |
| 6    | 回复 **“确认”**（要改就直接说改哪里）                        | 它运行 `build_profile.py`，回答最后附上 `<<<INSP_PROFILE_JSON … INSP_PROFILE_JSON>>>` |
| 7    | 等 1 分钟左右                                                | Windows 通知“用户画像已更新”；本地出现 `inspiration\profile\AGENT.md` 和 `profile.json`，`state.json` 的检索式也已更新 |
| 8    | 检查记忆页（可选）                                           | 有若干条以 `[灵感画像]`、`[灵感配置]` 开头的记忆             |

**没收到？** 点评价页的 **检查新结果**，或者运行 `python platform\receiver.py --config config.json --thread <会话链接>`。

### F2 更新画像

在网页里说 **“更新画像”**（或“我换方向了”）→ 它先用 3–5 行复述现有画像，问你哪些要改 → 只改你提到的部分 → 预览 → “确认”。之后的流程和 F1 第 6–8 步相同。

### F3 跑一次灵感（核心功能，摘要模式 5–15 分钟）

| 步   | 你做什么                                                     | 你应该看到什么（计算机面板 / 对话）                          |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | 网页里说：**“主动探索灵感”**。可以加方向，比如“今天看 surface code 解码器”；也可以临时指定模式：“…，**通读全文**”或“…，**只看摘要**” | `load_skill insp-nightly`，然后复述硬纪律                    |
| 2    | 等它准备数据                                                 | `python3 /workspace/skills/insp-nightly/scripts/insp_run.py prepare …`，输出 `MODE persisted/bootstrap/attached`，以及 `raw / seen / new / shortlisted` 计数 |
| 3    | （只在全文模式）等它抓原文                                   | 出现 `FIND_QUOTES python3 /workspace/fq <ID>`，之后它用 `python3 /workspace/fq <ID>` 找引文。最多读 4 篇 |
| 4    | 等它写卡                                                     | `load_skill insp-card`，写 `/workspace/inspiration_out/cards.json` |
| 5    | 等它交付                                                     | 运行 `insp_run.py finalize`；回答最后**原样**附上 `<<<INSP_BUNDLE_JSON … INSP_BUNDLE_JSON>>>`，然后直接结束（**不应**再调用 `complete` 交付工具） |
| 6    | 看评价页                                                     | 1 分钟内弹出 Windows 通知，评价页顶部显示“新到 N 张卡片 · 点击载入” |

**也可以不进网页**：直接点评价页顶部的 **主动探索灵感**。按钮会自动带上已见 ID 和当前的读取模式。

**结果状态**：

| 状态                           | 意思                                | 你要做什么                                                   |
| ------------------------------ | ----------------------------------- | ------------------------------------------------------------ |
| `OK`                           | 有新卡片                            | 去评价                                                       |
| `NO_CANDIDATES`                | 没有新的、值得出卡的论文            | 正常结果。可以换个方向再说一次                               |
| `FETCH_BLOCKED`                | 沙箱连不上 arXiv                    | 不用管：收货员会自动改走本地抓取（日志在 `runs\fallback.log`） |
| `STATE_MISSING`                | 状态文件损坏                        | 运行 `python insp-init\scripts\validate_insp_state.py inspiration` |
| `VALIDATION_FAILED` / 收货被拒 | 整包没通过本地校验，或混入了测试 ID | 看评价页状态栏和 `inspiration\inbox\receiver_status.json`；这时**状态不会被修改** |

**运行中要留意的异常**

**网页显示“任务已中断”，但卡片已经收到了？**（
先看评价页状态栏：如果写着“最近结果 HH:MM 已入库（N 张）”，说明结果早就收到了。“任务已中断”是平台在收尾阶段的交付校验告警（典型报错 `DELIVERY_VALIDATION_LIMIT`），不影响已入库的卡片。**不要点“从断点继续”**：它只会让灵感官再跑一轮，新结果会被去重挡住，白花时间和 token。1.4.5 起技能和系统提示词都要求它“输出结果块后直接结束、不调用 complete”，这种情况应该很少再出现；如果还出现，请用 §11 的方式反馈，并附上会话链接。

### F4 在评价页评价卡片

| 按键 | 动作                        | 结果                                                         |
| ---- | --------------------------- | ------------------------------------------------------------ |
| 空格 | 翻面看细节                  | 只记隐式信号（`implicit.jsonl`），**不影响权重**             |
| 1    | 采纳                        | +1.0                                                         |
| 2    | 否决                        | −1.0，来源论文进负面清单；可以填“屏蔽主题”                   |
| 3    | 已做过                      | −1.0，进负面清单                                             |
| 4    | 不重要                      | −0.5                                                         |
| 5    | 追问                        | +0.5，卡片仍然算“待评价”；本地观察卡会把证据文件的打码副本一起发过去；没完成时回执里有“重试追问” |
| 6    | 立刻动手                    | +1.0；先弹出动手单（场景 / 参数 / 方法 / 产出 / 对照 / 判据 / 交付），点“确认开跑”才建任务 |
| E    | 改写卡片（Ctrl+Enter 保存） | +0.8，自动保存改动前后的差异                                 |
| Z    | 撤销最后一条评价            | 卡片恢复成上一条评价的状态，负面清单也一并恢复；撤销“立刻动手”会取消还没开始的任务 |
| →    | 跳过                        | 只记隐式信号                                                 |

**卡片标签（v1.5.1）**：来自你本地文件的卡片标“**本地观察 · 检测器**”（长函数 / 重复代码 / 改动热点 / 性能嫌疑 / 代码小问题 / 日志 / 数据反常 / 笔记 / 使用统计）；标“补位”的是本轮没有达标的 A 类、按“每轮至少 1 张 A 类”补上来的；“有动手单”= 灵感官已经写好了动手单。

**历史（v1.5.1）**：右上角“历史”→ 已评价列表（按评价时间倒序）。**每一条都能点开**，和评价时一样显示卡片：

| 按键                 | 动作                                                         |
| -------------------- | ------------------------------------------------------------ |
| 空格 / 点卡片        | 翻面                                                         |
| ← / →                | 上一张 / 下一张                                              |
| E（Ctrl+Enter 保存） | 改写，和评价时一样记一条“改写”（+0.8）；**分类不变**，列表显示“采纳 · 已改写” |
| Esc                  | 返回列表；列表里再按 Esc 回到评价                            |

历史详情里没有 1–6 分类按钮；想改分类，只能在当场评价时按 Z 撤销后重评。

**另外两种评价方式**：

- **Excel**：打开 `inspiration\inbox\verdict_inbox.csv`，在 `action` 列填“采纳 / 否决 …”，保存后运行 `python insp-feedback\scripts\record_verdict.py --state-dir inspiration --batch inspiration\inbox\verdict_inbox.csv --config config.json`；
- **在对话里**：对灵感官说“采纳 ins_xxx，理由……”→ 它输出 `INSP_VERDICTS_JSONL` 结果块 → 把它的回复存成 txt → 运行 `python platform\run_local.py --config config.json --import-verdicts 回复.txt`。**在网页里说了不等于记下了**，导入之后才生效。

### F5 周报与权重（每周一次，在本地运行）

> 这一版的周报**只在本地跑**：评价日志 `verdicts.jsonl` 只存在你的电脑上，不会推到沙箱里，所以在网页里让灵感官做复盘，它拿不到数据。

| 步   | 命令                                                         | 结果                                                    |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------- |
| 1    | `python platform\run_weekly.py --config config.json`         | `inspiration\reports\weekly_*.md` 和 `proposals_*.json` |
| 2    | 读周报：采纳率（健康区间 20%–60%）、两周重复率（≈0）、各类型 p̂、**本周唯一一条**权重调整提议 | 样本数 n<5 的只记录、不提议                             |
| 3    | 同意就运行 `python insp-review\scripts\weekly_review.py --state-dir inspiration --apply inspiration\reports\proposals_XXX.json --confirm --config config.json` | `taste\weights.json` 的版本号 +1                        |
| 4    | 后悔了就运行 `… weekly_review.py --state-dir inspiration --rollback` | 回到上一个版本                                          |

### F6 立刻动手（start_now）

1. 在评价页按 **6**。v1.5.1 起**按下去就建任务**（K1 已修，评价页需要用 `review_server.py --config config.json --open` 启动）：
   - 系统先把卡片整理成一份 **动手单**：场景 / 参数（数值 + 单位 + 来源：资料/文献、本地观察/实测、你提供）/ 方法 / 产出 / 对照 / 判据 / 交付；灵感官写好的就直接用，否则按卡片自动生成；
   - 动手单还缺“你提供”的参数时，会弹出动手单让你填，填完点“确认开跑”（Ctrl+Enter）；点“稍后再填”的话，它会留在“回执 → 待补动手单”里；
   - 默认（`actions.start_now_mode` = `"form"`，旧值 `"auto"` / `"confirm"` 同样处理）：按 6 先弹出预填好的动手单，核对或修改后点“确认开跑”才建任务；想动手单完整就直接开跑：`"direct"`；只想记评价不建任务：`"off"`；
   - 观察型卡片（例如 A 类代码优化）会把**卡片证据所在的本地文件**（脱敏后，≤ 2 MB）附给任务，这样沙箱里才能真的跑新旧两版；`observe.upload_code_snippets=false` 或 `actions.attach_local_files=false` 时不附；
2. 右上角“**回执**”（数字 = 等你处理的条数）：每个任务的分档、状态、会话链接、结论、动手单的“产出实测 / 对照实测 / 判据是否满足”、编排器检查（包括“动手单的交付物都在”）；T2 在这里点“批准执行”，跑完点“结果可用 / 需要重跑 / 结果作废”。任务结束时页面会弹提示。命令行的 `run_action.py --sweep` 仍然可以用来补建任务；
3. 系统按风险分档：**T0** 只读整理、**T1** 沙箱验证（这两档自动执行）；**T2** 先出方案，你运行 `run_action.py --approve act_…` 批准后才执行；**T3** 只出方案（涉及设备、湿实验、付款、对外发送），由你亲自执行；
4. 看进度用 `run_action.py --list`；回执在 `inspiration\actions\<job>\receipt.md`；
5. 确认回执：`run_action.py --ack act_… --result accept|rerun|discard --note "…"`（这一步**不影响权重**）。

### F7 记忆管理（网页记忆页）

| 前缀                            | 谁写的                           | 怎么处理                                                     |
| ------------------------------- | -------------------------------- | ------------------------------------------------------------ |
| `[灵感画像]` `[灵感配置]`       | 收货员，按你确认过的画像整体替换 | 不要手改；要改就走 F2                                        |
| `[灵感状态]`                    | 收货员（已推送 ID 的摘要）       | 不要手改                                                     |
| `[灵感设置]`                    | 收货员（论文读取模式）           | 改 `config.json`，然后运行 `receiver.py --sync-settings`     |
| 来源是 `auto_extraction` 的条目 | 平台自动抽取                     | 可能把某次的临时要求固化成了长期偏好。周报第 6 节会列出这些条目，由你决定保留 / 改写 / 删除 |

### F8 切换论文读取模式

`config.json → arxiv_fetch_mode`：`"abstract"`（默认，快）或 `"fulltext"`（通读全文，大约慢 4–5 倍，每次最多 4 篇）。

| 怎么跑         | 什么时候生效                                                 |
| -------------- | ------------------------------------------------------------ |
| 评价页按钮     | 下一次点击                                                   |
| 网页里 @灵感官 | 评价页开着时 1 分钟内自动同步；想立刻生效就运行 `python platform\receiver.py --config config.json --sync-settings` |
| 只临时换一次   | 网页里说“…，通读全文”或“…，只看摘要”                         |

### F9 本地资料库与观察通道（v1.5 新增，约 10 分钟配置）

**它做什么**：在你电脑上扫描你自己的资料，把“发现”写进 `inspiration\observed.jsonl`。下一次跑灵感时，这些发现和 arXiv 新论文一起进入筛选，灵感官据此写卡（多数是 A 类优化建议）。**扫描只在本地运行**，上传到沙箱的只有入围发现的摘要和证据片段（默认每条最多 40 行）。

**第 1 步：建资料库**

```powershell
python platform\observe.py --config config.json init
```

会在包根目录建 `library\` 和五个子文件夹，往里放东西：

| 文件夹           | 放什么                                               | 会发现什么                                                   |
| ---------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| `library\code`   | 你的物理 / 计算代码（py、c/cpp、f90、m、jl、r、sh…） | 过长或过复杂的函数、嵌套太深、重复代码块、循环里的低效写法；如果是 git 仓库，还会找“常改又复杂”的热点 |
| `library\logs`   | 运行日志、输出文件                                   | 程序崩溃（按最后的异常行归并）、NaN / Inf、不收敛、数值序列里的离群点 |
| `library\data`   | csv / tsv / dat / npy 等数据                         | 缺失值、常数列、离群值                                       |
| `library\notes`  | 科研笔记（md / txt / org）                           | 你写过的“有点怪 / 反常”“以后可以试试”“没搞懂”“待查”          |
| `library\papers` | 收集的论文（pdf / bib / 文献清单）                   | 你的高频主题；和当前 arXiv 关键词的缺口（可一键并入关键词）  |

**大的代码仓库不用复制**，在 `config.json → observe.roots` 里直接写原路径（`kind` 取 code / logs / data / notes / papers / mixed）：

```json
"observe": {
  "roots": [
    {"path": "library", "kind": "mixed", "label": "lib"},
    {"path": "D:\\research\\qmc-code", "kind": "code", "label": "qmc"}
  ]
}
```

没有 `observe` 这一段 = 不开观察通道，旧用户不受影响。`config.example.json` 里有带说明的完整版本（阈值、忽略目录 `exclude_dirs` / `exclude_globs`、文件数上限 `max_files`、时间上限 `time_budget_s` 等）。

**第 2 步：扫描并预览**

```powershell
python platform\observe.py --config config.json scan      # 第一次会“建立基线”
type inspiration\library\summary.md                         # 先看清单：扫了什么、发现了什么
python platform\observe.py --config config.json list      # 下次运行会入围哪几条（规则与沙箱完全一样）
python platform\observe.py --config config.json show obs_xxxxxxxxxxxx   # 看某一条的完整证据
```

**第 3 步：照常跑灵感**。评价页按钮或定时任务触发时，收货员会**先自动扫描一次**再上传（`observe.auto_scan_before_run`，默认开）。想只处理观察通道、不抓 arXiv：在网页里说“主动探索灵感，只看本地观察”（对应 prepare `--observed-only`）。

**卡片长什么样**：评价页和每日摘要里标“本地观察 · 文件位置”（如 `library://qmc/vmc.py#L12-L140`）；引文是你自己的代码行 / 日志行 / 笔记原句。评价方式和论文卡一样（1–6、E 改写）。

## 先分清两个「library」

| 路径                   | 是什么                                                       | 谁创建                       | 进不进 Git                      |
| ---------------------- | ------------------------------------------------------------ | ---------------------------- | ------------------------------- |
| `library/`（包根目录） | **你的资料库**：代码 / 笔记 / 论文 / 数据 / 日志，是扫描的**输入** | `observe.py init` 或你手动建 | **不进**（`.gitignore` 已忽略） |
| `inspiration/library/` | **观察结果**：`manifest.json`、`codescan.json`、`summary.md`、`changes.jsonl`、`observed_history.jsonl` | 每次 `scan` 自动写           | 不进（`inspiration/` 整体忽略） |

**下面说的「library」都指前者**——你的资料库。

---

## 1. library 是怎么被创建出来的

它**不在发布包里**，因为里面是你的私人数据。创建方式有两种，效果一样：

### 方式 A：一条命令（推荐）

```powershell
copy platform\config.example.json config.json          # 如果还没有 config.json
python platform\observe.py --config config.json init
```

它会：

1. 按 `config.json → observe.roots` 逐个检查目录；
2. **只对包目录内**（`./library/code` 这类相对路径）的缺失目录执行 `mkdir -p`；
3. 写一个 `library\README.txt`（说明每个子目录放什么）；
4. 如果 `config.json` 里还没有 `observe` 段，把示例里的整段写进去（原文件先备份成 `config.json.bak-<时间戳>`）；
5. 打印每个根的 `新建 / 存在 / 缺失`：

```text
  新建 code     code    C:\...\inspiration-skills\library\code
  新建 notes    notes   C:\...\inspiration-skills\library\notes
  新建 papers   papers  C:\...\inspiration-skills\library\papers
  新建 data     data    C:\...\inspiration-skills\library\data
  新建 logs     logs    C:\...\inspiration-skills\library\logs
```

### 方式 B：手动建

```powershell
mkdir library\code, library\notes, library\papers, library\data, library\logs
```

### 创建后目录长这样

```text
inspiration-skills\
├── config.json
├── library\
│   ├── README.txt
│   ├── code\       ← 你的代码（含 git 仓库时，会读 git log 算改动热点）
│   ├── notes\      ← .md .txt .org .tex 笔记
│   ├── papers\     ← .pdf .bib .ris 与阅读笔记
│   ├── data\       ← .csv .tsv .dat（只看最近 N 天改过、5 MB 以内的）
│   └── logs\       ← .log .out .err .txt
└── inspiration\    ← 运行时状态（不是资料库）
```

> 空目录完全正常：扫描器会跳过没有文件的根，不会报错。

---

## 2. 一个扫描周期发生了什么

```text
observe.py scan
  ├─ 遍历 observe.roots（每个根有 label / kind / 是否存在）
  ├─ library.py    → 文件清单 + 变化（新增/修改/删除）→ inspiration/library/{manifest.json,changes.jsonl,summary.md}
  ├─ codescan.py   → 代码问题（长函数 / 参数多 / 嵌套深 / 静默吞错 / 重复 / 改动热点）→ codescan.json
  ├─ anomaly.py    → 日志与数据异常（NaN、异常退出、指标突变）
  ├─ notes/papers/usage → 笔记、论文、使用习惯
  └─ observed.py   → 合并成 inspiration/observed.jsonl（**只写发现，不写全文**）
```

运行时（`auto_scan_before_run: true`）会在每次「主动探索灵感」前自动扫一次；
也可以手动：

```powershell
python platform\observe.py --config config.json scan
python platform\observe.py --config config.json list        # 预览入围的观察
python platform\observe.py --config config.json scan --all  # 看未入围的原因
python platform\observe.py --config config.json show <id>   # 看单条详情
```

`scan` 成功时会打印：

```text
observed.jsonl 12 条 → 入围 6 条（规则：state.json → observed，与沙箱 prepare 相同）
  ✔ [optimization] 严重度 2.9 · power_dep.py 的循环里反复读文件  (obs_xxx · library://code/power_dep.py#L520)
```

---

## 3. 接入你已有的仓库（重点）

**当仓库内容很庞大时不需要把仓库复制进 `library/`。** 直接把 `observe.roots` 指向它。

### 3.1 改哪里

`config.json` 里的 `observe.roots` 数组，每一项三个字段：

| 字段    | 必填 | 说明                                                         |
| ------- | ---- | ------------------------------------------------------------ |
| `path`  | ✅    | 绝对路径（如`D:/research/qmc`）或相对 `config.json` 的路径（`./library/code`） |
| `kind`  | 建议 | `code` / `notes` / `papers` / `data` / `logs` / `mixed`，只影响 `.txt`、`.pdf` 这类歧义扩展名归到哪一类 |
| `label` | 建议 | 证据里显示成 `library://<label>/<相对路径>`；不写则用目录名。**同名会自动加数字**（`qmc2`） |

> `path` 支持 `%USERPROFILE%`、`~` 环境变量/家目录展开；相对路径按 **`config.json` 所在目录** 解析。
> 不存在的路径不会报错，只会在 `init` 里显示「缺失」、扫描时跳过。

### 3.2 示例：把一个 git 仓库接进来

```json
{
  "observe": {
    "enabled": true,
    "auto_scan_before_run": true,
    "roots": [
      { "path": "D:/research/qmc",        "kind": "code",  "label": "qmc" },
      { "path": "D:/research/paper-notes","kind": "notes", "label": "notes" },
      { "path": "./library/papers",       "kind": "papers" },
      { "path": "./library/data",         "kind": "data" },
      { "path": "./library/logs",         "kind": "logs" }
    ]
  }
}
```

要点：

- 原来的 `./library/code` 那一项**可以整条删掉**，换成你的仓库；
- 想同时观察多个仓库，就写多项（`label` 不同）；
- **用正斜杠 `/`**（Windows 也认），避免 JSON 里 `\` 转义踩坑；
- 路径含空格可以正常写（`"D:/My Research/qmc"`）。

### 3.3 改完怎么验证

```powershell
python platform\observe.py --config config.json init     # 看 存在/缺失
python platform\observe.py --config config.json scan     # 真的扫一遍
```

`init` 输出里你的仓库应显示 **`存在`**；`scan` 后看 `inspiration/observed.jsonl` 是否有新条目，
或 `python platform\observe.py --config config.json list`。

### 3.4 为什么证据里看不到我的绝对路径

`observed.jsonl` 里的 `evidence_ref` 是 `library://<label>/<相对路径>#L<行号>`，
例如 `library://qmc/src/solver.py#L145`。绝对路径只留在本机 `inspiration/library/*.json`。

### 3.5 不想改 JSON 的替代做法

| 做法                                           | 命令                                           | 注意                                                         |
| ---------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 在 `library/code` 下建**目录联接**（junction） | `mklink /J library\code\qmc D:\research\qmc`   | 扫描器会跟进去；但删除 junction 时别用 `rmdir /s` 误删源目录 |
| 直接把仓库复制进 `library/code`                | `xcopy /E /I D:\research\qmc library\code\qmc` | 占空间、容易不同步，不推荐                                   |
| 软链接（symlink）                              | `mklink /D ...`                                | Windows 需要开发者模式或管理员权限                           |

**推荐还是改 `observe.roots`**：零拷贝、路径不进证据、可随时换。

---

## 4. 常用调参

| 配置项                                                       | 默认                                                         | 什么时候改                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `exclude_dirs`                                               | 已内置 `.git`、`node_modules`、`__pycache__`、`venv`、`build`、`dist` 等 | 你的仓库有大目录（`data/raw`、`third_party`）时加上          |
| `exclude_globs`                                              | `*.min.js`、`*.lock`、`*secret*`、`*.key`、`*.pem`、`.env` 等 | 想排除 `docs/**`、`*.bak`、`*.ipynb` 时加                    |
| `max_files`                                                  | 20000                                                        | 超大仓库扫太慢时降到 2000–5000                               |
| `lookback_days`                                              | 30                                                           | 只看最近两周就改 14                                          |
| `max_items`                                                  | 80                                                           | 观察太多、卡片被稀释时降到 30–40                             |
| `time_budget_s`                                              | 180                                                          | 扫描超时；想更快降到 60                                      |
| `upload_code_snippets`                                       | `true`                                                       | 改成 `false`：只上传指标（行数/复杂度/位置），不上传代码原文 |
| `snippet_lines`                                              | 40                                                           | 代码片段最多几行                                             |
| `code.long_function_lines` / `complexity` / `nesting` / `max_args` | 80 / 15 / 5 / 7                                              | 觉得太吵就调高阈值                                           |
| `code.churn_days`                                            | 90                                                           | git 改动热点的回看天数                                       |

---

## 5. 隐私边界

- **原始文件不出本机**；上传到沙箱的是：
  - `state.json` / `observed.jsonl` / 画像 / 口味规则（快照）；
  - 观察证据里的**脱敏片段**（`upload_code_snippets=true` 时，最多 `snippet_lines` 行）；
- **密钥、令牌、邮箱、个人目录会自动脱敏**；
- 想更保守：`"upload_code_snippets": false`，灵感官仍能提"这个函数太长/这里吞了异常"，
  但看不到具体代码行，提出的改法会更笼统；
- `exclude_globs` 再加一层：`["*secret*", "*token*", "*.env", "config*.json"]`。

---

## 6. 常见问题

| 现象                               | 原因                                              | 处理                                                 |
| ---------------------------------- | ------------------------------------------------- | ---------------------------------------------------- |
| `init` 显示 `缺失`                 | 路径写错，或仓库不在那个位置                      | 用绝对路径重试；确认盘符                             |
| `scan` 后 `observed.jsonl` 是 0 条 | 根目录不存在 / 全被排除 / 阈值太高 / 最近没有改动 | 先 `scan --all` 看原因；再放宽阈值或 `lookback_days` |
| 扫描很久                           | 仓库太大                                          | 调 `max_files`、`time_budget_s`，加 `exclude_dirs`   |
| 卡片里没有代码行                   | `upload_code_snippets=false`                      | 改回 `true`（或接受更笼统的建议）                    |
| 换了目录后旧观察还在               | `observed.jsonl` 是累积的                         | 备份后删除 `inspiration/observed.jsonl`，重新 `scan` |
| `A 类候选：待用 0`                 | 观察型卡片配额用完了（已出卡/已否决）             | 评价页看 `A 类候选` 统计；重置演示数据可清           |
| 不想观察任何东西                   | —                                                 | `"observe": {"enabled": false}` 或删掉整段           |


