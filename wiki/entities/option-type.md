---
title: Option 类型
created: 2026-05-04
updated: 2026-05-04
type: entity
tags: [lean-syntax, types]
sources: [book/FPLean/GettingToKnow/Polymorphism.lean]
---

# Option 类型

## 定义

```lean
inductive Option (α : Type) where
  | none : Option α
  | some (val : α) : Option α
```

表示**可能缺失的值**。`some` 表示有值，`none` 表示无值。类似其他语言的可空类型，但允许多层嵌套。

## 常见用法

```lean
def thirdOption (xs : List α) : Option α := xs[2]?

-- 安全查找
def List.find? (p : α → Bool) (xs : List α) : Option α := do
  for x in xs do
    if p x then return x
  failure
```

## 相关概念

- [[polymorphism]] — 多态类型
- [[monads]] — Option 是 Monad 实例（失败效果）
- [[functor-applicative-monad]] — Alternative 实例
