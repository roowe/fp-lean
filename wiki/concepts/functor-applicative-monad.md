---
title: Functor / Applicative / Monad 层级
created: 2026-05-04
updated: 2026-05-09
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

对包装里的值施加函数，包装不变。

```lean
class Functor (f : Type u → Type v) where
  map : (α → β) → f α → f β
```

`f` 是类型构造器——吃进一个类型，吐出一个类型（如 `Option`、`List`）。`map` 把函数施加到 `f` 包装里的值上，包装本身不变：

```lean
-- f = Option 时，map 的类型是 (α → β) → Option α → Option β
map (fun n => n + 1) (some 5)    -- some 6
map (fun n => n + 1) none        -- none（没有值，什么都不做）

-- f = List 时，map 的类型是 (α → β) → List α → List β
map (fun n => n + 1) [1, 2, 3]   -- [2, 3, 4]
```

和 Monad 的区别：Functor 只能"改里面的值"，不能"根据里面的值决定下一步做什么"——那是 `>>=` 的事。

运算符：`g <$> x` 等价于 `Functor.map g x`

## Applicative（应用函子）

Functor 的升级版。两个新能力：`pure`（包装）和 `seq`（包装里的函数施加到包装里的值）。

```lean
class Applicative (f : Type → Type) extends Functor f where
  pure : α → f α
  seq : f (α → β) → (Unit → f α) → f β  -- 写作 <*>
```

`seq` 的第二个参数是 `Unit → f α` 而不是 `f α`，这是为了延迟求值——不立刻算第二个参数，等第一个成功再说。`Unit →` 可以当成惰性包装，忽略它，直接看返回的 `f α` 就行。

`pure` 就是加包装。`seq` 比较绕，换成具体类型看：

```lean
-- f = Option 时，seq 的类型是 Option (α → β) → Option α → Option β
-- 函数被包装在 Option 里，值也被包装在 Option 里
some (fun n => n + 1) <*> some 5      -- some 6（两边都有值，正常算）
some (fun n => n + 1) <*> none        -- none（值那边没有，短路）
none <*> some 5                       -- none（函数那边没有，短路）
```

和 Functor、Monad 的区别，用一个例子说清：

```lean
-- Functor：函数是普通的，只能改一个包装里的值
map (fun x => x + 1) (some 5)             -- some 6

-- Applicative：函数也可以被包装，能组合多个包装值
-- 但前一步的结果不能影响"接下来做什么"
let add x y := x + y
some add <*> some 3 <*> some 5            -- some 8

-- Monad：前一步的结果能影响下一步（Applicative 做不到）
some 5 >>= fun x =>
  if x > 3 then some "大" else none       -- some "大"（根据 x 的值做决定）
```

一句话：**Functor 改一个值，Applicative 组合多个值，Monad 根据值做决定。**

### Applicative 四法则

1. **恒等**：`pure id <*> v = v`
2. **复合**：`pure (· ∘ ·) <*> u <*> v <*> w = u <*> (v <*> w)`
3. **同态**：`pure f <*> pure x = pure (f x)`
4. **交换**：`u <*> pure x = pure (fun f => f x) <*> u`

### Validate — 累积错误的 Applicative（书里的示例，非标准库）

```lean
-- 这不是 Lean 标准库自带的类型，是书里设计的示例
inductive Validate (ε α : Type) : Type where
  | ok : α → Validate ε α
  | errors : NonEmptyList ε → Validate ε α
```

Validate 是 Applicative 但**不是** Monad——因为它累积所有错误而非遇到第一个就停止。

## Monad（单子）

Monad 允许前一步的值影响后续计算。只需实现 `bind` + `pure`，自动获得 `map`、`seq` 等。详见 [[monads]]。

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

## 项目实践

### 02-json-parser — Validate 作为 Applicative（非 Monad）

`Validate` 的 `seq` 实现会累积所有错误而非遇到第一个就停止，这是它**不能**成为 Monad 的原因：

```lean
instance : Applicative (Validate ε) where
  pure := .ok
  seq f x := match f with
    | .ok g => g <$> (x ())
    | .errors errs => match x () with
      | .ok _ => .errors errs
      | .errors errs' => .errors (errs ++ errs')  -- 累积错误！
```

如果 `Validate` 也是 Monad（`bind` 遇错即停），就会与 `seq` 的累积行为矛盾，违反 Applicative-Monad 一致性约束。

## 相关概念

- [[monads]] — Monad 详解
- [[type-classes]] — 类型类机制
- [[monad-transformers]] — 变换器组合效果
- [[universes]] — 宇宙多态
- [[monad-vs-applicative]] — 三者能力对比
