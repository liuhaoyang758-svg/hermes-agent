# Agent-Reach 完整使用手册

> **项目地址**: https://github.com/Panniantong/Agent-Reach  
> **定位**: 给 AI Agent 装上"互联网眼睛"，打通 13+ 个平台的访问壁垒，无需付费 API  
> **已安装环境**: Windows / PowerShell / Node v24.16.0 / Python 3.14

---

## 目录

1. [项目概览](#1-项目概览)
2. [已安装工具清单](#2-已安装工具清单)
3. [平台支持总览](#3-平台支持总览)
4. [零配置平台（开箱即用）](#4-零配置平台开箱即用)
5. [需登录配置的平台](#5-需登录配置的平台)
6. [PowerShell 使用命令速查](#6-powershell-使用命令速查)
7. [mcporter 使用规则](#7-mcporter-使用规则)
8. [配置 Cookie 的统一流程](#8-配置-cookie-的统一流程)
9. [环境健康检查](#9-环境健康检查)
10. [实战使用示例](#10-实战使用示例)
11. [常见问题排查](#11-常见问题排查)
12. [更新维护](#12-更新维护)

---

## 1. 项目概览

Agent-Reach 是一个**脚手架工具**，不是框架。它做的事：

- 帮你选好每个平台对应的最佳 CLI 工具
- 自动安装依赖、配置认证
- 提供统一的健康检查命令
- Agent 直接调用底层工具（yt-dlp、twitter-cli、bili-cli 等），不经过 Agent-Reach 包装层

**核心设计哲学**：
- 某个渠道挂了 → 自动切换备用后端
- Cookie 只存在本地 → 不上传、不外传
- 所有工具开源 → 代码完全可审查

---

## 2. 已安装工具清单

| 工具 | 版本/位置 | 用途 |
|------|----------|------|
| `agent-reach` | `C:\Users\Administrator\.agent-reach-venv\Scripts\` | 主程序、健康检查 |
| `mcporter` | `C:\...\fnm\node-versions\v24.16.0\installation\` | MCP 服务运行器 |
| `yt-dlp` | Python Scripts | YouTube 字幕提取 |
| `bili-cli` | pipx | B站搜索、热门、视频详情 |
| `opencli` | npm global | 小红书、Reddit、Twitter 桌面访问 |
| `feedparser` | Python | RSS 订阅解析 |
| `Node.js` | v24.16.0 (via fnm) | mcporter 运行环境 |
| `Python` | 3.14.3 | agent-reach 运行环境 |

---

## 3. 平台支持总览

| 平台 | 状态 | 底层工具 | 备注 |
|------|------|---------|------|
| 🌐 任意网页 | ✅ 零配置 | Jina Reader | 读取任意 URL 转 Markdown |
| 🔍 全网语义搜索 | ✅ 零配置 | Exa via mcporter | AI 搜索引擎，免费无需 Key |
| 📺 YouTube | ✅ 零配置 | yt-dlp | 字幕提取、视频信息 |
| 📺 B站 | ✅ 零配置 | bili-cli | 搜索、热门、排行、视频详情 |
| 💻 V2EX | ✅ 零配置 | 公开 API | 热门帖子、节点、详情 |
| 📡 RSS | ✅ 零配置 | feedparser | 任意 RSS/Atom 源 |
| 💬 微信公众号 | ✅ 零配置 | Exa + 阅读 | 搜索+全文阅读 |
| 📦 GitHub | ⚙️ 需登录 | gh CLI | 需 `gh auth login` |
| 🐦 Twitter/X | ⚙️ 需 Cookie | twitter-cli / opencli | 搜推文、时间线 |
| 📕 小红书 | ⚙️ 需 Cookie | opencli / xhs-cli | 搜索、读笔记、评论 |
| 📖 Reddit | ⚙️ 需登录 | opencli / rdt-cli | 搜帖子、读评论 |
| 🎵 抖音 | ⚙️ 需配置 | douyin-mcp-server | 视频解析、无水印下载 |
| 🎙️ 小宇宙播客 | ⚙️ 需 Groq Key | Whisper 转录 | 播客音频转文字 |
| 💼 LinkedIn | ⚙️ 需配置 | linkedin-mcp | Profile、职位搜索 |
| 📈 雪球 | ⚙️ 需 Cookie | rookiepy | 股票行情、热门帖子 |
| 📰 微博 | ⚙️ 需配置 | mcporter MCP | 热搜、搜索、用户动态 |

---

## 4. 零配置平台（开箱即用）

### 4.1 全网语义搜索（Exa）

```powershell
# 基础搜索
mcporter call exa.web_search_exa query="你的搜索词"

# 指定结果数量
mcporter call exa.web_search_exa query="TikTok玩具热卖" numResults=10

# 限定在特定网站搜索
mcporter call exa.web_search_exa query="玩具热卖榜" includeDomains="tt123.com,chwang.com"

# 代码相关搜索
mcporter call exa.get_code_context_exa query="Python async await用法" tokensNum=3000
```

> ⚠️ **PowerShell 注意**：搜索词里有中文冒号（：）会报错，改用 `query="..."` 格式，不要用括号语法。

### 4.2 任意网页阅读（Jina Reader）

```powershell
# 读取任意网页，返回 Markdown
curl "https://r.jina.ai/https://目标网页URL"

# 示例：读取 GitHub 仓库页面
curl "https://r.jina.ai/https://github.com/Panniantong/Agent-Reach"

# 示例：读取新闻文章
curl "https://r.jina.ai/https://www.36kr.com/p/某篇文章"
```

> ⚠️ **限制**：需要登录或 JS 动态渲染的页面（如 echotik.live 后台数据）无法读取。

### 4.3 YouTube 字幕提取

```powershell
# 提取字幕并保存到临时目录
yt-dlp --write-sub --skip-download -o "C:/tmp/%(id)s" "https://www.youtube.com/watch?v=视频ID"

# 查看视频信息（标题、描述、时长等）
yt-dlp --dump-json "https://www.youtube.com/watch?v=视频ID"

# 搜索 YouTube 视频（返回链接）
yt-dlp "ytsearch5:搜索词" --get-title --get-id
```

### 4.4 B站（bili-cli）

```powershell
# 搜索视频
bili search "AI教程" --type video -n 5

# 搜索用户
bili search "技术博主" --type user -n 5

# 获取热门视频
bili hot -n 10

# 获取排行榜
bili rank

# 获取视频详情（BV号）
bili video BV1xx411x7xx

# 获取视频字幕
bili subtitle BV1xx411x7xx
```

### 4.5 V2EX 社区

```powershell
# 热门帖子
curl -s "https://www.v2ex.com/api/topics/hot.json" -H "User-Agent: agent-reach/1.0"

# 特定节点帖子（python/tech/jobs/qna/programmers 等）
curl -s "https://www.v2ex.com/api/topics/show.json?node_name=python&page=1" -H "User-Agent: agent-reach/1.0"

# 帖子详情（ID 从 URL 获取）
curl -s "https://www.v2ex.com/api/topics/show.json?id=帖子ID" -H "User-Agent: agent-reach/1.0"

# 帖子回复
curl -s "https://www.v2ex.com/api/replies/show.json?topic_id=帖子ID&page=1" -H "User-Agent: agent-reach/1.0"

# 用户信息
curl -s "https://www.v2ex.com/api/members/show.json?username=用户名" -H "User-Agent: agent-reach/1.0"
```

### 4.6 RSS 订阅

```powershell
# 读取 RSS 源（PowerShell 内联 Python）
python -c "import feedparser; [print(e.title, '-', e.link) for e in feedparser.parse('RSS地址').entries[:5]]"

# 常用 RSS 源示例
# 36氪：https://36kr.com/feed
# 少数派：https://sspai.com/feed
# 人民日报：http://www.people.com.cn/rss/politics.xml
# 新华社：http://www.xinhuanet.com/rss/newsworld.xml
```

### 4.7 微信公众号

```powershell
# 通过 Exa 搜索公众号文章（无需配置）
mcporter call exa.web_search_exa query="site:mp.weixin.qq.com AI Agent落地实践"

# 读取具体公众号文章
curl "https://r.jina.ai/https://mp.weixin.qq.com/s/文章ID"
```

---

## 5. 需登录配置的平台

### 5.1 GitHub

```powershell
# 第一步：登录
gh auth login
# 按提示选择 GitHub.com → HTTPS → 浏览器认证

# 验证登录状态
gh auth status

# 搜索仓库
gh search repos "MCP server" --sort stars --limit 10

# 搜索代码
gh search code "agent-reach" --language python

# 查看仓库
gh repo view Panniantong/Agent-Reach

# 查看 Issues
gh issue list -R owner/repo --state open
gh issue view 123 -R owner/repo

# 查看 PR
gh pr list -R owner/repo
gh pr view 123 -R owner/repo

# 查看 Actions
gh run list --repo owner/repo --limit 10
```

### 5.2 Twitter/X

**配置步骤**：

1. Chrome 浏览器登录 https://twitter.com（建议用小号）
2. 安装 Chrome 插件：[Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
3. 在 Twitter 页面点击插件 → Export → Header String
4. 将导出的字符串设置为环境变量：

```powershell
# 在 PowerShell Profile 中添加（持久化）
$env:TWITTER_AUTH_TOKEN = "你的auth_token值"
$env:TWITTER_CT0 = "你的ct0值"
```

**使用命令**：

```powershell
# 首页时间线（最稳定）
twitter feed -n 20

# 搜索推文
twitter search "Claude Code" -n 10

# 读取单条推文
twitter tweet https://twitter.com/xxx/status/123456

# 用户时间线
twitter user-posts @elonmusk -n 20

# 用户资料
twitter user @elonmusk

# 读取长文（X Article）
twitter article https://twitter.com/i/web/article/...
```

> ⚠️ **封号风险**：务必用小号，不要在 VPS/数据中心 IP 上频繁调用。

**search 失败时的重试顺序**：
1. 直接重试：`twitter search "query" -n 10`
2. 升级后重试：`pipx upgrade twitter-cli && twitter search "query" -n 10`
3. 换 OpenCLI：`opencli twitter search "query" -f yaml`
4. 改用稳定命令：`twitter feed` 或 `twitter user-posts @xx`

### 5.3 小红书

小红书有三个后端，优先级：**OpenCLI > xiaohongshu-mcp > xhs-cli**

**查看当前活跃后端**：

```powershell
& "C:\Users\Administrator\.agent-reach-venv\Scripts\agent-reach.exe" doctor --json
# 看 xiaohongshu.active_backend 字段
```

#### 后端 A：OpenCLI（桌面首选）

**前提**：Chrome 已登录小红书 + 安装了 OpenCLI 扩展

```powershell
# 搜索笔记
opencli xiaohongshu search "AI工具" -f yaml

# 读笔记详情（用搜索结果中的完整 URL，含 xsec_token）
opencli xiaohongshu note "https://www.xiaohongshu.com/explore/笔记ID?xsec_token=xxx" -f yaml

# 查看评论
opencli xiaohongshu comments 笔记ID -f yaml

# 首页推荐
opencli xiaohongshu feed -f yaml

# 用户主页
opencli xiaohongshu user 用户ID -f yaml
```

#### 后端 B：xiaohongshu-mcp（服务器场景）

```powershell
# 检查登录状态
mcporter call 'xiaohongshu.check_login_status()' --timeout 120000

# 未登录时获取二维码
mcporter call 'xiaohongshu.get_login_qrcode()' --timeout 120000

# 搜索
mcporter call 'xiaohongshu.search_feeds(keyword: "AI工具")' --timeout 120000

# 笔记详情（feed_id 和 xsec_token 从搜索结果取）
mcporter call 'xiaohongshu.get_feed_detail(feed_id: "...", xsec_token: "...")' --timeout 120000
```

> ⚠️ 首次调用会下载约 150MB 无头浏览器，必须带 `--timeout 120000`

#### 后端 C：xhs-cli（备用）

```powershell
xhs search "AI工具"           # 搜索
xhs read 笔记ID或URL           # 读笔记
xhs comments 笔记ID或URL       # 评论
xhs hot                       # 热门
xhs feed                      # 推荐
```

> ⚠️ xhs-cli 自 2026-03 起停更，不稳定，新用户优先用后端 A/B

**重要限制**：
- 小红书强制 xsec_token 机制，**不能直接用裸 note_id 读取**，必须先搜索，再用结果中的完整 URL
- 高频请求会触发验证码，每次操作间隔 2-3 秒

### 5.4 Reddit

Reddit 没有零配置路径，必须登录态。

**查看当前后端**：
```powershell
& "C:\Users\Administrator\.agent-reach-venv\Scripts\agent-reach.exe" doctor --json
# 看 reddit.active_backend 字段
```

#### 后端 A：OpenCLI（桌面首选）

**前提**：Chrome 已登录 reddit.com + 安装了 OpenCLI 扩展

```powershell
# 搜索帖子
opencli reddit search "AI Agent" -f yaml

# 读帖子全文 + 评论
opencli reddit read 帖子ID -f yaml

# 浏览 subreddit
opencli reddit subreddit LocalLLaMA -f yaml

# 热门帖子
opencli reddit hot -f yaml
opencli reddit popular -f yaml

# subreddit 信息
opencli reddit subreddit-info LocalLLaMA -f yaml
```

#### 后端 B：rdt-cli

```powershell
# 安装（必须从 GitHub 装，PyPI 版落后）
pipx install "git+https://github.com/public-clis/rdt-cli.git"

# 先登录
rdt login

# 搜索帖子
rdt search "Claude Code" --limit 10

# 读帖子
rdt read 帖子ID

# 浏览 subreddit
rdt sub python --limit 20

# 热门
rdt popular --limit 10
rdt all --limit 10
```

> ⚠️ 中国大陆需要代理才能访问 Reddit

### 5.5 抖音

```powershell
# 配置（将 MCP 服务注册到 mcporter）
mcporter config add douyin http://localhost:18070/mcp --system

# 启动抖音 MCP 服务（需要先安装 douyin-mcp-server）
# 使用后调用
mcporter call 'douyin.parse_video(url: "https://v.douyin.com/xxx")'
```

### 5.6 小宇宙播客

```powershell
# 需要 Groq API Key（免费获取：https://console.groq.com）
# 获取后设置环境变量：
$env:GROQ_API_KEY = "gsk_你的key"

# 播客音频转文字
# 先下载音频，再调用 Whisper 转录
```

### 5.7 LinkedIn

```powershell
# 配置（MCP 服务）
mcporter config add linkedin http://localhost:3000/mcp --system

# 使用（需先启动 linkedin-mcp 服务）
mcporter call 'linkedin.search_jobs(query: "AI Engineer", location: "Beijing")'
mcporter call 'linkedin.get_profile(url: "https://www.linkedin.com/in/xxx")'
```

### 5.8 雪球（股票）

```powershell
# 配置（需要雪球账号 Cookie）
# 浏览器登录 xueqiu.com → Cookie-Editor 导出 → 设置环境变量

# 搜索股票
mcporter call 'xueqiu.search(query: "茅台")'

# 股票行情
mcporter call 'xueqiu.get_stock(symbol: "SH600519")'

# 热门帖子
mcporter call 'xueqiu.hot_posts()'
```

---

## 6. PowerShell 使用命令速查

### mcporter 调用语法

```powershell
# ✅ 推荐：key=value 格式（PowerShell 最稳定）
mcporter call exa.web_search_exa query="搜索词"
mcporter call exa.web_search_exa query="搜索词" numResults=10

# ✅ 也可以：冒号格式（搜索词不含中文标点时）
mcporter call 'exa.web_search_exa(query: "search term")'

# ❌ 不要用：JSON 格式（PowerShell 下引号冲突）
mcporter call 'exa.web_search_exa({"query": "搜索词"})'  # 会报错

# ❌ 搜索词含中文冒号会报错
mcporter call exa.web_search_exa query="清镇市：差异化考核"  # 报错
# 改为：
mcporter call exa.web_search_exa query="清镇市差异化考核"
```

### 常用快速命令

```powershell
# 查看 mcporter 已配置的服务
mcporter config list

# 查看 exa 可用的工具列表
mcporter tools exa

# 检查 agent-reach 各平台状态
& "C:\Users\Administrator\.agent-reach-venv\Scripts\agent-reach.exe" doctor

# 激活 agent-reach 虚拟环境
& "C:\Users\Administrator\.agent-reach-venv\Scripts\Activate.ps1"
```

---

## 7. mcporter 使用规则

mcporter 是 MCP（Model Context Protocol）服务的运行器，负责连接各类 AI 工具服务。

### 配置管理

```powershell
# 查看所有已配置的服务
mcporter config list

# 添加新服务（HTTP 方式）
mcporter config add 服务名 http://服务地址/mcp --system

# 添加新服务（命令方式）
mcporter config add 服务名 --command "启动命令" --system

# 删除服务
mcporter config remove 服务名

# 配置文件位置
# 系统级：C:\Users\Administrator\.mcporter\mcporter.json
# 项目级：当前目录\config\mcporter.json
```

### 已配置服务（本机）

```powershell
mcporter config list
# 应显示：
# exa  →  https://mcp.exa.ai/mcp (HTTP)
```

### 调用格式参考

```powershell
# 格式1：最简单（推荐）
mcporter call 服务名.工具名 参数名="参数值"

# 格式2：带括号
mcporter call '服务名.工具名(参数名: "参数值")'

# 格式3：位置参数
mcporter call '服务名.工具名("参数值1", "参数值2")'

# 附加选项
mcporter call 服务名.工具名 参数="值" --timeout 30000   # 超时30秒
```

---

## 8. 配置 Cookie 的统一流程

需要 Cookie 的平台（Twitter、小红书、雪球等）统一流程：

### 步骤

1. **安装 Chrome 插件**：[Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)

2. **浏览器登录目标平台**（建议用小号，有封号风险）

3. **导出 Cookie**：
   - 打开对应平台页面
   - 点击 Cookie-Editor 插件图标
   - 点击 **Export** → 选择 **Header String** 格式
   - 复制导出的字符串

4. **设置环境变量**（以 Twitter 为例）：

```powershell
# 临时设置（当前会话有效）
$env:TWITTER_AUTH_TOKEN = "从Cookie中提取的auth_token值"
$env:TWITTER_CT0 = "从Cookie中提取的ct0值"

# 永久设置（写入 PowerShell Profile）
Add-Content $PROFILE "`n`$env:TWITTER_AUTH_TOKEN = 'auth_token值'"
Add-Content $PROFILE "`n`$env:TWITTER_CT0 = 'ct0值'"
```

### 各平台环境变量名

| 平台 | 环境变量 |
|------|---------|
| Twitter | `TWITTER_AUTH_TOKEN`, `TWITTER_CT0` |
| 雪球 | Cookie 文件路径（rookiepy 自动读取浏览器） |
| 小红书 | OpenCLI 复用浏览器登录态，无需手动设置 |

---

## 9. 环境健康检查

### 全面检查

```powershell
# 检查所有平台状态（显示可用/不可用/需配置）
& "C:\Users\Administrator\.agent-reach-venv\Scripts\agent-reach.exe" doctor

# JSON 格式输出（更详细，可看 active_backend）
& "C:\Users\Administrator\.agent-reach-venv\Scripts\agent-reach.exe" doctor --json

# 检查是否有新版本
& "C:\Users\Administrator\.agent-reach-venv\Scripts\agent-reach.exe" check-update
```

### 单项检查

```powershell
# 检查 Node.js 版本
node --version  # 应显示 v24.16.0

# 检查 mcporter
mcporter --version

# 检查 Python
python --version  # 应显示 3.14.x

# 检查 yt-dlp
yt-dlp --version

# 检查 bili-cli
bili --version

# 检查 gh CLI
gh --version
gh auth status
```

### PowerShell Profile 配置（当前有效配置）

```powershell
# 路径：C:\Users\Administrator\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
# 内容：
$env:PATH = "C:\Users\Administrator\AppData\Local\Microsoft\WinGet\Links\fnm.exe;" + $env:PATH
$env:PATH = $env:PATH -replace '"', ''
fnm env --use-on-cd | Out-String | Invoke-Expression
fnm use 24 2>$null
$env:PATH = "C:\Users\Administrator\AppData\Roaming\fnm\node-versions\v24.16.0\installation;" + $env:PATH
$env:PATH = "C:\Users\Administrator\AppData\Local\Python\pythoncore-3.14-64\Scripts;" + $env:PATH
$env:PATH = "C:\Users\Administrator\.local\bin;" + $env:PATH
```

---

## 10. 实战使用示例

### 电商选品场景

```powershell
# 搜索 TikTok 玩具热卖榜（从专业媒体）
mcporter call exa.web_search_exa query="TikTok玩具品类热卖2026" includeDomains="tt123.com,chwang.com,amz123.com"

# 读取某个具体分析文章
curl "https://r.jina.ai/https://www.tt123.com/t/具体文章ID"

# 搜索竞品信息
mcporter call exa.web_search_exa query="TikTok美区毛绒玩具爆款 2026年"

# B站找竞品视频
bili search "TikTok选品教程" --type video -n 10
```

### 市场调研场景

```powershell
# 全网搜索行业动态
mcporter call exa.web_search_exa query="2026年跨境电商物流趋势"

# 搜索微信公众号文章
mcporter call exa.web_search_exa query="site:mp.weixin.qq.com 跨境物流渠道最新动态"

# V2EX 技术社区讨论
curl -s "https://www.v2ex.com/api/topics/show.json?node_name=jobs&page=1" -H "User-Agent: agent-reach/1.0"

# 读取具体政策原文
curl "https://r.jina.ai/https://www.mofcom.gov.cn/article/xxxxxx"
```

### 内容创作场景

```powershell
# 搜索小红书爆款内容（需配置 Cookie）
opencli xiaohongshu search "AI工具使用技巧" -f yaml

# YouTube 视频要点提取
yt-dlp --write-sub --skip-download -o "C:/tmp/%(id)s" "https://www.youtube.com/watch?v=VIDEO_ID"

# B站相关视频
bili search "AI Agent教程" --type video -n 5

# 读取行业报告
curl "https://r.jina.ai/https://报告URL"
```

### GitHub 代码调研场景

```powershell
# 搜索高 Star 项目
gh search repos "MCP server" --sort stars --limit 10

# 搜索特定语言代码
gh search code "agent-reach" --language python

# 查看项目 Issues
gh issue list -R Panniantong/Agent-Reach --state open

# 查看最新 Release
gh release list -R Panniantong/Agent-Reach
```

### Twitter 舆情监控场景（需配置）

```powershell
# 搜索特定话题
twitter search "Claude Code" -n 20

# 关注某用户动态
twitter user-posts @AnthropicAI -n 20

# 读取热门推文
twitter feed -n 30
```

### RSS 自动监控场景

```powershell
# 读取 AI 新闻 RSS
python -c "
import feedparser
feeds = [
    'https://36kr.com/feed',
    'https://sspai.com/feed'
]
for url in feeds:
    f = feedparser.parse(url)
    print(f'=== {f.feed.title} ===')
    for e in f.entries[:3]:
        print(f'  {e.title}')
        print(f'  {e.link}')
"
```

---

## 11. 常见问题排查

### Q1：`mcporter` 报 `Unknown MCP server 'exa'`

```powershell
# 原因：exa 配置不在当前目录的 config 中
# 解决：在 home 目录重新添加
cd C:\Users\Administrator
mcporter config add exa https://mcp.exa.ai/mcp
mcporter config list  # 验证已添加
```

### Q2：`fnm use 24` 报错

```powershell
# 原因：Profile 中 fnm 语法问题
# 检查 Profile
Get-Content $PROFILE

# 修复：确保 Profile 中是
# fnm use 24 2>$null   （不是 fnm use 24 2>）
```

### Q3：`mcporter` 命令找不到（新开终端后）

```powershell
# 原因：PATH 没有包含 Node 24 的安装目录
# 解决：检查 Profile 是否包含以下行
# $env:PATH = "C:\Users\Administrator\AppData\Roaming\fnm\node-versions\v24.16.0\installation;" + $env:PATH

# 临时修复
$env:PATH = "C:\Users\Administrator\AppData\Roaming\fnm\node-versions\v24.16.0\installation;" + $env:PATH
```

### Q4：Jina Reader 返回空内容或登录页

```powershell
# 原因：目标网站需要登录或 JS 渲染
# 解决：
# 1. 改用 Exa 搜索相关内容（不依赖特定 URL）
# 2. 如果是已知的公开页面，尝试直接 API 访问
# 3. 对于需要 Cookie 的站点，配置对应的 CLI 工具
```

### Q5：小红书搜索报 `AUTH_REQUIRED`

```powershell
# 原因：Chrome 浏览器里没有登录小红书
# 解决：在 Chrome 里打开 xiaohongshu.com 并登录（用小号）
# OpenCLI 扩展会自动复用浏览器登录态
```

### Q6：Twitter search 报 404

```powershell
# Twitter 频繁改 GraphQL 端点，按重试链操作：
# 1. 直接重试
twitter search "query" -n 10
# 2. 升级工具
pipx upgrade twitter-cli
twitter search "query" -n 10
# 3. 换 OpenCLI
opencli twitter search "query" -f yaml
# 4. 改用稳定命令
twitter feed -n 20
```

### Q7：B站搜索报错（yt-dlp 方式）

```powershell
# 原因：B站已对 yt-dlp 全面实施 412 风控
# 解决：改用 bili-cli
bili search "搜索词" --type video -n 5
# 不要用 yt-dlp 读 B站
```

---

## 12. 更新维护

### 更新 Agent-Reach

```powershell
# 方式1：让 AI Agent 执行（推荐）
# 发给 AI：帮我更新 Agent Reach：https://raw.githubusercontent.com/Panniantong/agent-reach/main/docs/update.md

# 方式2：手动更新
& "C:\Users\Administrator\.agent-reach-venv\Scripts\pip.exe" install --upgrade "https://github.com/Panniantong/agent-reach/archive/main.zip"
```

### 更新各上游工具

```powershell
# 更新 yt-dlp
yt-dlp -U

# 更新 twitter-cli
pipx upgrade twitter-cli

# 更新 bili-cli
pipx upgrade bili-cli

# 更新 mcporter
npm update -g mcporter

# 更新 gh CLI
winget upgrade GitHub.cli
```

### 查看版本信息

```powershell
& "C:\Users\Administrator\.agent-reach-venv\Scripts\agent-reach.exe" --version
mcporter --version
yt-dlp --version
bili --version
node --version
python --version
gh --version
```

---

## 附录：各平台上游工具 Star 数参考

| 工具 | Stars | 说明 |
|------|-------|------|
| yt-dlp | 154K ⭐ | 视频下载神器，支持 1800+ 站点 |
| Jina Reader | 9.8K ⭐ | 网页转 Markdown |
| twitter-cli | 2.1K ⭐ | Twitter 命令行工具 |
| xhs-cli | 1.5K ⭐ | 小红书 CLI |
| linkedin-mcp | 1.2K ⭐ | LinkedIn MCP 服务 |
| bili-cli | 590 ⭐ | B站 CLI |
| rdt-cli | 304 ⭐ | Reddit CLI |
| feedparser | 2.3K ⭐ | RSS 解析库 |

---

*文档生成时间：2026-06-14*  
*对应 Agent-Reach 版本：v1.4.0+*  
*本文档基于实际安装测试，Windows / PowerShell 环境*
