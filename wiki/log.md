# Wiki Log

> 所有 wiki 操作的按时间记录。仅追加。
> 格式：`## [YYYY-MM-DD] action | subject`
> 操作类型：create, ingest, update, query, lint

## [2026-05-04] create | Wiki initialized
- Domain: Functional Programming in Lean
- Structure created with SCHEMA.md, index.md, log.md

## [2026-05-04] ingest | 全书 12 章批量录入
- 批量提取全书 12 章核心概念，创建 14 个概念页 + 2 个对比页
- 概念页：evaluation-and-types, functions-and-definitions, structures, datatypes-pattern-matching, polymorphism, io-and-side-effects, propositions-and-proofs, type-classes, monads, functor-applicative-monad, monad-transformers, dependent-types, tactics-and-induction, tail-recursion, termination-proofs, universes
- 对比页：monad-vs-applicative, definitional-vs-propositional-equality
- 所有页面间已建立 wikilinks 交叉引用
- Updated: index.md (total: 16 pages), log.md

## [2026-05-09] update | 基于 8 个实战项目源码全面整理 wiki
- 重读 8 个项目源码：01-calc, 02-json-parser, 03-grep, 04-eval-prove, 05-safe-sort, 06-typed-db, 07-formal-verify, 08-type-checker
- 重新统计概念频率（基于 8 个主要项目而非 11 个含变体版本）
- 更新 index.md：新增「项目一览」表格，频率标记改为 8 分制
- 为全部 17 个概念页添加「项目实践」章节，展示真实项目代码示例
- 为 2 个对比页更新项目实践（monad-vs-applicative 添加 Validate 示例）
- 为 array-type 实体页添加项目实践（05-safe-sort Fin + swap）
- 概念页 frontmatter updated 日期更新为 2026-05-09
- Updated: index.md, log.md, 全部 concept 页面, monad-vs-applicative.md, array-type.md (total: 25 pages)
