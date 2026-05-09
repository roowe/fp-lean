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

## 高阶方法

对列表中的每个元素施加操作，或对整个列表做聚合：

| 方法 | 类型 | 说明 |
|------|------|------|
| `map f` | `(α → β) → List β` | 对每个元素施加 `f` |
| `filter p` | `(α → Bool) → List α` | 保留满足 `p` 的元素 |
| `foldl f init` | `(β → α → β) → β → β` | 左折叠，从左到右累积 |
| `find? p` | `(α → Bool) → Option α` | 找第一个满足 `p` 的元素 |
| `any p` | `(α → Bool) → Bool` | 是否存在满足 `p` 的元素 |
| `contains x` | `[BEq α] → α → Bool` | 是否包含 `x` |
| `dropWhile p` | `(α → Bool) → List α` | 跳过开头连续满足 `p` 的元素 |
| `reverse` | `List α` | 反转列表 |

```lean
-- map：变换每个元素
[1, 2, 3].map (· + 1)           -- [2, 3, 4]

-- filter：保留满足条件的元素
[1, 2, 3, 4].filter (· > 2)     -- [3, 4]

-- foldl：左折叠累积
[1, 2, 3].foldl (· + ·) 0       -- 6

-- find?：找第一个
[1, 2, 3].find? (· > 2)         -- some 3

-- contains：成员检测
[1, 2, 3].contains 2            -- true

-- dropWhile：跳过开头空白
"  hello".toList.dropWhile Char.isWhitespace  -- ['h', 'e', 'l', 'l', 'o']
```

## 相关概念

- [[polymorphism]] — 多态类型
- [[datatypes-pattern-matching]] — 模式匹配
- [[functor-applicative-monad]] — Functor 实例
