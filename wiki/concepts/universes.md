---
title: 宇宙层级（Universes）
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [universe, types]
sources: [book/FPLean/FunctorApplicativeMonad/Universes.lean]
---

# 宇宙层级（Universes）

## 核心思想

为了避免 Girard/Russell 悖论（`Type : Type` 导致逻辑不一致），Lean 使用**宇宙层级**：`Type : Type 1 : Type 2 : ...`

## 完整层级

```
Sort 0 = Prop          -- 命题宇宙
Sort 1 = Type 0 = Type -- 常规类型
Sort 2 = Type 1        -- 类型的类型
Sort 3 = Type 2        -- ...
```

## 宇宙多态

用变量 `u` 表示宇宙层级，一份定义适配所有层级：

```lean
inductive MyList (α : Type u) : Type u where
  | nil : MyList α
  | cons : α → MyList α → MyList α

inductive Sum (α : Type u) (β : Type v) : Type (max u v) where
  | inl : α → Sum α β
  | inr : β → Sum α β
```

## 关键规则

- `Prop` 的特殊性：函数返回 `Prop` 时，整个函数类型在 `Prop` 中
- 参数不增加宇宙层级，索引增加 `+1`
- 多宇宙参数取 `max u v`

## 相关概念

- [[dependent-types]] — 参数 vs 索引的宇宙影响
- [[functor-applicative-monad]] — 标准库的宇宙多态定义
- [[evaluation-and-types]] — 类型基础
