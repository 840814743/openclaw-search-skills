# GitHub Explorer（OpenClaw Skill）

有些项目的 README 写得像发布会 PPT：

- "🚀 Next-gen / SOTA / blabla…"
- badge 一排，图很炫
- 但你真正想知道的：**活不活跃、坑多不多、作者回不回 Issue、到底适不适合你** —— README 基本不告诉你。

`github-explorer` 这个 skill 做的事很简单：

> 让 OpenClaw 帮你把"评估一个 GitHub 项目"这件事做得更像一个认真看过源码/社区的人，而不是只读了 README。

---

## 你会得到什么

当你对 OpenClaw 说：

- "帮我看看这个 GitHub 项目 raganything"
- "分析一下 HKUDS/LightRAG"

它会输出一份结构化报告（带链接、可追溯）：

- 一句话定位（它到底解决什么问题）
- 核心机制（不是复制 README，用人话讲架构/流程）
- 项目健康度（Stars/Forks/License、作者背景、commit 近况）
- 精选 Issues（按评论/价值挑 3–5 条，指出真实风险点）
- 适用场景 & 局限（什么时候该用/别碰）
- 竞品对比（同赛道项目 + 链接）
- 论文 / Demo / DeepWiki / Zread 等"知识图谱入口"
- 社区声量（X/Twitter、知乎/V2EX 等，**引用具体内容** + 原链接）
- 最后一段：它自己的判断（值不值得投入时间）

---

## 依赖

`github-explorer` 本身只是总控流程（一个 `SKILL.md`，没有脚本），真正干活靠同仓库的几个 plumbing skills：

| 依赖 Skill | 作用 | 位置 |
|------------|------|------|
| [search-layer](../search-layer/) | Exa + Tavily 多源搜索 + 去重 | 本仓库 `search-layer/` |
| [content-extract](../content-extract/) | 网页抓取失败/反爬时，自动走 MinerU 高保真解析 | 本仓库 `content-extract/` |
| [mineru-extract](../mineru-extract/) | MinerU 官方 API 封装（content-extract 的下游） | 本仓库 `mineru-extract/` |

另外还依赖 OpenClaw 的**内置工具**（无需额外安装）：

- `web_search` — Brave Search 检索
- `web_fetch` — 网页内容抓取
- `browser` — 动态页面渲染（备选）

> ⚠️ **四个 skill 需要一起安装**，否则 github-explorer 在调研过程中会缺少搜索和内容提取能力。

---

## 安装

### 方式一：让 OpenClaw 帮你装（推荐 🚀）

直接在对话里告诉你的 OpenClaw agent：

> 帮我安装这个 skill：https://github.com/blessonism/openclaw-search-skills

### 方式二：手动安装（symlink）

```bash
# 1. Clone 仓库
mkdir -p ~/.openclaw/workspace/_repos
git clone https://github.com/blessonism/openclaw-search-skills.git \
  ~/.openclaw/workspace/_repos/openclaw-search-skills

# 2. 链接到 skills 目录（四个一起）
cd ~/.openclaw/workspace/skills

ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/github-explorer github-explorer
ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/search-layer search-layer
ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/content-extract content-extract
ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/mineru-extract mineru-extract
```

> 💡 skills 目录因安装方式不同可能不同，常见的是 `~/.openclaw/workspace/skills/` 或 `~/.openclaw/skills/`。

---

## 配置

### 1) search-layer 的 API Keys（建议配）

环境变量：

```bash
export EXA_API_KEY="..."
export TAVILY_API_KEY="..."
```

或者写到 OpenClaw workspace 的 `TOOLS.md` 里（search-layer 会去读）：

```md
### Search
- **Exa**: `your-exa-key`
- **Tavily**: `your-tavily-key`
```

### 2) MinerU Token（可选，但强烈建议）

当你要抓微信/知乎/小红书这种经常 403 的站点时，`content-extract` 会用 MinerU 兜底。

```bash
cp mineru-extract/.env.example mineru-extract/.env
# 编辑 .env，填入你的 MINERU_TOKEN（从 https://mineru.net/apiManage 获取）
```

---

## 使用

直接在对话里说就行：

- "帮我看看这个 GitHub 项目 xxx"
- "这个 repo 怎么样"

你不需要指定 mode；skill 会自动：

- 项目调研 → 默认走 `search-layer --mode deep`
- 单源失败 → 自动降级，不会因为一个 API 挂了就中断
- 反爬站点 → 自动升级到 MinerU

---

## 常见问题

### Q: 为什么搜中文社区时结果少？

Brave 免费额度有速率限制，容易触发 429。这个 skill 的策略是：Brave 限流时继续用 Exa/Tavily。所以 **Exa/Tavily 的 key 很关键**，建议都配上。

### Q: SKILL.md 和 README.md 有什么区别？

- `SKILL.md` 是给 agent 看的"行为规范"——定义了工作流、输出模板、自检清单
- `README.md` 是给人看的"使用说明"——你现在在读的就是

---

## License

MIT
