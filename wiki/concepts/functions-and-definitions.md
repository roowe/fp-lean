---
title: 函数与定义
created: 2026-05-04
updated: 2026-05-09
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

## 项目实践

除 06-typed-db 外，所有项目都大量使用函数定义的各种语法。

| 项目 | 关键用法 |
|------|---------|
| 01-calc | `def` + 模式匹配定义 `eval`/`toString`；`let rec` 递归解析器 |
| 02-json-parser | `mutual` ... `end` 互递归函数；`where` 辅助定义 |
| 03-grep | `let rec loop` 局部递归；`where` 子句在 `parseArgs` 中 |
| 04-eval-prove | `def` + 模式匹配定义 `simplify`/`eval`/`evalSafe` |
| 05-safe-sort | `have : ... := by grind` 局部证明绑定 |
| 07-formal-verify | `by` 策略块定义证明函数 |
| 08-type-checker | `def` + 模式匹配定义 `typeCheck`/`eval`；`bif` 条件求值 |

**04-eval-prove — 模式匹配 + do 记法定义 evalSafe（EvalProve/Eval.lean）**

```lean
def Expr.evalSafe (env : String → Option Int) : Expr → Except String Int
  | .num n => .ok n
  | .var x =>
    match env x with
    | some v => .ok v
    | none => .error s!"未绑定变量: {x}"
  | .add a b => do
    let va ← evalSafe env a
    let vb ← evalSafe env b
    return va + vb
```

**03-grep — where 子句 + let rec（MiniGrep/Config.lean）**

```lean
def parseArgs (args : List String) : Except String Config :=
  let rec loop (optMode : Bool) (pat? : Option String)
      (cfg : Config) (xs : List String) : Except String Config :=
    match xs with
    | [] => match pat? with
      | none => throw "No pattern provided"
      | some pat => return { cfg with pattern := pat }
    | tok :: rest => ...
  loop true none { pattern := "" } args
```

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
