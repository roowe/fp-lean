---
title: Prod 类型（积类型）
created: 2026-05-04
updated: 2026-05-04
type: entity
tags: [lean-syntax, types]
sources: [book/FPLean/GettingToKnow/Polymorphism.lean]
---

# Prod 类型（积类型）

## 定义

```lean
structure Prod (α : Type) (β : Type) : Type where
  fst : α
  snd : β
```

语法糖：`α × β` 等价于 `Prod α β`，`("five", 5)` 创建 `Prod String Nat`。

## 常见用法

```lean
-- 模式匹配 pair
let (xs, ys) : List α × List β := unzip xys

-- 函数返回多个值
def unzip : List (α × β) → List α × List β
  | [] => ([], [])
  | (x, y) :: xys =>
    let (xs, ys) := unzip xys
    (x :: xs, y :: ys)
```

## 相关概念

- [[polymorphism]] — 多态类型
- [[sum-type]] — 和类型（二选一）
- [[structures]] — 结构体基础
