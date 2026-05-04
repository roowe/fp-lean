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
