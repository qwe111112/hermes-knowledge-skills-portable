---
name: wiki-maintainer
description: Use this skill when Hermes must turn normalized source Markdown into an Obsidian-style personal knowledge wiki by creating or updating source, concept, entity, and topic notes with links, index updates, and an append-only log.
---

# Wiki Maintainer

## Trigger Conditions

Use this skill when the user asks Hermes to:

- 沉淀到 wiki
- 更新知识库
- 按 wiki 思想处理
- 从 raw 文档生成 Obsidian 笔记
- create concept/entity/topic notes

Do not use this skill directly on messy exports unless the user explicitly asks to skip normalization. Messy exports should go through `markdown-source-compiler` first.

## Portable Configuration

This skill must be portable across machines. Do not hard-code any absolute path in the skill itself.

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
├── inbox/              # messy material handled by markdown-source-compiler
├── raw/                # normalized source Markdown
└── wiki/               # Obsidian wiki maintained by this skill
    ├── index.md
    ├── log.md
    ├── domains/
    ├── sources/
    ├── concepts/
    ├── entities/
    └── topics/
```

When the user first configures this skill, explain:

- Put uncleaned files in `{KNOWLEDGE_ROOT}/inbox/{web|pdf|video|notes}/`
- Normalized source files live in `{KNOWLEDGE_ROOT}/raw/{web|pdf|video|notes}/`
- This skill writes final wiki notes to `{KNOWLEDGE_ROOT}/wiki/`

Use this onboarding response format:

```markdown
已识别 Knowledge Root：`{KNOWLEDGE_ROOT}`

你的知识流水线目录是：

- 未清洗材料：`{KNOWLEDGE_ROOT}/inbox/`
- Hermes 清洗后的规范 raw：`{KNOWLEDGE_ROOT}/raw/`
- 最终 Obsidian Wiki：`{KNOWLEDGE_ROOT}/wiki/`

`wiki-maintainer` 只处理 `raw/` 里的规范源文档。如果文件还在 `inbox/`，请先运行 `markdown-source-compiler`。
```

**IMPORTANT: When re-installing this skill or loading it on a new machine, you MUST execute the Portable Configuration steps (confirm KNOWLEDGE_ROOT + show onboarding response) before processing any files. Do not skip this step even if you think you already know the path.**

## Role

You are a wiki maintainer.

Your task is to read normalized source Markdown and update a durable Obsidian-compatible Markdown wiki.

Do not merely summarize the source. Integrate useful knowledge into existing wiki pages.

## Input

Input should come from:

```text
{KNOWLEDGE_ROOT}/raw/{web|pdf|video|notes}/
```

These files should already be normalized by the `markdown-source-compiler` skill.

## Output

Write and update files under:

```text
{KNOWLEDGE_ROOT}/wiki/
```

Use this structure:

```text
wiki/
├── index.md
├── log.md
├── domains/
├── sources/
├── concepts/
├── entities/
└── topics/
```

## Core Rules

- Prefer updating existing pages over creating duplicates.
- Assign every source and durable page to a domain.
- If the source does not fit an existing domain, create or propose a new domain.
- Every important concept should either link to an existing page or get a new page.
- Keep source summaries separate from durable concept knowledge.
- Preserve uncertainty and conflicts.
- Do not turn a promotional claim into a fact.
- Keep Obsidian links readable.
- Update `index.md`.
- Append to `log.md`; do not rewrite old logs.

## Workflow

1. Read the normalized source.
2. Determine the domain first, such as `Linux`, `AI`, `网络工程`, `量化投资`, `自动化运维`, or `个人知识管理`.
3. Identify metadata, summary, claims, concepts, entities, procedures, commands, examples, risks, and caveats.
4. Create or update one note in `wiki/sources/`.
5. Create or update durable notes in `wiki/domains/`, `wiki/concepts/`, `wiki/entities/`, and `wiki/topics/`.
6. Link related pages using Obsidian wiki links.
7. Update `wiki/index.md`.
8. Append to `wiki/log.md`.

## Source Note Template

```markdown
---
title:
type: source
domain:
source_type:
source_url:
author:
published:
captured:
processed:
status: processed
tags: []
---

# Title

## Summary

What this source says.

## Key Claims

- Claim from the source.

## Extracted Concepts

- [[Concept]]

## Entities

- [[Entity]]

## Procedures

### Procedure

1. Step

## Technical Details

- Commands, code, versions, metrics, tools.

## Caveats

- Doubts, conflicts, missing context, promotional claims.

## Related

- [[Related Page]]
```

## Concept Note Template

```markdown
---
title:
type: concept
domain:
updated:
confidence: medium
tags: []
---

# Concept

## Summary

Durable explanation of the concept.

## Core Idea

The essential idea.

## Details

- Detail

## Examples

- Example

## Procedures

1. Step

## Risks and Caveats

- Risk or limitation.

## Sources

- [[Source Note]]

## Related

- [[Related Concept]]
```

## Entity Note Template

```markdown
---
title:
type: entity
entity_type:
updated:
tags: []
---

# Entity

## Summary

## Related Concepts

- [[Concept]]

## Sources

- [[Source Note]]
```

## Topic Note Template

```markdown
---
title:
type: topic
updated:
tags: []
---

# Topic

## Overview

## Key Concepts

- [[Concept]]

## Sources

- [[Source Note]]
```

## Domain Note Template

```markdown
---
title:
type: domain
updated:
tags: []
---

# Domain

## Scope

What belongs in this domain.

## Key Topics

- [[Topic]]

## Key Concepts

- [[Concept]]

## Sources

- [[Source Note]]
```

## Linking Rules

- Use `[[Page Name]]` for internal links.
- Link the first meaningful mention of important concepts.
- Avoid linking generic words.
- Use stable, human-readable filenames.
- Prefer filenames that exactly match the page title, such as `Browser First.md`, not slug filenames such as `browser-first.md`.
- If a source title contains square brackets or other link-hostile characters, create a clean source filename and title alias. Example: use `Linux 从 Terminal First 到 Browser First 的 Linux 工作流转变.md` instead of `[linux] 从 Terminal First 到 Browser First 的 Linux 工作流转变.md`.
- Every `[[Wiki Link]]` must resolve to an actual filename stem in the vault unless it is an intentional future stub.
- If a page does not exist but should, create it.

## Source Filename Rules

Source notes in `wiki/sources/` should use the clean semantic title from the normalized raw file, but should normally drop the date prefix and `.normalized` suffix.

Rules:

- If raw file is `YYYY-MM-DD-清晰中文标题.normalized.md`, source note should be `清晰中文标题.md`.
- If the raw title contains English, translate the general title into Simplified Chinese while preserving technical terms.
- Remove abnormal filename characters and category brackets.
- Do not put date prefixes on durable concept/entity/topic pages.
- Durable pages should be named by the concept itself, such as `Browser First.md`, `Terminal First.md`, `RAG.md`, or `最大回撤.md`.

Bad:

```markdown
concepts/browser-first.md
[[Browser First]]
[[[linux] Title]]
sources/2026-05-13-[linux] Title.normalized.md
```

Good:

```markdown
concepts/Browser First.md
[[Browser First]]
[[Linux 从 Terminal First 到 Browser First 的 Linux 工作流转变]]
sources/Linux 从 Terminal First 到 Browser First 的工作流转变.md
```

## Conflict Rules

If sources disagree, do not merge them silently.

Use:

```markdown
## Conflicts

- [[Source A]] says ...
- [[Source B]] says ...
- Current status: unresolved
```

## Index Rules

`wiki/index.md` should contain:

```markdown
# Knowledge Index

## Domains

- [[Linux]]

## Topics

- [[Topic]]

## Concepts

- [[Concept]]

## Entities

- [[Entity]]

## Sources

- [[Source Note]]

## Recently Processed

- YYYY-MM-DD - [[Source Note]]
```

## Log Rules

Append one entry per processed source:

```markdown
## YYYY-MM-DD

- Processed: `raw/.../source.md`
- Created: [[New Page]]
- Updated: [[Existing Page]]
- Caveats: ...
```

Date rules:

- Use the current local date provided by the runtime/user environment for `processed`, `updated`, and `log.md` entries.
- Never write a future date unless the user explicitly provided that future date.
- If reprocessing an existing source on the same day, append a short `(reprocess)` marker instead of creating a misleading new date.

## Completion Checklist

Before finishing:

- There is a source note.
- There is a domain assignment.
- Important concepts/entities/topics are linked.
- Existing pages were updated where appropriate.
- `index.md` was updated.
- `log.md` was appended.
- Unverified claims remain marked as source claims or caveats.
