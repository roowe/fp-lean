---
title: 类型类（Type Classes）
created: 2026-05-04
updated: 2026-05-09
type: concept
tags: [type-class, polymorphism]
sources: [book/FPLean/TypeClasses/Polymorphism.lean, book/FPLean/TypeClasses/Pos.lean, book/FPLean/TypeClasses/OutParams.lean, book/FPLean/TypeClasses/Indexing.lean, book/FPLean/TypeClasses/StandardClasses.lean, book/FPLean/TypeClasses/Coercions.lean]
---

# 类型类（Type Classes）

## 核心思想

类型类是 Lean 的**重载机制**——通过实例隐式参数 `[C α]`，编译器自动搜索合适的方法实现。类型类本质是结构体 + 自动实例搜索。

## 与 Interface 的异同

**同**：都是"定义抽象行为 → 具体类型实现 → 使用时自动找具体实现"。流程一样。

**异**：谁能给类型添加实现。
- Interface：只有定义该类型的人能加。写在类定义里。
- Type class：任何人都能加。写在类型定义外面。

实际后果：你拿到一个第三方库的类型，想让它支持某个行为——interface 做不到，type class 可以。

## 定义与使用

```lean
class Plus (α : Type) where
  plus : α → α → α

instance : Plus Nat where
  plus := Nat.add

#eval Plus.plus 3 5    -- 8
```

方括号 `[Plus α]` 是**实例隐式参数**，Lean 通过递归实例搜索自动填充。

## 核心标准类型类

| 类型类 | 运算符 | 说明 |
|--------|--------|------|
| `HAdd α β γ` | `x + y` | 异构加法 |
| `Add α` (= `HAdd α α α`) | `x + y` | 同构加法 |
| `HMul α β γ` | `x * y` | 异构乘法 |
| `OfNat α n` | `37` | 数字字面量重载 |
| `ToString α` | `s!"{x}"` | 转字符串 |
| `BEq α` | `x == y` | 布尔相等（返回 `Bool`） |
| `Ord α` | `compare` | 三路比较（`lt`/`eq`/`gt`） |
| `Hashable α` | `hash` | 哈希（返回 `UInt64`） |
| `Append α` | `xs ++ ys` | 追加 |
| `Functor f` | `f <$> x`, `fmap` | [[functor-applicative-monad|函子]] |
| `GetElem coll idx item inBounds` | `xs[i]` | 安全索引 |

## 输出参数

`outParam` 标记的参数不需要提前确定，通过搜索结果填充：

```lean
class HPlus (α : Type) (β : Type) (γ : outParam Type) where
  hPlus : α → β → γ
```

## 默认实例

`@[default_instance]` 在类型信息不完整时提供回退选择。

## GetElem 类型类

四参数类型类，实现安全索引：

```lean
class GetElem (coll : Type) (idx : Type) (item : outParam Type)
    (inBounds : outParam (coll → idx → Prop)) where
  getElem : (c : coll) → (i : idx) → inBounds c i → item
```

索引合法性由 `inBounds` 命题函数定义，详见 [[propositions-and-proofs]]。

## BEq vs DecidableEq

两个都是"相等"，但用途不同：

| 类型类 | 返回类型 | 用途 |
|--------|---------|------|
| `BEq` | `Bool` | 运行时比较，`x == y` |
| `DecidableEq` | `Decidable (x = y)` | 编译期判断 `x = y` 命题是否可证 |

`BEq` 是程序用的（拿到 `true`/`false`），`DecidableEq` 是证明用的（让 `simp`、`omega` 等策略知道相等性可判定）。

## bif — Bool 模式匹配

`bif` 是对 `Bool` 的内置模式匹配，保证返回类型统一：

```lean
bif cond then val1 else val2
-- cond : Bool，返回类型统一
```

与 `if` 不同，`bif` 不依赖 `Decidable` 实例，直接按 `Bool` 值分叉。

## 强制转换（Coercions）

四种强制转换类型类，在类型不匹配时自动插入：

| 类型类 | 方向 | 例子 |
|--------|------|------|
| `Coe α β` | `α → β` | `Pos → Nat` |
| `CoeDep α val β` | 特定值 `→ β` | 非空列表 `→ NonEmptyList` |
| `CoeSort α Sort` | `α → Type/Prop` | `Bool → Prop`（`b = true`） |
| `CoeFun α ftype` | `α → 函数` | `Adder → (Nat → Nat)` |

Lean 自动组合多个 `Coe` 形成**强制转换链**。`↑` 语法手动插入。

## 自动派生

```lean
deriving instance BEq, Hashable, Repr for Pos
```

| 可派生的类 | 说明 |
|-----------|------|
| `BEq` | 布尔相等 `x == y` |
| `Hashable` | 哈希值 |
| `Repr` | 调试输出 `repr x` |
| `Inhabited` | 提供默认值（如 `Nat` 默认 `0`） |
| `DecidableEq` | 可判定相等（编译期 `x = y` 命题） |
| `Nonempty` | 证明类型非空（`mutual` + `partial` 块需要） |
| `Ord` | 三路比较 |

## 项目实践

| 项目 | 使用的类型类 |
|------|-------------|
| 01-calc | `deriving Repr, BEq, Nonempty`；手动实现 `ToString` |
| 02-json-parser | `deriving Repr, BEq`；手动实现 `Functor`/`Applicative` for `Validate` |
| 05-safe-sort | `Ord` 类型类（`insertSorted [Ord α]`） |
| 07-formal-verify | `Ord`、`DecidableEq`、`BEq`（`insert`/`insertionSort` 参数约束） |
| 08-type-checker | `deriving Repr, DecidableEq, BEq` for `Ty`；手动实现 `ToString` |

**02-json-parser — 手动实现 Functor/Applicative（JsonParser/Validate.lean）**

```lean
instance : Functor (Validate ε) where
  map f
    | .ok a => .ok (f a)
    | .errors errs => .errors errs

instance : Applicative (Validate ε) where
  pure := .ok
  seq f x :=
    match f with
    | .ok g => g <$> (x ())
    | .errors errs =>
      match x () with
      | .ok _ => .errors errs
      | .errors errs' => .errors (errs ++ errs')
```

## 相关概念

- [[polymorphism]] — 参数化多态
- [[functor-applicative-monad]] — Functor 类型类
- [[structures]] — 类型类本质是结构体
- [[propositions-and-proofs]] — `Decidable` 类型类
