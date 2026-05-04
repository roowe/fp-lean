---
title: 归纳数据类型与模式匹配
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [pattern-matching, lean-syntax, getting-started]
sources: [book/FPLean/GettingToKnow/DatatypesPatterns.lean]
---

# 归纳数据类型与模式匹配

## 核心思想

归纳数据类型（inductive datatype）是 Lean 表达数据的核心机制。它同时支持"选择"（多个构造器）和"递归"（类型引用自身）。消费数据的方式（模式匹配）与创建数据的方式（构造器）一一对应。

## 归纳类型定义

```lean
inductive Bool where
  | false : Bool
  | true : Bool

inductive Nat where
  | zero : Nat
  | succ (n : Nat) : Nat
```

### 三种类型分类

| 类型 | 构造器数 | 递归 | 例子 |
|------|---------|------|------|
| 积类型（product） | 1 | 否 | [[structures]] |
| 和类型（sum） | 多 | 否 | `Bool` |
| 归纳类型 | 多 | 是 | `Nat`, [[list-type]], [[option-type]] |

## 模式匹配

通过检查构造器来分解数据，同时提取字段：

```lean
def isZero (n : Nat) : Bool :=
  match n with
  | Nat.zero => true
  | Nat.succ k => false
```

### 模式匹配定义（便捷语法）

将参数移至冒号右侧，直接列出各分支：

```lean
def drop : Nat → List α → List α
  | Nat.zero, xs => xs
  | _, [] => []
  | Nat.succ n, x :: xs => drop n xs
```

### 自然数模式

```lean
def length (xs : List α) : Nat :=
  match xs with
  | [] => 0
  | y :: ys => Nat.succ (length ys)
```

自然数模式用字面量 `0` 和 `n + 1` 代替 `Nat.zero` / `Nat.succ`。

### 或模式（Or-patterns）

多个模式共享同一结果：

```lean
def isWeekend : Weekday → Bool
  | .saturday | .sunday => true
  | _ => false
```

## 结构递归与终止性

Lean **要求所有递归函数终止**。递归调用必须在结构更小的输入上进行：

```lean
def even (n : Nat) : Bool :=
  match n with
  | Nat.zero => true
  | Nat.succ k => not (even k)    -- k 比 n 更小
```

不满足终止性检查时，可标记 `partial`（但牺牲形式化证明能力）。详见 [[tail-recursion]] 和 [[termination-proofs]]。

## 内置多态类型

```lean
-- 列表
inductive List (α : Type) where
  | nil : List α
  | cons : α → List α → List α

-- 可能缺失的值
inductive Option (α : Type) where
  | none : Option α
  | some (val : α) : Option α
```

详见 [[list-type]]、[[option-type]]、[[prod-type]]、[[sum-type]]。

## 相关概念

- [[evaluation-and-types]] — 类型和求值
- [[polymorphism]] — 多态类型
- [[dependent-types]] — 依赖类型（类型中包含程序）
- [[tactics-and-induction]] — 归纳证明
