---
theme: seriph
background: https://cover.sli.dev
title: Redbook-MCP：小红书抓取与自动评论演示
info: |
  项目演示：展示如何使用 Playwright + FastMCP 实现小红书笔记抓取、分析与自动评论。
  包含核心接口：登录(`login`)、搜索(`search_notes`)、抓取笔记内容(`get_note_content`)、抓取评论(`get_note_comments`)、分析并生成评论(`analyze_note` / `post_smart_comment`)。
class: text-center
comark: true
duration: 20min
---

# 项目概览

本项目实现了一个基于 Playwright 的小红书爬取与交互层，并通过 FastMCP 暴露工具函数供 MCP 客户端调用。主要目标：

- 自动化登录（保留浏览器会话）
- 搜索笔记并提取笔记链接
- 抓取笔记正文及评论
- 基于笔记内容生成智能评论并发布

示例脚本：`test_mcp.py` 用于演示如何调用这些工具函数。

---

# 代码结构（高层）

- `xiaohongshu_mcp.py`：核心实现，包含：
  - `process_url(url)`：标准化和修正 URL
  - `ensure_browser()`：启动 Playwright 浏览器上下文并检测登录
  - `login()`：引导用户在打开的浏览器窗口内登录
  - `search_notes(keywords, limit=5)`：按关键词搜索笔记并返回链接/标题
  - `get_note_content(url)`：抓取并解析笔记正文、作者、发布时间
  - `get_note_comments(url)`：滚动并抓取评论列表
  - `analyze_note(url)`：调用 `get_note_content` 并进行简单关键词/领域分析
  - `post_smart_comment(url, comment_type)` / `post_comment(url, comment)`：生成或直接发布评论

- `test_mcp.py`：演示脚本，调用上述接口做登录、搜索与抓取演示。

---

# 核心函数说明（摘要）

- `login()`：打开 `https://www.xiaohongshu.com` 并点击登录，引导用户手动扫码或输入账号完成登录；使用持久化 `user_data_dir` 保存会话。
- `ensure_browser()`：在首次调用时启动 Chromium 持久化上下文（`headless=False`），创建 `main_page` 并设置超时。
- `search_notes(keywords, limit)`：访问 `https://www.xiaohongshu.com/search_result?keyword=...`，定位帖子卡片并提取链接和标题，返回前 `limit` 条结果。
- `get_note_content(url)`：访问笔记链接，滚动加载、检查错误提示、通过多种 DOM / JS 策略获取标题、作者、发布时间与正文，返回格式化文本。
- `get_note_comments(url)`：滚动至评论区，逐步加载更多评论，并使用多套选择器提取评论文本。
- `analyze_note(url)`：将抓取到的内容进行分词与领域关键词匹配，返回结构化分析结果（用于评论生成）。
- `post_comment(url, comment)`：定位评论输入框并发送评论（尝试点击发送按钮、回车或 JS 点击三种方式）。

---

# 使用与运行（PowerShell）

建议在项目根目录下执行：

```powershell
# 创建并激活虚拟环境（如尚未）
python -m venv .\venv
& .\venv\Scripts\Activate.ps1

# 安装依赖
python -m pip install -r requirements.txt

# 安装 Playwright 浏览器（仅需首次）
playwright install

# 运行演示脚本（示例：抓取单个笔记或搜索链接）
python .\test_mcp.py "https://www.xiaohongshu.com/...."
```

注意：`playwright install` 可能需要网络并下载浏览器二进制。

---

# 登录与会话注意事项

- 使用持久化用户数据目录 `browser_data` 保存登录状态，便于后续调用无需重复登录。
- 浏览器以 `headless=False` 启动以便手动扫码登录；生产环境可考虑安全地管理 Cookie/Token 并切换为无头模式。
- 请勿在公共环境下泄露 `browser_data` 或持久化目录中的敏感信息。

---

# 自动化抓取策略与鲁棒性

- 多种选择器与 JS 提取：代码为不同页面结构准备了多条备选策略（CSS / XPath / JS 长文本提取）。
- 滚动与延时：脚本在关键点使用滚动 + sleep 来确保动态加载内容到 DOM。
- 错误检测：通过页面文本检查常见错误提示（如“内容不存在”）以避免误抓取。

---

# MCP 集成与工作流

- 文件顶部通过 `mcp = FastMCP("xiaohongshu_scraper")` 初始化 MCP 服务。
- 使用 `@mcp.tool()` 装饰器导出工具函数，MCP 客户端（例如 Claude for Desktop）可通过 stdio/transport 调用这些方法。
- 典型流程：客户端请求 `login` → `search_notes` → `get_note_content`/`get_note_comments` → `post_smart_comment`。

---

# 风险与合规提醒

- 请遵守目标站点的使用条款与反爬策略；在未经允许的情况下批量抓取或自动发布可能违反平台政策。
- 对自动发布功能（`post_comment` / `post_smart_comment`）要格外谨慎，避免垃圾评论或滥用。

---

# 快速参考：常用命令

- 激活虚拟环境（PowerShell）: `& .\venv\Scripts\Activate.ps1`
- 安装依赖：`python -m pip install -r requirements.txt`
- Playwright 浏览器安装：`playwright install`
- 运行演示：`python .\test_mcp.py <笔记URL或search_result链接>`

---

class: px-20

<PoweredBySlidev mt-10 />
