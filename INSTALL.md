# 安装与第一次运行

> 适用版本：`1.5.9-test`· 主支持平台：Windows 10/11

---

## 0. 准备清单

| 项 | 要求 | 怎么确认 |
|---|---|---|
| 操作系统 | Windows 10/11（主支持）；Linux/macOS 可用但桌面集成未完整验证 | — |
| Python | **3.9+**，只需标准库；`pypdf` 可选（全文模式 PDF 兜底） | `python --version` |
| 账号 | 能登录 <https://sciencepal.ai> | 浏览器打开 |
| 令牌 | **API Key**（`sp_live_…`）+ **会话令牌**，必须同一账号 | 见第 2 步 |
| 端口 | 评价页默认 `8765` 空闲 | `netstat -ano | findstr 8765` |

> 首次安装建议在**空目录**里解压，避免和旧版本混在一起。

---



---

## 1. 准备两种令牌（必须来自同一个账号）

平台对两种令牌的权限不同，**两种都要放**：

| 令牌 | 能做什么 | 存放位置 |
|---|---|---|
| **API Key**（`sp_live_…`） | 发起运行、读运行事件流、读写沙箱文件 | `%USERPROFILE%\.sciencepal\run_token` |
| **会话令牌**（网页登录后的 Bearer） | 导入技能、建智能体、读会话文件（收货）、列会话、读写记忆 | `%USERPROFILE%\.sciencepal\session_token` |

```powershell
mkdir $env:USERPROFILE\.sciencepal -Force
notepad $env:USERPROFILE\.sciencepal\run_token       # 粘贴 sp_live_… 后保存
notepad $env:USERPROFILE\.sciencepal\session_token   # 粘贴会话令牌后保存
```

**成功判断**：`python platform\setup_platform.py --config config.json --check` 不报 401/403。
**失败**：`401` 多半是令牌过期或不是同一账号；重新从浏览器取一次。

---

真实平台上两种令牌的权限不同（测评时实测过），**两种都要放**：

| 令牌                                | 能做什么                                                     | 不能做什么                                             | 存放位置                                  |
| ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------ | ----------------------------------------- |
| **API Key**（`sp_live_…`）          | 发起运行、读运行事件流、读写沙箱文件                         | 读会话文件、列会话、读写记忆、导入技能（返回 401/403） | `%USERPROFILE%\.sciencepal\run_token`     |
| **会话令牌**（网页登录后的 Bearer） | 导入技能、建智能体、读会话文件（收货）、列最近会话、读写记忆、查技能加载字段 | —                                                      | `%USERPROFILE%\.sciencepal\session_token` |

**拿 API Key**：在平台的 API Key / 访问密钥页面生成（对应接口 `/api/access-keys`），复制 `sp_live_` 开头的整串。

**拿会话令牌**（平台没有专门的入口，只能从浏览器里取；以平台实际界面为准）：

1. 登录 sciencepal.ai，按 `F12` 打开开发者工具，切到 **Network（网络）**；
2. 在页面上随便点一下（比如打开智能体列表），找一条以 `/api/` 开头的请求；
3. 在 **Request Headers** 里找到 `Authorization: Bearer xxxxx`，复制 `Bearer ` **后面**那一串。

**存放**（文件里只放令牌本身，不要带引号、空格或 `Bearer`）：

```powershell
mkdir $env:USERPROFILE\.sciencepal -Force
Set-Content -NoNewline $env:USERPROFILE\.sciencepal\run_token     "sp_live_……"
Set-Content -NoNewline $env:USERPROFILE\.sciencepal\session_token "eyJ……"
```

> ⚠ 令牌**不能**出现在包目录、对话、截图、反馈文件里。会话令牌会过期：过期后收货员自动退回“仅 API Key”模式，评价页状态行会显示出来，这时重新取一次即可。

## 2. 生成本地配置与状态

```powershell
copy platform\config.example.json config.json
xcopy /E /I insp-init\assets\灵感状态模板\inspiration inspiration
```

- `config.json` 是你的本地配置（**不要提交到仓库**）；
- `inspiration\` 是运行时状态目录（卡片、评价、画像、快照都在这里）；
- 想改观察范围，编辑 `config.json` 里的 `observe.roots`（code / notes / papers / data / logs）。

---

## 3. 导入技能并建「灵感官」

```powershell
python platform\setup_platform.py --config config.json --replace
python platform\setup_platform.py --config config.json --check
```

**成功**（`--check` 全绿）：

```
PASS 技能全部绑定
PASS API Key 能看到灵感官（同一账号）
PASS 没有残留的灵感定时任务
PASS 会话令牌可读会话列表（收货员能自动发现网页里发起的会话）
安装检查通过
```

**失败**：

| 现象 | 处理 |
|---|---|
| `401/403` | 令牌不对或过期，重取（第 2 步） |
| `agent not found` | 先 `--replace` 再 `--check` |
| 技能绑定不全 | 再跑一次 `--replace`；仍失败看 `platform\setup_platform.py` 的输出 |

---

## 4. 启动本机评价页

任选其一：

```powershell
powershell -ExecutionPolicy Bypass -File platform\windows\start_review_server.ps1
```

或双击 `platform\windows\start_review_server.bat`。

**成功**：浏览器打开 <http://127.0.0.1:8765/> 能看到评价页；控制台无报错。
**失败**：

| 现象 | 处理 |
|---|---|
| 端口被占用 | 改 `config.json` 里 `review.port`，或 `python platform\review_server.py --config config.json --stop` 后再启 |
| 页面空白 | 确认 `inspiration\state.json` 存在（第 3 步没做会缺） |
| 通知不弹 | Windows 设置 → 通知 → 允许应用通知；部分系统需要「专注助手」关闭 |

---

## 5. 第一次跑

1. 打开 <http://127.0.0.1:8765/>；
2. 在 SciencePal 网页里**新开一个对话**，选「灵感官」，说 **“建立画像”**（会问 7 个问题）；
3. 再**新开一个对话**，说 **“主动探索灵感”**，或点评价页的「主动探索灵感」按钮；
4. 等几分钟，卡片出现在评价页并弹通知。

**成功**：评价页出现卡片；点开能看到来源、引文、最小验证、判据、推翻条件。
**失败**：

| 现象 | 原因 | 处理 |
|---|---|---|
| 一轮跑完没有卡 | 论文方向没有新候选（`seen_ids` 已包含） | 换检索式 / 调大 `max_results`（300–500）/ 重置演示数据 |
| 只有观察卡、没有论文卡 | 检索式太宽或与画像不匹配 | 检查 `state.json` 的 `sources[].query`；确认模型没有用 `--query` 覆盖 |
| 会话里说「没有状态」 | `inspiration\state.json` 缺失 | 重跑第 3 步 |
| 卡片重复 | `demo_reset` 后旧 bundle 被重收 | 重置前备份 `inspiration\inbox\collected.jsonl` |

---

## 6. 日常使用

| 想做什么 | 怎么做 |
|---|---|
| 再跑一轮 | 网页里说“主动探索灵感”，或点评价页按钮 |
| 评价卡片 | 评价页点「采纳 / 否决 / 已做过 / 不重要」；按 5 追问，按 6 立刻动手 |
| 看周报 | `python platform\run_weekly.py --config config.json` |
| 立即动手 | 评价页按 6 → 填动手单→ 批准；结果在评价页回执 |
| 换论文读取模式 | `config.json` 里 `arxiv_fetch_mode`：`abstract`（快）/ `fulltext`（读原文） |
| 清空演示数据 | `demo_reset.bat`（备份在 `demo_backups\`，可用 `demo_restore.bat` 还原） |

---

## 7. 升级

```powershell
# 1) 停评价页
python platform\review_server.py --config config.json --stop

# 2) 解压新包，覆盖到旧目录（保留 config.json / platform_ids.json / inspiration / library）

# 3) 重新导入技能（技能有改动时必须）
python platform\setup_platform.py --config config.json --replace
python platform\setup_platform.py --config config.json --check

# 4) 清理清单外的旧文件（会移到 .stale_backup\）
python scripts\selfcheck.py stale --clean

# 5) 自检
python scripts\selfcheck.py g1
```

---

## 8. 卸载 

- **彻底卸载**：停止评价页，删除包目录与 `%USERPROFILE%\.sciencepal\`；
  平台上的技能与「灵感官」需要在 SciencePal 网页里手动删除。

---

## 9. 遇到问题怎么报

```powershell
python platform\collect_feedback.py --config config.json --test <节点> --result fail --note "现象 + 复现步骤"
```

会在 `feedback\` 生成一个 Markdown 包（环境、最近运行、日志摘要），直接发回即可。
请不要附带 `config.json` 或令牌文件。
