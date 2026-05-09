---
title: 宇宙层级（Universes）
created: 2026-05-04
updated: 2026-05-09
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

## 项目实践

### 06-typed-db — DBType.asType 宇宙模式

用宇宙模式将数据库类型编码映射到 Lean 真实类型，避免直接在 `Type` 上做模式匹配：

```lean
inductive DBType where | int | string | bool
abbrev DBType.asType : DBType → Type
  | .int => Int | .string => String | .bool => Bool
```

`asType` 作为解释函数，让依赖类型的返回类型可以依赖编码值（如 `DBType.int`），返回对应的 Lean 类型。

### 08-type-checker — Ty.asType 宇宙模式

同样的模式用于类型检查器，`fn` 的情况涉及函数类型：

```lean
inductive Ty where | nat : Ty | bool : Ty | fn : Ty → Ty → Ty
abbrev Ty.asType : Ty → Type
  | .nat => Nat | .bool => Bool | .fn a b => a.asType → b.asType
```

`.fn a b => a.asType → b.asType` 将编码的函数类型映射到 Lean 真正的函数类型，让 `eval` 的返回类型可以依赖 `ty` 参数。

## 相关概念

- [[dependent-types]] — 参数 vs 索引的宇宙影响
- [[functor-applicative-monad]] — 标准库的宇宙多态定义
- [[evaluation-and-types]] — 类型基础
