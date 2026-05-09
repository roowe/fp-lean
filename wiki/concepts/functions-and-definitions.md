---
title: 函数与定义
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [functions, lean-syntax, getting-started]
sources: [book/FPLean/GettingToKnow/FunctionsDefinitions.lean]
---

# 函数与定义

## 核心思想

函数是 Lean 的一等公民。每个 Lean 函数**恰好接受一个参数**，多参数通过柯里化实现。

## 定义函数

`def` 关键字定义值和函数，`:=` 赋值：

```lean
def add1 (n : Nat) : Nat := n + 1

def maximum (n : Nat) (k : Nat) : Nat :=
  if n < k then k else n
```

## 柯里化（Currying）

多参数函数实际上是逐参数返回新函数：

```lean
#check add1       -- add1 : Nat → Nat
#check maximum    -- maximum : Nat → Nat → Nat
#check maximum 3  -- maximum 3 : Nat → Nat  ← 部分应用！
```

箭头**右结合**：`Nat → Nat → Nat` 即 `Nat → (Nat → Nat)`

## 匿名函数

```lean
fun x => x + 1          -- 完整写法
(· + 1)                  -- 简洁写法（用 · 占位）
```

## 局部定义

```lean
def unzip : List (α × β) → List α × List β
  | [] => ([], [])
  | (x, y) :: xys =>
    let (xs, ys) : List α × List β := unzip xys
    (x :: xs, y :: ys)
```

`let` 不可递归，递归需 `let rec`。

## 便捷语法

| 语法 | 说明 |
|------|------|
| `def name (x : T) : R := body` | 函数定义 |
| `fun x => body` | 匿名函数 |
| `(· + 1)` | 匿名函数简写 |
| `let x := e` | 局部绑定 |
| `let rec f ...` | 递归局部绑定 |
| `where` | 函数后附带的局部定义 |
| `private` | 限制为模块内部可见 |
| `open ... in` | 单个定义内打开命名空间 |

### where 子句

```lean
def parseExpr (minPrec : Nat) (chars : List Char) : ... :=
  ... 使用 parseAtom 和 parseExprRest ...
where
  parseAtom ... := ...
  parseExprRest ... := ...
```

`where` 定义的函数只在该函数内可见，类似 `let rec` 但写在函数后面，不增加缩进层级。

### private 可见性

```lean
private def isDigit (c : Char) : Bool := c.isDigit
```

`private` 限制定义为模块内部可见，外部无法调用。

### open ... in 局部打开

```lean
open ParseState in
def parseNull (s : ParseState) : ... := ...
-- 只在这个定义内打开了 ParseState 命名空间
```

## 相关概念

- [[evaluation-and-types]] — 类型和求值
- [[datatypes-pattern-matching]] — 模式匹配定义函数
- [[polymorphism]] — 多态函数
- [[type-classes]] — 通过类型类重载运算符
