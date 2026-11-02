# 灵感官 · Beta 用户测试说明（1.5.9-test）

> 这是一份**用户测试版**（beta）：功能能完整跑通，但还没大范围用过。

## 1. 这是什么

在 SciencePal 网页里对「灵感官」说一句 **“主动探索灵感”**，它会：

1. 去 arXiv 抓你关注方向上的新论文（去重 → 读摘要 → 打分 → 入围）；
2. 读你电脑上的资料库（代码 / 笔记 / 论文 / 数据 / 日志），找出值得做的事；
3. 把结果写成结构化「灵感卡片」，送到你电脑上的评价页；
4. 你对卡片的每一次评价（采纳 / 否决 / 已做过 / 不重要 / 追问 / 立刻动手）都会被记录，
   周报会**提议**调整权重，你确认后才生效。

它**不替你执行**任何未经批准的任务；所有卡片都只是「申请」。

## 2. 你需要准备什么

| 项 | 要求 |
|---|---|
| 系统 | Windows 10/11（主支持）；Linux / macOS 可用，但通知与桌面集成只在 Windows 完整验证 |
| Python | **3.9+**，只需标准库（`pypdf` 可选，用于全文模式 PDF 兜底） |
| 账号 | 能登录 [sciencepal.ai](https://sciencepal.ai) 的账号 |
| 令牌 | **API Key**（`sp_live_…`）和**会话令牌**，必须来自同一个账号；放进 `%USERPROFILE%\.sciencepal\`（见 `useguide.md` 第 2 节） |

## 3. 安装（2 步）

```powershell
# 1) 生成本地配置并初始化状态
copy platform\config.example.json config.json
xcopy /E /I insp-init\assets\灵感状态模板\inspiration inspiration

# 2) 导入技能、建灵感官（需要会话令牌）
python platform\setup_platform.py --config config.json --replace
python platform\setup_platform.py --config config.json --check
```

完整步骤、每一步「看到什么算成功 / 不成功怎么办」，见 **`useguide.md`**。

## 4. 第一次跑

1. 启动评价页（任选其一）：
   ```powershell
   powershell -ExecutionPolicy Bypass -File platform\windows\start_review_server.ps1
   ```
   或双击 `platform\windows\start_review_server.bat`；
2. 浏览器打开 <http://127.0.0.1:8765/>；
3. 在 SciencePal 网页里新开一个对话，选「灵感官」，说 **“建立画像”**（7 个问题）；
4. 再新开一个对话，说 **“主动探索灵感”**，或点评价页的「主动探索灵感」按钮；
5. 几分钟后卡片出现在评价页，同时弹出 Windows 通知。

## 5. 怎么反馈

- **推荐**：在包目录运行
  ```powershell
  python platform\collect_feedback.py --config config.json --test <节点号> --result pass|fail|unsure --note "现象 + 复现步骤"
  ```
  会在 `feedback/` 生成一个 Markdown 包（含环境、最近运行、日志摘要），把它发回即可；
- 或在 GitHub Issue 里贴：版本号（`VERSION`）、复现步骤、期望 / 实际、`release_evidence/` 里最近一次 g1 证据。

## 6. 不要做

- 不要把 `config.json`、`platform_ids.json`、`%USERPROFILE%\.sciencepal\` 里的令牌发给别人；
- 不要把敏感数据放进 `library/`——本地观察只会把**脱敏片段**上传到沙箱，但请自行确认内容。

## 7. 版本与许可

- 版本：`1.5.9-test`（见 `VERSION`）
- 许可：见仓库根目录 `LICENSE`
