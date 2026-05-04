---
title: Monad vs Applicative vs Functor
created: 2026-05-04
updated: 2026-05-04
type: comparison
tags: [functor, applicative, monad, comparison]
sources: [book/FPLean/FunctorApplicativeMonad/ApplicativeContract.lean]
---

# Monad vs Applicative vs Functor

## 能力对比

| 维度 | Functor | Applicative | Monad |
|------|---------|-------------|-------|
| 核心操作 | `map : (α→β) → f α → f β` | `pure + seq` | `bind` |
| 纯值注入 | 无 | `pure` | `pure` |
| 包裹函数应用 | 无 | `<*>` | 通过 `bind` |
| 前步影响后步 | 不能 | 不能 | 能 |
| 法则数 | 2 | 4 | 3 |
| 可用实例数 | 最多 | 中等 | 最少 |

## 关键区别：前步能否影响后步

**Functor/Applicative**：所有计算的结构在开始前就确定了。

```lean
-- Applicative: 两个独立计算
pure f <*> x <*> y
-- x 和 y 的选择不影响彼此
```

**Monad**：前一步的值决定后续计算的选择。

```lean
-- Monad: 第二步依赖第一步的结果
x >>= fun v => if v > 0 then computeA v else computeB v
```

## 实例对比

| 类型 | Functor? | Applicative? | Monad? | 说明 |
|------|---------|-------------|--------|------|
| `Option` | 是 | 是 | 是 | 失败时短路 |
| `List` | 是 | 是 | 是 | 非确定性 |
| `Except ε` | 是 | 是 | 是 | 异常处理 |
| `Validate ε` | 是 | 是 | **否** | 累积所有错误（非短路） |
| `Pair α` | 是 | **否** | **否** | 无法提供合适的 pure |
| `IO` | 是 | 是 | 是 | 副作用序列化 |

## 设计原则

- **API 使用最弱抽象**：如果只需要 `map`，用 `Functor` 约束而非 `Monad`
- **实例提供最强实现**：如果类型可以成为 Monad，就提供 Monad 实例

## 相关概念

- [[functor-applicative-monad]] — 完整层级定义
- [[monads]] — Monad 详解
- [[type-classes]] — 类型类继承机制
