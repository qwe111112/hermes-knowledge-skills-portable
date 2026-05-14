---
name: knowledge-query-assistant
description: Use this skill when Hermes must answer questions using an existing Obsidian knowledge wiki produced by the knowledge pipeline. It retrieves relevant wiki notes, builds a grounded context bundle, answers with wiki links and source awareness, and clearly separates known facts from inference.
---

# Knowledge Query Assistant

## Trigger Conditions

Use this skill when the user asks Hermes to:

- 根据我的知识库回答
- 查询我的 Obsidian Wiki
- 根据已有知识沉淀总结
- 比较两个概念并引用我的资料
- 从 wiki 中找答案
- answer using my knowledge base
- query the existing wiki

Do not use this skill to ingest or rewrite documents. Use:

- `markdown-source-compiler` for messy Markdown cleanup.
- `wiki-maintainer` for writing or updating wiki pages.

## Portable Configuration

This skill must be portable across machines. Do not hard-code absolute paths.

Before answering, establish:

```text
KNOWLEDGE_ROOT = the Obsidian knowledge folder provided by the user
```

If missing, ask the user for it.

Expected layout:

```text
{KNOWLEDGE_ROOT}/
├── raw/
└── wiki/
    ├── index.md
    ├── log.md
    ├── domains/
    ├── sources/
    ├── concepts/
    ├── entities/
    └── topics/
```

Default search scope:

```text
{KNOWLEDGE_ROOT}/wiki/
```

Use `raw/` only when a wiki source note points to a raw file and the answer needs deeper evidence.

## Retrieval Policy

Use this order:

1. Read `wiki/index.md` to understand available domains, topics, concepts, entities, and sources.
2. Search filenames under `wiki/` for direct matches.
3. Search note contents under `wiki/` for query keywords and related terms.
4. Open the most relevant notes.
5. Follow high-value links from those notes, especially `Sources`, `Related`, `Extracted Concepts`, `Entities`, and `Topics`.
6. If facts are uncertain, open related source notes under `wiki/sources/`.
7. Use `raw/` only if the wiki source note is insufficient.

Preferred context bundle:

- 1 to 3 concept/topic/domain notes
- 1 to 3 source notes
- 0 to 2 entity notes

Avoid loading the whole vault unless the user explicitly asks for exhaustive analysis.

## Query Expansion

When searching, expand likely variants:

- English and Chinese terms: `RAG` / `检索增强生成`
- Project names and aliases: `vLLM` / `vllm`
- Concepts and implementations: `PagedAttention` / `KV Cache`
- Acronyms and full names: `PD 分离` / `prefill decode disaggregation`

Do not invent aliases. If unsure, search conservative variants only.

## Answer Rules

Answers must be grounded in the wiki.

Required behavior:

- Start from what the knowledge base contains.
- Cite relevant wiki pages using Obsidian links, such as `[[vLLM]]` or `[[分布式 LLM 推理]]`.
- Distinguish:
  - `知识库已有`
  - `根据现有笔记推断`
  - `知识库暂未覆盖`
- Preserve caveats and uncertainty.
- If sources conflict, explain the conflict instead of hiding it.
- If the wiki lacks enough evidence, say so and suggest what source should be added.

Do not:

- Pretend the knowledge base contains information it does not contain.
- Answer only from general model memory when the user asked to use the knowledge base.
- Over-cite every sentence.
- Copy large passages from source notes.

## Answer Format

For short factual questions:

```markdown
根据当前知识库，...

来源：[[Page A]], [[Page B]]
```

For comparison questions:

```markdown
## 结论

...

## 对比

| 维度 | A | B |
|---|---|---|

## 依据

- [[Page A]]
- [[Page B]]

## 不确定点

- ...
```

For troubleshooting or technical guidance:

```markdown
## 建议

1. ...

## 原因

...

## 相关知识

- [[Concept]]
- [[Source]]

## 风险

- ...
```

## Quality Checklist

Before answering:

- Relevant wiki notes were searched.
- At least one source/concept/topic page was checked when available.
- The answer includes wiki links for important claims.
- Missing knowledge is explicitly marked.
- Inference is labeled as inference.
- No unsupported claim is presented as a wiki-backed fact.

## Optional Obsidian Integration

If the user has Obsidian Local REST API configured and provides endpoint/key, Hermes may query Obsidian through that API.

MVP default:

- Use filesystem access to read Markdown files under `{KNOWLEDGE_ROOT}/wiki/`.
- Do not require Obsidian to be running.

