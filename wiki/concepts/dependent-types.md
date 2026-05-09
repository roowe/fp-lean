---
title: 依赖类型（Dependent Types）
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [dependent-types, indexed-family, universe]
sources: [book/FPLean/DependentTypes/IndexedFamilies.lean, book/FPLean/DependentTypes/UniversePattern.lean, book/FPLean/DependentTypes/TypedQueries.lean, book/FPLean/DependentTypes/IndicesParametersUniverses.lean, book/FPLean/DependentTypes/Pitfalls.lean]
---

# 依赖类型（Dependent Types）

## 核心思想

在大多数语言中，类型和程序是严格分离的。Lean 打破这一限制——**程序可以计算类型，类型可以包含程序**。依赖类型让类型系统能表达远超常规的约束。

## 索引类型族（Indexed Families）

类型的构造器可以接受**值**作为参数，而不仅仅是类型：

```lean
inductive Vect (α : Type u) : Nat → Type u where
  | nil : Vect α 0
  | cons : α → Vect α n → Vect α (n + 1)
```

`Vect` 的第二个参数是 `Nat`（值，不是类型）——**长度编码在类型中**。

### 类型保证正确性

```lean
def Vect.zip : Vect α n → Vect β n → Vect (α × β) n
  | .nil, .nil => .nil
  | .cons x xs, .cons y ys => .cons (x, y) (zip xs ys)
```

两个参数**必须长度相同**——编译期保证，无需运行时检查。

## 宇宙模式（Universe Pattern）

用自定义的"编码"类型代表一个类型子集，通过解释函数映射到真实类型：

```lean
inductive NatOrBool where | nat | bool
abbrev NatOrBool.asType (code : NatOrBool) : Type :=
  match code with | .nat => Nat | .bool => Bool
```

这允许在依赖类型中引用"类型的类型"而不触及 [[universes]] 层级问题。

## 参数 vs 索引

| 位置 | 名称 | 特性 |
|------|------|------|
| 冒号前 | **参数** | 不增加宇宙层级，定义相等性自动适用 |
| 冒号后 | **索引** | 增加 `+1` 宇宙层级，需要依赖模式匹配 |

```lean
inductive WithParam (α : Type u) : Type u where ...    -- 参数
inductive WithIdx : Type u → Type (u + 1) where ...    -- 索引
```

## 陷阱：计算卡住

依赖类型编程最大的挑战是**类型计算可能卡住**：

```lean
-- plusL 对第一个参数做模式匹配——能用在类型中
def Nat.plusL : Nat → Nat → Nat | 0, k => k | n+1, k => plusL n k + 1

-- plusR 对第二个参数做模式匹配——用在类型中会卡住！
def Nat.plusR : Nat → Nat → Nat | n, 0 => n | n, k+1 => plusR n k + 1
```

当模式匹配在变量处卡住时，需要**命题相等性证明**来"解卡"：

```lean
theorem plusR_zero_left : (k : Nat) → k = Nat.plusR 0 k := by
  induction k <;> simp [Nat.plusR] <;> assumption

-- 使用 ▸ 操作符部署证明
def appendR : Vect α n → Vect α k → Vect α (n.plusR k)
  | .nil, ys => plusR_zero_left _ ▸ ys
  | .cons x xs, ys => plusR_succ_left _ _ ▸ .cons x (appendR xs ys)
```

### 定义相等 vs 命题相等

| 概念 | 说明 | 例子 |
|------|------|------|
| **定义相等** | 编译器自动识别 | `1 + 1` 和 `2` |
| **命题相等** | 需要显式证明 | `k = Nat.plusR 0 k` |

## 高级模式：证明作为参数

依赖类型可以将正确性证明编码在类型中，编译期保证程序正确：

### 证明作为构造子参数

```lean
-- Subschema 的 cons 需要证明"列存在于上层 Schema"
inductive Subschema : Schema → Schema → Type where
  | cons : HasColumn n t super → Subschema sub super → Subschema (⟨n, t⟩ :: sub) super

-- Disjoint 的 cons 需要证明"列名不重叠"
inductive Disjoint : Schema → Schema → Type where
  | cons : NameNotIn n s2 → Disjoint s1 s2 → Disjoint (⟨n, t⟩ :: s1) s2
```

调用时必须同时提供正确性证明，否则编译不过。

### 依赖类型求值器

```lean
-- eval 的返回类型 ty.asType 依赖参数 ty
def eval : (ctx : Context) → (e : Expr) → (ty : Ty) →
    CheckResult ctx e ty → Env ctx → ty.asType
-- 传入 .nat 返回 Nat，传入 .bool 返回 Bool
```

只有通过类型检查（持有 `CheckResult`）的项才能被求值，从架构上杜绝类型错误。

### 依赖类型环境

```lean
-- ty.asType 出现在构造子参数里
inductive Env : Context → Type where
  | cons : (ty : Ty) → ty.asType → Env ctx → Env ((x, ty) :: ctx)
```

用 `HasType` 证明作为"钥匙"安全取值：

```lean
def lookup : HasType ctx x ty → Env ctx → ty.asType
```

## 相关概念

- [[polymorphism]] — 参数化多态（依赖类型的基础版）
- [[universes]] — 宇宙层级
- [[propositions-and-proofs]] — 命题与证明基础
- [[tactics-and-induction]] — 证明策略
- [[definitional-vs-propositional-equality]] — 计算卡住的深层原因
