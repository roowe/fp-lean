---
title: List 类型
created: 2026-05-04
updated: 2026-05-04
type: entity
tags: [lean-syntax, types]
sources: [book/FPLean/GettingToKnow/Polymorphism.lean]
---

# List 类型

## 定义

```lean
inductive List (α : Type) where
  | nil : List α
  | cons : α → List α → List α
```

语法糖：`[1, 2, 3]` 等价于 `List.cons 1 (List.cons 2 (List.cons 3 List.nil))`

## 常见操作

```lean
def length (xs : List α) : Nat :=
  match xs with
  | [] => 0
  | y :: ys => Nat.succ (length ys)

def mapM [Monad m] (f : α → m β) : List α → m (List β)
  | [] => pure []
  | x :: xs => do
    pure ((← f x) :: (← mapM f xs))
```

## 相关概念

- [[polymorphism]] — 多态类型
- [[datatypes-pattern-matching]] — 模式匹配
- [[functor-applicative-monad]] — Functor 实例
