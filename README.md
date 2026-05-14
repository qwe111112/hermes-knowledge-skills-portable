# Hermes Knowledge Skills

把网页剪藏、视频字幕、PDF 转 Markdown 和个人笔记，编译成可长期维护的 Obsidian 个人知识库。

这个项目提供三个 Hermes Agent skills：

- `markdown-source-compiler`：把混乱 Markdown 整理成规范 raw Markdown。
- `wiki-maintainer`：把规范 raw Markdown 沉淀成 Obsidian Wiki 页面。
- `knowledge-query-assistant`：调用现有 Wiki 回答问题，并标注来源与不确定性。

它适合想用 Hermes + Obsidian 搭建个人版 LLM Wiki 的用户。

## Why

日常收集的资料通常不是直接可用的知识：

- 视频字幕有时间戳、口语、ASR 错误。
- 网页剪藏有导航、广告、重复内容。
- PDF 转 Markdown 可能有破碎标题、表格和 OCR 问题。
- 自己写的笔记可能结构松散、命名混乱。

这个项目把知识处理拆成两个阶段：

```text
messy markdown
  -> normalized raw markdown
  -> Obsidian wiki knowledge
```

核心目标不是让 AI 临时总结一篇文章，而是让 AI 持续维护一个可读、可链接、可演化的个人知识库。

## Workflow

用户只需要告诉 Hermes 一个知识库根目录：

```text
KNOWLEDGE_ROOT = your Obsidian knowledge folder
```

例如：

```text
<drive>:\<ObsidianVaultOrFolder>\Knowledge
/<home>/<ObsidianVaultOrFolder>/Knowledge
```

Hermes 会使用这个可移植目录结构：

```text
{KNOWLEDGE_ROOT}/
├── inbox/              # 未清洗材料，永不覆盖
│   ├── web/
│   ├── pdf/
│   ├── video/
│   └── notes/
├── raw/                # Hermes 清洗后的规范 Markdown
│   ├── web/
│   ├── pdf/
│   ├── video/
│   └── notes/
└── wiki/               # Obsidian Wiki
    ├── index.md
    ├── log.md
    ├── domains/
    ├── sources/
    ├── concepts/
    ├── entities/
    └── topics/
```

## Skills

### markdown-source-compiler

用于第一阶段：清洗和规范化资料。

输入：

```text
{KNOWLEDGE_ROOT}/inbox/{web|pdf|video|notes}/
```

输出：

```text
{KNOWLEDGE_ROOT}/raw/{web|pdf|video|notes}/
```

它会处理：

- 视频字幕时间戳和口语清理
- 网页剪藏噪音清理
- PDF/docling 输出结构修复
- 英文标题翻译
- 异常文件名符号清理
- 技术名词保留
- 不确定内容和可疑 claims 标注

规范 raw 文件命名规则：

```text
YYYY-MM-DD-清晰中文标题.normalized.md
```

示例：

```text
Input:
[linux] 从 Terminal First 到 Browser First 的 Linux 工作流转变.md

Output:
2026-05-12-Linux 从 Terminal First 到 Browser First 的工作流转变.normalized.md
```

### wiki-maintainer

用于第二阶段：把规范 raw 文档沉淀成 Obsidian Wiki。

输入：

```text
{KNOWLEDGE_ROOT}/raw/{web|pdf|video|notes}/
```

输出：

```text
{KNOWLEDGE_ROOT}/wiki/
```

它会创建或更新：

- `wiki/sources/`：来源笔记
- `wiki/domains/`：领域首页
- `wiki/concepts/`：概念页
- `wiki/entities/`：工具、人物、协议、产品等实体页
- `wiki/topics/`：主题页
- `wiki/index.md`：全局索引
- `wiki/log.md`：处理日志

Wiki source note 会去掉 raw 文件的日期前缀和 `.normalized` 后缀：

```text
raw/video/2026-05-12-Linux 从 Terminal First 到 Browser First 的工作流转变.normalized.md
  -> wiki/sources/Linux 从 Terminal First 到 Browser First 的工作流转变.md
```

### knowledge-query-assistant

用于第三阶段：让 Hermes 调用现有 Obsidian Wiki 回答问题。

默认检索范围：

```text
{KNOWLEDGE_ROOT}/wiki/
```

检索顺序：

```text
wiki/index.md
  -> 相关 domain/topic/concept/entity/source 页面
  -> 必要时回溯 raw/
```

回答要求：

- 优先根据 wiki 已有内容回答
- 使用 `[[Obsidian Links]]` 标注来源
- 区分“知识库已有”和“根据现有笔记推断”
- 知识库没有覆盖时明确说明
- 保留 caveats 和来源不确定性

## Installation

复制这两个 skill 文件夹到 Hermes skills 目录：

```text
markdown-source-compiler/
wiki-maintainer/
knowledge-query-assistant/
```

常见本地目录形态：

```text
<Hermes config dir>/skills/
```

安装后确认 Hermes 能识别：

```powershell
hermes skills list
```

应能看到：

```text
markdown-source-compiler
wiki-maintainer
knowledge-query-assistant
```

## Usage

### 1. 初始化 Knowledge Root

告诉 Hermes：

```text
我的 KNOWLEDGE_ROOT 是 <你的 Obsidian Knowledge 路径>。
请使用 markdown-source-compiler 和 wiki-maintainer。
```

Hermes 应该回复：

```text
未清洗材料：{KNOWLEDGE_ROOT}/inbox/
Hermes 清洗后的规范 raw：{KNOWLEDGE_ROOT}/raw/
最终 Obsidian Wiki：{KNOWLEDGE_ROOT}/wiki/
```

### 2. 放入未清洗文件

根据来源类型放入：

```text
inbox/web/      # 网页剪藏
inbox/pdf/      # PDF 转 Markdown
inbox/video/    # 视频字幕
inbox/notes/    # 手写笔记
```

### 3. 运行第一阶段

示例 prompt：

```text
请使用 markdown-source-compiler，整理 inbox/video/xxx.md。
输出到 raw/video，不要修改 inbox 原文件。
```

### 4. 运行第二阶段

示例 prompt：

```text
请使用 wiki-maintainer，把 raw/video/xxx.normalized.md 沉淀到 wiki。
更新 index.md 和 log.md。
```

也可以让 Hermes 一次跑完整流程：

```text
请使用 markdown-source-compiler 和 wiki-maintainer，
处理 inbox/video/xxx.md：
先生成 normalized raw，再沉淀到 wiki。
```

### 5. 调用知识库回答

示例 prompt：

```text
我的 KNOWLEDGE_ROOT 是 <你的 Obsidian Knowledge 路径>。
请使用 knowledge-query-assistant，
根据我的 wiki 比较 vLLM 和 SGLang，并引用相关页面。
```

推荐回答结构：

```text
结论
对比表
依据页面
不确定点
```

## Example

输入文件：

```text
inbox/video/[linux] 从 Terminal First 到 Browser First 的 Linux 工作流转变.md
```

第一阶段输出：

```text
raw/video/2026-05-12-Linux 从 Terminal First 到 Browser First 的工作流转变.normalized.md
```

第二阶段输出：

```text
wiki/index.md
wiki/log.md
wiki/domains/Linux.md
wiki/sources/Linux 从 Terminal First 到 Browser First 的工作流转变.md
wiki/concepts/Terminal First.md
wiki/concepts/Browser First.md
wiki/entities/Arch Linux.md
wiki/entities/Sway.md
wiki/topics/Linux 工作流.md
```

## Design Principles

- 原始文件不可覆盖。
- raw 是规范来源材料，不是最终知识。
- wiki 是长期知识沉淀。
- 不把来源 claims 直接当事实。
- 保留不确定性、ASR 错误、OCR 问题和来源偏见。
- Obsidian 链接必须可解析。
- 文件名要适合人读，也要适合长期维护。
- skill 不写死用户机器路径，只依赖 `{KNOWLEDGE_ROOT}`。

## Filename Rules

Raw 文件：

```text
YYYY-MM-DD-清晰中文标题.normalized.md
```

Wiki source 文件：

```text
清晰中文标题.md
```

Concept/entity/topic 文件：

```text
概念名.md
实体名.md
主题名.md
```

示例：

```text
raw/video/2026-05-12-Linux 从 Terminal First 到 Browser First 的工作流转变.normalized.md
wiki/sources/Linux 从 Terminal First 到 Browser First 的工作流转变.md
wiki/concepts/Browser First.md
wiki/entities/Sway.md
```

## Repository Layout

建议开源仓库结构：

```text
hermes-knowledge-skills/
├── README.md
├── LICENSE
├── markdown-source-compiler/
│   └── SKILL.md
├── wiki-maintainer/
│   └── SKILL.md
└── knowledge-query-assistant/
    └── SKILL.md
```

## Status

当前版本处于 MVP 阶段，已验证：

- 可移植 `{KNOWLEDGE_ROOT}` 配置
- 视频字幕 Markdown 清洗
- 日期前缀文件命名
- 异常符号清理
- Obsidian wiki 页面生成
- index/log 更新
- source/concept/entity/topic/domain 分层
- 基于 wiki 的问答检索和来源标注

## License

建议使用 MIT License，便于别人复制、修改和二次分发。
