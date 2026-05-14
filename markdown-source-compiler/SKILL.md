---
name: markdown-source-compiler
description: Use this skill when Hermes must turn messy Markdown from video subtitles, web clippers, PDF converters, or human notes into a clean, faithful, standardized source Markdown document. The output is not a wiki page; it is normalized raw material for later wiki ingestion.
---

# Markdown Source Compiler

## Trigger Conditions

Use this skill when the user asks Hermes to:

- 整理 Markdown
- 清洗字幕
- 规范化网页剪藏
- 整理 PDF 转 Markdown
- 把下载/导出的材料变成规范 raw 文档
- prepare a source document for wiki ingestion

Do not use older summary-only skills for this workflow unless the user explicitly asks for a polished standalone summary instead of a normalized source document.

## Portable Configuration

This skill must be portable across machines. Do not hard-code any absolute path in the skill output or workflow.

Before processing, establish the user's knowledge root:

```text
KNOWLEDGE_ROOT = the Obsidian knowledge folder provided by the user
```

If the user has not provided a knowledge root, ask for it first. Use the exact path the user provides. Example shapes:

- Windows: `<drive>:\<ObsidianVaultOrFolder>\Knowledge`
- macOS/Linux: `/<home>/<ObsidianVaultOrFolder>/Knowledge`

After the knowledge root is known, use this relative layout:

```text
{KNOWLEDGE_ROOT}/
├── inbox/              # messy exported or clipped material, never overwrite
│   ├── web/
│   ├── pdf/
│   ├── video/
│   └── notes/
├── raw/                # normalized source Markdown produced by this skill
│   ├── web/
│   ├── pdf/
│   ├── video/
│   └── notes/
└── wiki/               # maintained by wiki-maintainer, not this skill
```

When the user first configures this skill, explain:

- Put uncleaned files in `{KNOWLEDGE_ROOT}/inbox/{web|pdf|video|notes}/`
- This skill writes cleaned files to `{KNOWLEDGE_ROOT}/raw/{web|pdf|video|notes}/`
- The next skill, `wiki-maintainer`, writes final knowledge pages to `{KNOWLEDGE_ROOT}/wiki/`

Use this onboarding response format:

```markdown
已识别 Knowledge Root：`{KNOWLEDGE_ROOT}`

你的知识流水线目录是：

- 未清洗材料：`{KNOWLEDGE_ROOT}/inbox/`
- Hermes 清洗后的规范 raw：`{KNOWLEDGE_ROOT}/raw/`
- 最终 Obsidian Wiki：`{KNOWLEDGE_ROOT}/wiki/`

请把网页剪藏放到 `inbox/web/`，PDF 转换结果放到 `inbox/pdf/`，视频字幕放到 `inbox/video/`，手写笔记放到 `inbox/notes/`。
```

**IMPORTANT: When re-installing this skill or loading it on a new machine, you MUST execute the Portable Configuration steps before processing any files. Do NOT skip the KNOWLEDGE_ROOT confirmation and onboarding response even if you think you already know the path. Confirm the path with the user explicitly.**

## Role

You are a source document compiler.

Your task is to convert messy, unreliable, or poorly structured Markdown into a clean source Markdown document that preserves the source faithfully while making it readable, structured, and suitable for later wiki ingestion.

Do not turn the document into a knowledge wiki page. Do not over-summarize. Do not add unsupported facts.

This skill produces normalized source material. The next stage is handled by `wiki-maintainer`.

## Input

Input usually comes from:

- video subtitles exported as Markdown
- web clipper Markdown
- docling PDF-to-Markdown output
- rough human notes

These inputs may contain:

- timestamps
- repeated lines
- filler words
- broken headings
- ads, navigation, and boilerplate
- iframe/player snippets
- duplicated paragraphs
- bad tables
- missing metadata
- claims that may be false or exaggerated

## Output Destination

Write the cleaned source document under:

```text
{KNOWLEDGE_ROOT}/raw/{web|pdf|video|notes}/
```

The original input should remain unchanged under:

```text
{KNOWLEDGE_ROOT}/inbox/{web|pdf|video|notes}/
```

If the user provides a one-off input file outside `{KNOWLEDGE_ROOT}/inbox/`, treat that file as read-only and still write normalized output under `{KNOWLEDGE_ROOT}/raw/{web|pdf|video|notes}/`.

If the current system only has `raw/`, treat the original source as read-only and create the cleaned file with a clear suffix such as:

```text
{KNOWLEDGE_ROOT}/raw/video/<title>.normalized.md
```

## Core Rules

- Preserve source meaning.
- Remove noise, not evidence.
- Do not invent facts.
- Do not convert uncertain transcript fragments into confident technical facts.
- Do not silently correct suspicious claims; mark them.
- Keep source-specific metadata.
- Keep exact commands, code, URLs, dates, version numbers, error messages, and numeric claims.
- Convert messy transcript language into readable prose only when meaning is clear.
- Keep uncertainty visible.
- When a known technical entity conflicts with the transcript, prefer a cautious phrasing and put the conflict in `## Caveats`.

## English-to-Chinese Translation Rule

**IF** the source document is primarily in English, **THEN** add an automatic translation step **before** the rest of the compilation pipeline:

### Translation Guidelines

- Translate the document content into **Chinese (Simplified)** while keeping the overall structure intact
- **DO NOT translate** the following technical terms — keep them in their original English form:
  - Protocol names (e.g. "Model Context Protocol", "HTTP", "TCP/IP")
  - Programming languages, frameworks, libraries (e.g. "Python", "React", "Django")
  - Tool/software names (e.g. "Docker", "Kubernetes", "Git")
  - API names, SDK names, library names
  - Code keywords (e.g. "def", "class", "import", "async")
  - Command names, CLI flags
  - File extensions, environment variable names
- Translate general English prose naturally into Chinese
- Preserve proper nouns (people names, company names, product names) in original form
- Preserve all code blocks, command-line examples, and URLs exactly as-is
- After translation, label the document with `source_language: en` and `translated: true` in frontmatter
- Translate the doc, DO NOT rewrite or reorganize it (the reorganization can be done in the wiki-maintainer stage)

## Required Frontmatter

Every output file must start with:

```yaml
---
title:
source_type:
source_url:
author:
published:
captured:
compiled:
compiler: hermes
status: normalized
confidence: medium
tags: []
---
```

Use empty values when unknown. Use `confidence: low` if the input is incomplete, promotional, or factually suspicious.

## Required Structure

Use this structure unless the content clearly requires another one:

```markdown
# Title

## Source Summary

Briefly describe what this source says. This is a source summary, not a final knowledge conclusion.

## Clean Outline

- Section 1
- Section 2

## Main Content

### Section

Cleaned source content.

## Key Claims

- Claim from the source.

## Technical Details

- Commands, code, formulas, procedures, parameters, tools, versions, metrics.

## Caveats

- Exaggerated, unsupported, ambiguous, outdated, or promotional points.

## Useful For Wiki

- Concepts that may deserve wiki pages later.
- Entities that may deserve wiki pages later.
- Procedures that may deserve wiki pages later.
```

## Video Subtitle Rules

For subtitles:

- If the document already has a human-written `## 简介`, use it as a candidate summary, but verify it against the transcript instead of copying it blindly.
- Remove timestamp prefixes from body prose.
- Keep a `## Timeline` section when timestamps are useful.
- Merge fragmented subtitle lines into paragraphs.
- Remove filler words such as "嗯", "啊", "对吧" when they add no meaning.
- Preserve technical terms, numbers, claims, and examples.
- Separate creator claims from verified facts.
- If the transcript contains mistaken speech recognition such as "Brother first" for "Browser First", correct it only when the surrounding context makes the correction obvious.
- Keep important English workflow names as English plus Chinese explanation, for example `Terminal First（终端优先）` and `Browser First（浏览器优先）`.
- For proper nouns that may be ASR errors, such as `Sway`, `Wayland`, `Xorg`, `Firefox`, `Chrome`, `SurfingKeys`, and `Vimium`, do not invent relationships between them. If unsure, write "the transcript appears to mention..." and add a caveat.

### Chinese ASR Correction Patterns (Bilibili / Chinese Tech Talks)

Chinese ASR on Bilibili tech videos frequently mangles English technical terms. Common patterns:

**Phonetic approximation** — English name read with Chinese accent → mangled by ASR:
- `拉姆CPP / 拉玛CPP / 兰马CPP` → `llama.cpp`
- `希浪 / 新浪 / I希浪 / S新浪 / I希浪` → `SGLang`
- `VRM` → `vLLM`
- `欧拉玛 / 欧拉A` → `Ollama`
- `deep sk / deep sick` → `DeepSeek`
- `穆萨雷` → `MUSA`
- `TREATIN` → Triton (language compiler)
- `MOON cake` → MoonCake
- `REDX attention` → RadixAttention
- `treat` → TRT (TensorRT)
- `metron` → Megatron
- `long chain / long fuse / long graph` → LangChain / LangFuse / LangGraph

**Semantic substitution** — ASR hears different Chinese word:
- `dance 模型 / dance` → Dense 模型 (dense is a common ML term often mis-heard as "dance")
- `destroy / DESTROLL` → Distill (distilled models)
- `perfume` → Prefill (the ML inference phase)
- `DECO` → Decode
- `batch / badge` → batch

**Correction rules:**
1. When 3+ surrounding captions reference the same technical concept, assume the ASR got the name wrong and correct it
2. Always list every correction in `## Caveats` so the user can verify
3. If uncertain between two possible corrections, use `the source appears to mention [最可能的术语] (transcript says: "xxx")` 
4. Do NOT invent tool relationships from noisy transcript text (e.g., do not write "Sway is based on Xorg" from ASR noise)

## Technical QA Rules

Before finalizing, run a short consistency pass:

- Check whether any known tool/protocol relationship was asserted from noisy transcript text.
- If the source says or implies something technically suspicious, preserve it as a source claim rather than a fact.
- Example: do not write "Sway is based on Xorg"; use "the source discusses Wayland/Xorg-era window-manager choices and mentions Sway" unless the source clearly states a correct technical relationship.
- Avoid turning speech-recognition artifacts into new tool names.

Timeline format:

```markdown
## Timeline

- `00:00` Topic name
- `02:15` Topic name
```

## Web Clipper Rules

For clipped web pages:

- Remove navigation, footer, share buttons, cookie banners, and ads.
- Preserve article title, author, source URL, publish date, and cited links.
- Keep quoted material short.
- Preserve code blocks and tables.
- Mark broken extraction areas under `## Caveats`.

## PDF/docling Rules

For docling output:

- Repair heading hierarchy.
- Repair tables when feasible.
- Preserve page references if present.
- Keep figure captions and references.
- Mark OCR uncertainty or broken tables.

## Human Note Rules

For rough notes:

- Preserve the author's intent.
- Organize into headings.
- Mark TODOs and unclear parts.
- Do not make informal notes sound more certain than they are.

## Claim Handling

Use this format when a source makes claims:

```markdown
## Key Claims

- The source claims that ...
- The source claims that ...
```

Use this format for doubtful claims:

```markdown
## Caveats

- The claim "..." is not verified by this source.
- The source appears promotional here.
- This numeric result depends on unstated assumptions.
```

## Completion Checklist

Before finishing:

- The output is valid Markdown.
- The output has frontmatter.
- The original input was not overwritten.
- Timestamps/noise/ads are removed or isolated.
- Claims remain traceable to the source.
- Suspicious content is marked under `## Caveats`.
- The document is suitable as normalized raw material for wiki ingestion.

## Output Naming

For every normalized output file, generate a clean portable filename.

Filename format:

```text
YYYY-MM-DD-清晰中文标题.normalized.md
```

Date prefix rules:

- Prefer `captured`, `created`, or another capture date from source frontmatter if it is a valid `YYYY-MM-DD`.
- Otherwise use the current date.
- Do not use an upload/publish date as the file prefix unless there is no capture/created/compiled date.

Title rules:

- Derive a concise title from frontmatter `title`, H1, or the document's main topic.
- If the source filename/title is primarily English, translate the general meaning into Simplified Chinese.
- Preserve important technical terms in English, such as `Terminal First`, `Browser First`, `Linux`, `Arch Linux`, `RAG`, `LLM`, `API`, `SDK`, `Docker`, `Kubernetes`, `React`, `Python`, `Git`.
- Remove or rewrite category brackets such as `[linux]`; convert them into normal title words when useful, such as `Linux`.
- Remove abnormal filename characters, including:

```text
< > : " / \ | ? * [ ] { } ` ~
```

- Remove control characters, emoji decorations, repeated punctuation, and trailing dots/spaces.
- Collapse repeated whitespace and separators into a single space or hyphen.
- Keep the filename under 120 characters when practical.
- The H1/title inside the document should match the cleaned semantic title, without the date prefix and without `.normalized`.

Examples:

```text
Input filename:
[linux] 从 Terminal First 到 Browser First 的 Linux 工作流转变.md

Output filename:
2026-05-12-Linux 从 Terminal First 到 Browser First 的工作流转变.normalized.md

Input filename:
How I use RAG with Obsidian???!!.md

Output filename:
2026-05-13-我如何在 Obsidian 中使用 RAG.normalized.md
```

If two output filenames collide, append a short numeric suffix:

```text
2026-05-13-标题.normalized-2.md
```
