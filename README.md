# OpenClaw Search Skills

一组 [OpenClaw](https://github.com/openclaw/openclaw) 技能（Skills），用于 **GitHub 项目深度调研** 和 **多源内容提取**。

## 包含什么

| Skill | 干什么的 |
|-------|---------|
| **[github-explorer](./github-explorer/)** | 对你说"帮我看看这个项目"，就能拿到一份完整的尽职调查报告——Stars、Issues、竞品、社区口碑、以及一段主观判断。 |
| **[search-layer](./search-layer/)** | 多源搜索（Exa + Tavily）+ 自动去重。Brave 由 OpenClaw 内置的 `web_search` 提供。 |
| **[content-extract](./content-extract/)** | URL → 干净的 Markdown。遇到反爬站点（微信、知乎）自动降级到 MinerU 解析。 |
| **[mineru-extract](./mineru-extract/)** | [MinerU](https://mineru.net) 官方 API 的封装层。把 PDF、Office 文档、HTML 页面转成 Markdown。 |

## 它们之间的关系

```
github-explorer（总控大脑）
├── search-layer ──── Exa + Tavily 并行搜索
├── content-extract ── 智能 URL → Markdown
│   └── mineru-extract ── MinerU API（重活）
└── OpenClaw 内置工具 ── web_search, web_fetch, browser
```

`github-explorer` 是你直接交互的 skill，其他三个是它的管道——当然你也可以单独使用它们。

## 安装

### 方式一：让 OpenClaw 帮你装（推荐 🚀）

直接在对话里告诉你的 OpenClaw agent：

> 帮我安装这个 skill：https://github.com/blessonism/openclaw-search-skills

agent 会自动 clone 仓库并把 skill 链接到正确的位置。

### 方式二：用 ClawHub CLI

如果这些 skill 已发布到 [ClawHub](https://clawhub.com)：

```bash
npx clawhub install github-explorer
npx clawhub install search-layer
npx clawhub install content-extract
npx clawhub install mineru-extract
```

### 方式三：手动安装

```bash
# 1. Clone 到任意位置
mkdir -p ~/.openclaw/workspace/_repos
git clone https://github.com/blessonism/openclaw-search-skills.git \
  ~/.openclaw/workspace/_repos/openclaw-search-skills

# 2. 链接到你的 skills 目录
cd ~/.openclaw/workspace/skills

ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/github-explorer github-explorer
ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/search-layer search-layer
ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/content-extract content-extract
ln -s ~/.openclaw/workspace/_repos/openclaw-search-skills/mineru-extract mineru-extract
```

> 💡 你的 skills 目录可能因安装方式不同而不同，常见的是 `~/.openclaw/workspace/skills/` 或 `~/.openclaw/skills/`。

## 配置

### 搜索 API Keys（search-layer 需要）

两种方式任选其一：

**环境变量：**

```bash
export EXA_API_KEY="your-exa-key"        # https://exa.ai
export TAVILY_API_KEY="your-tavily-key"  # https://tavily.com
```

**或写到 TOOLS.md（OpenClaw workspace 根目录）：**

```markdown
### Search
- **Exa**: `your-exa-key`
- **Tavily**: `your-tavily-key`
```

### MinerU Token（可选，content-extract 需要）

只有当你需要抓取微信/知乎/小红书等反爬站点时才需要：

```bash
cp mineru-extract/.env.example mineru-extract/.env
# 编辑 .env，填入你的 MinerU token（从 https://mineru.net/apiManage 获取）
```

### Python 依赖

```bash
pip install requests  # 唯一的外部依赖
```

## 使用

直接在对话里说：

> "帮我看看这个 GitHub 项目 raganything"

> "分析一下 HKUDS/LightRAG"

agent 会自动读取 `github-explorer/SKILL.md`，启动多源调研流水线，输出结构化报告。

### 单独使用各 skill

**search-layer：**

```bash
python3 search-layer/scripts/search.py "RAG framework comparison" --mode deep --num 5
```

模式：`fast`（仅 Exa）、`deep`（Exa + Tavily 并行）、`answer`（Tavily 带 AI 摘要）

**content-extract：**

```bash
python3 content-extract/scripts/content_extract.py --url "https://mp.weixin.qq.com/s/some-article"
```

**mineru-extract：**

```bash
python3 mineru-extract/scripts/mineru_extract.py "https://example.com/paper.pdf" --model pipeline --print
```

## 环境要求

- [OpenClaw](https://github.com/openclaw/openclaw)（agent 运行时）
- Python 3.10+
- `requests`（pip install）
- API Keys：Exa 和/或 Tavily（用于 search-layer），MinerU token（可选，用于 content-extract）

## License

MIT
