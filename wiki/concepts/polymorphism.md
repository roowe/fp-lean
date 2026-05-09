---
title: 多态
created: 2026-05-04
updated: 2026-05-09
type: concept
tags: [polymorphism, types, getting-started]
sources: [book/FPLean/GettingToKnow/Polymorphism.lean]
---

# 多态（Polymorphism）

## 核心思想

类型和函数可以接受**类型作为参数**，使同一代码适用于多种类型。Lean 的多态特指**参数化多态**（parametric polymorphism）。

## 多态结构体

```lean
structure PPoint (α : Type) where
  x : α
  y : α

def origin : PPoint Float := { x := 0.0, y := 0.0 }
```

## 隐式参数

用花括号 `{α : Type}` 声明，Lean 自动推断：

```lean
def replaceX {α : Type} (point : PPoint α) (newX : α) : PPoint α :=
  { point with x := newX }
```

Lean 也支持**自动隐式参数**——省略声明，根据使用自动推断。

## 核心多态类型

| 类型 | 含义 | 语法糖 |
|------|------|--------|
| `List α` | 元素列表 | `[1, 2, 3]` |
| `Option α` | 可能缺失的值 | — |
| `Prod α β` | 对（pair） | `α × β` |
| `Sum α β` | 二选一 | `α ⊕ β` |
| `Unit` | 只有一个值 `()` | — |
| `Empty` | 没有值（不可达代码） | — |

## 依赖类型初窥

返回类型可以依赖参数值（详见 [[dependent-types]]）：

```lean
def posOrNegThree (s : Sign) :
    match s with | Sign.pos => Nat | Sign.neg => Int :=
  match s with
  | Sign.pos => (3 : Nat)
  | Sign.neg => (-3 : Int)
```

## 类型推断

Lean 根据上下文自动确定类型参数，但函数参数通常需要注解。当 Lean 无法确定类型时，会生成**元变量**（`?m.XYZ`）。

## 数值类型转换

Lean 的数值类型不自动转换，需要显式操作：

| 转换 | 方法 | 说明 |
|------|------|------|
| Nat → Int | `Int.ofNat n` 或 `(n : Int)` | 自然数转整数（向上转型） |
| Nat → Float | `Float.ofNat n` | 自然数转浮点 |
| String → Nat | `"123".toNat?` | 返回 `Option Nat`，失败为 `none` |
| Nat → Char | `Char.ofNat cp` | 码点转字符（Unicode） |
| Float 构造 | `Float.ofScientific mant neg exp` | 科学计数法构造浮点数 |

```lean
-- Nat 和 Int 混合运算需要显式转换
let x : Nat := 5
let y : Int := -(Int.ofNat x)    -- Nat → Int，然后取负

-- 科学计数法构造浮点
Float.ofScientific 314 false (-2)  -- 3.14
```

## 项目实践

多态在以下项目中广泛使用：

- **01-calc** — 解析器中的多态函数
- **02-json-parser** — 多态类型如 `Validate ε α`
- **03-grep** — 多态 monad 变换器 `ReaderT`/`ExceptT`
- **06-typed-db** — `DBType.asType` 多态映射
- **08-type-checker** — 多态类型定义

示例：02-json-parser 的 `Validate`（`Validate.lean`）— 类型参数 `ε` 和 `α` 使同一类型适配不同错误类型和结果类型：

```lean
inductive Validate (ε : Type) (α : Type) where
  | ok : α → Validate ε α
  | errors : List ε → Validate ε α
```

## 相关概念

- [[evaluation-and-types]] — 类型基础
- [[type-classes]] — 另一种多态机制（ad-hoc 多态）
- [[dependent-types]] — 类型依赖值
- [[universes]] — 类型层级的宇宙系统
