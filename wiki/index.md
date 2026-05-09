# Wiki Index

> 内容目录。每个 wiki 页面按类型分类，附一行摘要。
> 阅读此文件快速定位相关页面。
> Last updated: 2026-05-09 | Total pages: 25

## 项目一览

> 8 个实战项目，从计算器到类型检查器，逐步深入 FP + Lean。

| # | 项目 | 核心主题 | 关键技术 |
|---|------|---------|---------|
| 01 | [命令行计算器](https://github.com/DrumstickLab/lean-fp-demos/tree/main/01-calc) | 表达式解析与求值 | 归纳类型、Except 单子、递归下降解析器 |
| 02 | [JSON 解析器](https://github.com/DrumstickLab/lean-fp-demos/tree/main/02-json-parser) | 解析 + 错误累积 | Validate Applicative、mutual recursion、fuel 模式 |
| 03 | [小型 grep](https://github.com/DrumstickLab/lean-fp-demos/tree/main/03-grep) | 文件搜索工具 | ReaderT + ExceptT 变换器栈、IO |
| 04 | [表达式求值器 + 证明](https://github.com/DrumstickLab/lean-fp-demos/tree/main/04-eval-prove) | 求值与化简正确性 | 表达式化简、induction + grind 证明 |
| 05 | [安全数组排序](https://github.com/DrumstickLab/lean-fp-demos/tree/main/05-safe-sort) | 终止性 + 索引安全 | Fin、termination_by、fun_induction |
| 06 | [带类型的迷你数据库](https://github.com/DrumstickLab/lean-fp-demos/tree/main/06-typed-db) | 编译期类型安全 | 索引类型族、宇宙模式、依赖类型 |
| 07 | [形式化验证练习](https://github.com/DrumstickLab/lean-fp-demos/tree/main/07-formal-verify) | 定理证明实战 | 列表性质、排序性质、字符串性质 |
| 08 | [类型检查器](https://github.com/DrumstickLab/lean-fp-demos/tree/main/08-type-checker) | STLC 类型推导 | HasType 索引族、CheckResult、依赖模式匹配 |

## Concepts
<!-- 按学习顺序排列（依赖关系从上到下） -->
<!-- 频率标记：🔥高频(6+/8) · 📦常规(3-5/8) · 🧊低频(1-2/8) · ❄️未涉及(0/8) -->
<!-- 依据 8 个主要项目：01-calc, 02-json-parser, 03-grep, 04-eval-prove, 05-safe-sort, 06-typed-db, 07-formal-verify, 08-type-checker -->

- [[datatypes-pattern-matching]] — 🔥高频(8/8) · 归纳类型、模式匹配、结构递归、终止性
- [[functions-and-definitions]] — 📦常规(7/8) · `def`、柯里化、匿名函数、局部定义
- [[type-classes]] — 📦常规(5/8) · 类型类机制、实例搜索、Ord、deriving、BEq
- [[polymorphism]] — 📦常规(5/8) · 参数化多态、隐式参数、`List`/`Option`/`Prod`/`Sum`
- [[io-and-side-effects]] — 📦常规(5/8) · IO monad、do 记法、求值 vs 执行、嵌套动作
- [[structures]] — 📦常规(4/8) · 结构体定义、字段访问、函数式更新、ParseState/Config
- [[monads]] — 📦常规(4/8) · Monad 类型类、Except/Option 实例、bind/pure、do 记法
- [[propositions-and-proofs]] — 📦常规(4/8) · 命题即类型、策略模式、逻辑连接词、编译期检查
- [[tactics-and-induction]] — 📦常规(3/8) · 归纳策略、grind、split、函数归纳
- [[evaluation-and-types]] — ❄️未涉及(0/8) · 求值、类型基础、`#eval`/`#check`、类型层级
- [[parsing]] — 🧊低频(2/8) · 递归下降、mutual recursion、fuel 模式、Validate 错误累积
- [[functor-applicative-monad]] — 🧊低频(1/8) · Functor/Applicative/Monad 层级、Validate 实例
- [[dependent-types]] — 🧊低频(2/8) · 索引类型族、宇宙模式、HasColumn/HasType、证明作为参数
- [[universes]] — 🧊低频(2/8) · 宇宙层级、宇宙多态、DBType.asType/Ty.asType
- [[monad-transformers]] — 🧊低频(1/8) · ReaderT + ExceptT 堆叠、.run 解包
- [[termination-proofs]] — 🧊低频(1/8) · termination_by、have 表达式、Fin 安全索引
- [[tail-recursion]] — ❄️未涉及(0/8) · 累加器传递风格、尾调用消除、等价性证明

## Entities

- [[list-type]] — List 列表类型，`[1, 2, 3]` 语法糖
- [[option-type]] — Option 可选类型，`some`/`none`，安全查找
- [[prod-type]] — Prod 积类型 `α × β`，配对值
- [[sum-type]] — Sum 和类型 `α ⊕ β`，二选一
- [[array-type]] — Array 动态数组，`#[1, 2, 3]`，for 循环
- [[string-type]] — String 字符串操作，`toList`/`ofList`，`splitOn`，转义序列

## Comparisons

- [[monad-vs-applicative]] — Functor/Applicative/Monad 能力对比、Validate 不是 Monad
- [[definitional-vs-propositional-equality]] — 定义相等 vs 命题相等、计算卡住问题

## Queries

- [[project-pitfalls]] — 8 个项目的难点分析与钻研指南（按难度排序）
