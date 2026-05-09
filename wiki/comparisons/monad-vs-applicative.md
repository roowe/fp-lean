---
title: Monad vs Applicative vs Functor
created: 2026-05-04
updated: 2026-05-09
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

## 项目实践

> 02-json-parser 项目中 `Validate` 是本系列唯一「Applicative 但非 Monad」的类型，完美展示了两者的区别。

`Validate` 用 Applicative 的 `seq` 累积所有错误（不短路），这用 Monad 的 `bind` 无法实现——`bind` 要求看到前步结果才能决定后续计算，而错误累积需要**同时**收集所有错误：

```lean
-- 02-json-parser/JsonParser/Validate.lean
instance : Applicative (Validate ε) where
  seq f x := match f with
    | .ok g => g <$> (x ())
    | .errors errs => match x () with
      | .ok _ => .errors errs           -- 只保留函数的错误
      | .errors errs' => .errors (errs ++ errs')  -- 合并双方的错误
```

相比之下，01-calc 和 04-eval-prove 中的 `Except` 是 Monad——遇到第一个错误就停止（短路），适合**只需要第一个错误**的场景。

| 项目 | 类型 | 层级 | 用途 |
|------|------|------|------|
| 01-calc | `Except String` | Monad | 除零错误，遇到即停 |
| 02-json-parser | `Validate String` | Applicative（非 Monad） | 解析错误累积 |
| 03-grep | `ReaderT + ExceptT` | Monad 变换器栈 | 配置传递 + 错误处理 |
| 04-eval-prove | `Except String` | Monad | 未绑定变量错误 |

## 相关概念

- [[functor-applicative-monad]] — 完整层级定义
- [[monads]] — Monad 详解
- [[type-classes]] — 类型类继承机制
