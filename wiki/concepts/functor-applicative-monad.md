---
title: Functor / Applicative / Monad 层级
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [functor, applicative, monad, type-class]
sources: [book/FPLean/FunctorApplicativeMonad/Inheritance.lean, book/FPLean/FunctorApplicativeMonad/Applicative.lean, book/FPLean/FunctorApplicativeMonad/ApplicativeContract.lean, book/FPLean/FunctorApplicativeMonad/Alternative.lean, book/FPLean/FunctorApplicativeMonad/Universes.lean, book/FPLean/FunctorApplicativeMonad/Complete.lean]
---

# Functor / Applicative / Monad 层级

## 核心思想

Functor、Applicative、Monad 形成能力递增的类型类层级。设计原则：**API 使用最弱抽象**（最大化适用范围），**实例提供最强实现**（最大化功能）。

## 层级关系

```
Functor          -- map
  ↑ extends
Applicative      -- pure + seq (<*>)
  ↑ extends
Monad            -- bind (>>=)
  ↑ extends
Alternative      -- failure + orElse (<|>)
```

每一层只需实现核心操作，其余通过默认实现自动派生。

## Functor（函子）

```lean
class Functor (f : Type u → Type v) where
  map : (α → β) → f α → f β
```

法则：
1. **恒等**：`map id = id`
2. **合成**：`map (g ∘ f) = map g ∘ map f`

运算符：`g <$> x` 等价于 `Functor.map g x`

## Applicative（应用函子）

```lean
class Applicative (f : Type → Type) extends Functor f where
  pure : α → f α
  seq : f (α → β) → (Unit → f α) → f β  -- 写作 <*>
```

关键：**函数本身也被包裹在类型中**，前一步的值**不能**影响后续计算的选择。

### Applicative 四法则

1. **恒等**：`pure id <*> v = v`
2. **复合**：`pure (· ∘ ·) <*> u <*> v <*> w = u <*> (v <*> w)`
3. **同态**：`pure f <*> pure x = pure (f x)`
4. **交换**：`u <*> pure x = pure (fun f => f x) <*> u`

### Validate — 累积错误的 Applicative

```lean
inductive Validate (ε α : Type) : Type where
  | ok : α → Validate ε α
  | errors : NonEmptyList ε → Validate ε α
```

Validate 是 Applicative 但**不是** Monad——因为它累积所有错误而非遇到第一个就停止。

## Monad（单子）

```lean
class Monad (m : Type → Type) extends Applicative m where
  bind : m α → (α → m β) → m β
```

Monad 允许前一步的值影响后续计算。只需实现 `bind` + `pure`，自动获得 `map`、`seq` 等。

### 层级间的约束

当类型同时有 Applicative 和 Monad 实例时，`seq` 的行为必须与由 `bind` 派生的默认实现一致。这解释了为何 Validate 不应是 Monad。

## Alternative（失败恢复）

```lean
class Alternative (f : Type → Type) extends Applicative f where
  failure : f α
  orElse : f α → (Unit → f α) → f α  -- 写作 <|>
```

`guard` 函数结合 Alternative 进行搜索过滤：

```lean
def guard [Alternative f] (p : Prop) [Decidable p] : f Unit :=
  if p then pure () else failure
```

## 类型类继承机制

类型类本质是结构体，支持：
- 单继承与多重继承
- 默认方法实现
- 菱形问题自动坍缩

详见 [[type-classes]] 和 [[structures]]。

## 相关概念

- [[monads]] — Monad 详解
- [[type-classes]] — 类型类机制
- [[monad-transformers]] — 变换器组合效果
- [[universes]] — 宇宙多态
- [[monad-vs-applicative]] — 三者能力对比
