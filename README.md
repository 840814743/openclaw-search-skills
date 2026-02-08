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

<details>
<summary><b>📋 效果展示（点击展开）</b></summary>

以下是 `github-explorer` 分析 [RAG-Anything](https://github.com/HKUDS/RAG-Anything) 时的真实输出（略有精简）：

---

### [RAG-Anything](https://github.com/HKUDS/RAG-Anything)

**🎯 一句话定位**

基于 LightRAG 构建的**全模态 RAG 框架**——把 PDF、Office、图片、表格、公式等多模态文档统一解析、构建知识图谱、跨模态检索问答，一个框架搞定。

**⚙️ 核心机制**

五阶段流水线架构：

1. **文档解析** — 集成 MinerU 做高保真文档结构提取，支持 PDF/Office/图片等格式
2. **多模态内容理解** — 自动分类路由到专用处理管道（VLM 图像分析、表格解释、LaTeX 公式解析），并行处理
3. **双图知识图谱构建** — 多模态元素 → 知识图谱实体，建立跨模态语义关系
4. **混合检索** — 向量相似度 + 图遍历融合，模态感知排序
5. **VLM 增强查询** — 查询时自动将文档图像送入 VLM 做多模态联合分析

**📊 项目健康度**

- **Stars**: 12,733 | **Forks**: 1,516 | **License**: MIT
- **团队/作者**: HKUDS（香港大学数据科学实验室），核心作者 Zirui Guo（131 commits），导师 Chao Huang。同团队出品 LightRAG（EMNLP 2025，30k+ stars）
- **Commit 趋势**: 创建于 2025-06-06，最近 push 2026-01-26。核心功能迭代放缓 → 快速成长期尾声，进入稳定维护阶段
- **最近动态**: README 更新、parser.py 修复、版本更新

**🔥 精选 Issue**

- [#70 Image is not analyzed properly](https://github.com/HKUDS/RAG-Anything/issues/70) — 图像未被 VLM 实际分析，输出只有文件路径。暴露 vision_model_func 配置易错性
- [#49 Document Processing stuck](https://github.com/HKUDS/RAG-Anything/issues/49) — 文档处理流水线卡住，疑似 LLM API 超时或异步事件循环问题
- [#91 DocProcessingStatus unexpected keyword argument](https://github.com/HKUDS/RAG-Anything/issues/91) — 版本兼容性 bug
- [#146 能够支持本地模型了吗](https://github.com/HKUDS/RAG-Anything/issues/146) — 社区对 Ollama/HuggingFace 本地部署的强烈需求

**🆚 竞品对比**

- **vs [LightRAG](https://github.com/HKUDS/LightRAG)** — 同团队，RAG-Anything 是多模态扩展。LightRAG 专注纯文本 GraphRAG，更轻量稳定
- **vs [GraphRAG](https://github.com/microsoft/graphrag)** — 微软方案，社区更大、文档更完善，但只处理文本
- **vs [RAGFlow](https://github.com/infiniflow/ragflow)** — 国产全栈 RAG 平台，自带 Web UI，开箱即用。RAG-Anything 更偏研究框架/SDK

**📰 社区声量**

- [@huang_chao4969](https://x.com/huang_chao4969/status/1954578722172158333): "Following the release of LightRAG, our HKUDS team member Zirui Guo is back with RAG-Anything!"
- [知乎: RAG-Anything：解锁多模态文档解析的下一代RAG引擎](https://zhuanlan.zhihu.com/p/1920578973072601126)

**💬 我的判断**

**值得关注，但谨慎投入生产。** 定位很准——多模态文档 RAG 确实是刚需。但从工程角度看，更像一个研究原型：97 个 open issues、README 示例都能报错、核心迭代已放缓。适合做学术研究或 PoC 验证；上生产建议先评估 RAGFlow。

---

> ☝️ 以上是真实、未经编辑的输出。每个链接都可点击、可追溯到原始来源。

</details>

## License

MIT
