# Wiki Schema

## Domain
Functional Programming in Lean — 基于《Functional Programming in Lean》书籍的知识库，涵盖 Lean 语言、函数式编程、类型系统、依赖类型、定理证明。

## Conventions
- 文件名：小写、连字符、无空格（如 `dependent-types.md`）
- 每个 wiki 页面以 YAML frontmatter 开头
- 使用 `[[wikilinks]]` 链接页面（每页至少 2 个出站链接）
- 更新页面时务必更新 `updated` 日期
- 新页面必须添加到 `index.md` 对应分类下
- 每次操作追加到 `log.md`
- 页面语言：中文为主，代码/术语保留英文原文

## Frontmatter
```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept | entity | comparison | query
tags: [从下方标签体系选取]
sources: [raw/articles/source-name.md 或 book 章节]
---
```

## Tag Taxonomy
- 语言基础: lean-syntax, types, functions, pattern-matching, polymorphism, structures
- 函数式编程: functor, applicative, monad, monad-transformer, do-notation, purity
- 类型系统: type-class, dependent-types, universe, indexed-family, coercion
- 证明: proposition, proof, tactic, induction, termination, equivalence
- 编程实践: io, error-handling, array, recursion, tail-recursion, performance
- 工具链: lake, elan, lean-lsp
- 元数据: comparison, summary, getting-started

## Page Thresholds
- **创建页面**：核心概念（类型、单子、依赖类型等）或出现 2+ 次的实体
- **不创建页面**：一笔带过的细节、书外的内容
- **拆分页面**：超过 200 行时拆分为子主题
- **归档页面**：内容完全过时时移入 `_archive/`

## Page Types
- **Concept 页面**：一个概念一个页面，含定义、关键代码、与其他概念的关系
- **Entity 页面**：Lean 标准库关键类型/函数（如 `Nat`、`IO`、`Except`）
- **Comparison 页面**：概念对比（如 Monad vs Applicative）
- **Query 页面**：有价值的问答和深入分析

## Update Policy
- 新信息与已有内容矛盾时，保留两个版本并标注来源和日期
- 在 frontmatter 中标记 `contradictions`
- 标记供用户审核
