---
title: 归纳数据类型与模式匹配
created: 2026-05-04
updated: 2026-05-09
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

## 项目实践

所有 8 个项目都使用了归纳数据类型和模式匹配。

| 项目 | 用法 |
|------|------|
| 01-calc | `Expr` 归纳类型，模式匹配求值 |
| 02-json-parser | `JValue` 归纳类型，模式匹配解析/查询 |
| 03-grep | `Config` 结构体模式匹配，`Except` 分支处理 |
| 04-eval-prove | `Expr` 归纳类型，`simplify` 模式匹配 |
| 05-safe-sort | `Ord.compare` 结果模式匹配，`Fin` 模式匹配 |
| 06-typed-db | `DBType`、`HasColumn`、`Subschema`、`Query` 等索引类型族 |
| 07-formal-verify | `insert`/`insertionSort` 模式匹配，策略证明中 `split` |
| 08-type-checker | `Expr`、`Ty` 归纳类型，`CheckResult` 依赖类型模式匹配 |

**01-calc — 表达式 AST（Calc/Expr.lean）**

```lean
inductive Expr where
  | num : Int → Expr
  | add : Expr → Expr → Expr
  | sub : Expr → Expr → Expr
  | mul : Expr → Expr → Expr
  | div : Expr → Expr → Expr
  | neg : Expr → Expr
deriving Repr, BEq, Nonempty
```

**08-type-checker — 依赖类型的检查结果（TypeChecker/Typecheck.lean）**

```lean
inductive CheckResult : Context → Expr → Ty → Type where
  | num : (n : Nat) → CheckResult ctx (.num n) .nat
  | bool : (b : Bool) → CheckResult ctx (.bool b) .bool
  | var : HasType ctx x ty → CheckResult ctx (.var x) ty
  | add :
      CheckResult ctx a .nat →
      CheckResult ctx b .nat →
      CheckResult ctx (.add a b) .nat
  | lam :
      CheckResult (ctx.bind x ty) body retTy →
      CheckResult ctx (.lam x ty body) (.fn ty retTy)
  | app :
      CheckResult ctx f (.fn argTy retTy) →
      CheckResult ctx arg argTy →
      CheckResult ctx (.app f arg) retTy
```

## 相关概念

- [[evaluation-and-types]] — 类型和求值
- [[polymorphism]] — 多态类型
- [[dependent-types]] — 依赖类型（类型中包含程序）
- [[tactics-and-induction]] — 归纳证明
