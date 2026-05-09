# Wiki Index

> 内容目录。每个 wiki 页面按类型分类，附一行摘要。
> 阅读此文件快速定位相关页面。
> Last updated: 2026-05-04 | Total pages: 20

## Concepts
<!-- 按学习顺序排列（依赖关系从上到下） -->
<!-- 频率标记：🔥高频(7+/11) · 📦常规(3-6/11) · 🧊低频(1-2/11) · ❄️未涉及(0/11) -->
<!-- 依据：01-calc, 01-calc-ds, 02-json-parser-kiro/gpt/ds, 03-grep, 04-eval-prove, 05-safe-sort, 06-typed-db, 07-formal-verify, 08-type-checker 共 11 个项目版本 -->

- [[evaluation-and-types]] — ❄️未涉及(0/11) · 求值、类型基础、`#eval`/`#check`、类型层级
- [[functions-and-definitions]] — 🧊低频(3/11) · `def`、柯里化、匿名函数、局部定义
- [[structures]] — 📦常规(5/11) · 结构体定义、字段访问、函数式更新、结构体继承
- [[datatypes-pattern-matching]] — 🔥高频(11/11) · 归纳类型、模式匹配、结构递归、终止性
- [[polymorphism]] — 📦常规(6/11) · 参数化多态、隐式参数、`List`/`Option`/`Prod`/`Sum`
- [[io-and-side-effects]] — 📦常规(6/11) · IO monad、do 记法、求值 vs 执行、嵌套动作
- [[propositions-and-proofs]] — 📦常规(4/11) · 命题即类型、策略模式、逻辑连接词、安全索引
- [[type-classes]] — 🔥高频(9/11) · 类型类机制、实例搜索、GetElem、强制转换、自动派生
- [[monads]] — 🔥高频(8/11) · Monad 类型类、常见实例、bind/pure、多态求值器
- [[functor-applicative-monad]] — 📦常规(3/11) · Functor/Applicative/Monad 层级、Validate、Alternative
- [[monad-transformers]] — 🧊低频(1/11) · 变换器工具箱、堆叠顺序、命令式 do 特性
- [[dependent-types]] — 🧊低频(2/11) · 索引类型族、宇宙模式、Vect、计算卡住
- [[tactics-and-induction]] — 📦常规(4/11) · 归纳策略、策略速查、grind、函数归纳
- [[tail-recursion]] — ❄️未涉及(0/11) · 累加器传递风格、尾调用消除、等价性证明
- [[termination-proofs]] — 🧊低频(2/11) · termination_by、have 表达式、Fin 安全索引、sorry
- [[universes]] — 🧊低频(1/11) · 宇宙层级、宇宙多态、Sort、Girard 悖论

## Entities

- [[list-type]] — List 列表类型，`[1, 2, 3]` 语法糖
- [[option-type]] — Option 可选类型，`some`/`none`，安全查找
- [[prod-type]] — Prod 积类型 `α × β`，配对值
- [[sum-type]] — Sum 和类型 `α ⊕ β`，二选一

## Comparisons

- [[monad-vs-applicative]] — Functor/Applicative/Monad 能力对比、设计原则
- [[definitional-vs-propositional-equality]] — 定义相等 vs 命题相等、计算卡住问题

## Queries
<!-- 暂无 -->
