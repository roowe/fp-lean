---
title: Sum 类型（和类型）
created: 2026-05-04
updated: 2026-05-04
type: entity
tags: [lean-syntax, types]
sources: [book/FPLean/GettingToKnow/Polymorphism.lean]
---

# Sum 类型（和类型）

## 定义

```lean
inductive Sum (α : Type) (β : Type) : Type where
  | inl : α → Sum α β
  | inr : β → Sum α β
```

语法糖：`α ⊕ β` 等价于 `Sum α β`。`inl`（左注入）和 `inr`（右注入）选择一侧。

## 常见用法

```lean
-- 表示两种类型的选择
def process : Sum String Nat → String
  | .inl s => s"Got string: {s}"
  | .inr n => s"Got number: {n}"
```

## 相关概念

- [[polymorphism]] — 多态类型
- [[prod-type]] — 积类型（配对）
- [[datatypes-pattern-matching]] — 模式匹配
