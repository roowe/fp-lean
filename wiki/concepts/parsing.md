---
title: 解析器设计模式
created: 2026-05-09
updated: 2026-05-09
type: concept
tags: [parsing, monad, design-pattern]
---

# 解析器设计模式

## 核心思想

解析器把字符串转换为结构化数据。Lean 中没有标准库的解析器，需要自己实现。常见的三种设计：递归下降、Pratt 优先级爬升、Parser Monad + 组合子。

## 消费式解析

所有解析器共享一个核心模式——接受输入，返回结果和剩余输入：

```lean
-- 类型模式：输入 → (结果, 剩余输入)
String → List Char → (Expr, List Char)
```

先 `String.toList` 转字符列表，逐个消费，返回解析结果和剩余部分。

## 递归下降解析器

按语法优先级分层，每层一个函数：

```lean
-- 三层优先级：expr > term > factor
mutual
  def parseExpr : ... := parseTerm >>= 继续 +/-
  def parseTerm : ... := parseFactor >>= 继续 */
  def parseFactor : ... := 数字 或 括号
end
```

- 简单直观，初学者容易理解
- 加新优先级需要加新函数层
- 需要 `mutual` 互递归块
- 书外知识，书里没教过解析器

## Pratt 解析器（优先级爬升）

单个函数，用 `minPrec` 参数控制优先级：

```lean
def parseExpr (minPrec : Nat) (chars : List Char) : ... :=
  先解析原子 → 然后看下一个运算符优先级是否 ≥ minPrec
  如果是就继续解析，否则返回
```

- 比递归下降更通用
- 加新运算符只需在优先级表里加一行
- 不需要 `mutual` 块
- 书外知识

## Parser Monad + 组合子

把解析器建模为 Monad，构建组合子库：

```lean
abbrev Parser α := List Char → Nat → Validate String (Nat × α)

-- 组合子
def peek    : Parser (Option Char)      -- 看当前字符
def satisfy : (Char → Bool) → Parser Char  -- 满足条件才消费
def char    : Char → Parser Char        -- 匹配特定字符
def lit     : String → Parser String    -- 匹配字符串
def skipWS  : Parser Unit               -- 跳过空白
def fail    : String → Parser α         -- 报错
```

小组合子拼接出大解析器：

```lean
def parseNull := lit "null" *> pure JValue.null
def parseTrue := lit "true" *> pure (JValue.bool true)
```

- 架构最优雅，复用性最强
- 经典 FP 模式（类似 Haskell Parsec）
- 需要理解 Monad 才能写出
- 书外知识

### `*>` (SeqRight) 组合子

```lean
lit "null" *> pure JValue.null
```

执行左侧但丢弃其值，保留右侧。用于"必须匹配但不关心结果"的场景。

## 错误恢复

工业级解析器不会遇到第一个错误就停止，而是跳到安全位置继续：

```lean
-- 失败后跳到下一个 , 或 ] 继续解析
let recPos := skipUntil cs p [',', ']']
match get cs recPos with
| some ',' => 继续解析下一个元素
| some ']' => 结束数组
| _ => 报错
```

书外知识，书里的 `Validate` 只累积错误但不做恢复。

## 转义序列解析

逐字符处理 `\n`、`\t`、`\"`、`\\`、`\uXXXX` 等：

```lean
if c == '\\' then
  match 下一个字符 with
  | 'n' => 换行, 't' => 制表符
  | 'u' => 读4位十六进制 → Char.ofNat
  | ...
```

书里完全没涉及字符串的逐字符解析。

## Pretty Printer（序列化）

把数据结构转回字符串，是解析的逆过程：

```lean
def JValue.toString : JValue → String
  | .null => "null"
  | .array arr => "[" ++ ", ".intercalate (arr.map toString) ++ "]"
  | .object kvs => "{" ++ ... ++ "}"
```

书里没涉及序列化。

## 三种方案对比

| 维度 | 递归下降 | Pratt | Parser Monad |
|------|---------|-------|-------------|
| 复杂度 | 低 | 中 | 高 |
| 扩展性 | 差（加优先级要加函数） | 好（加一行） | 好（加组合子） |
| 错误处理 | 遇错即停 | 遇错即停 | 可组合错误恢复 |
| 前置知识 | 少 | 少 | 需要 Monad |
| 适用场景 | 学习、简单语法 | 运算符多的语言 | 复杂语法、工业级 |

## 项目实践

### 01-calc — 递归下降解析器

用 `mutual` 互递归实现三层优先级（expr > term > factor），`partial def` 允许非结构递归：

```lean
mutual
partial def parseExpr : Parser Expr := fun input => do
  let (lhs, rest) ← parseTerm input
  parseExprAux lhs rest

partial def parseTerm : Parser Expr := fun input => do
  let (lhs, rest) ← parseFactor input
  parseTermAux lhs rest

partial def parseFactor : Parser Expr := fun input =>
  match input with
  | '(' :: rest => do
    let (e, rest') ← parseExpr rest
    match rest' with | ')' :: rest'' => pure (e, rest'') | ...
  | '-' :: rest => do let (e, rest') ← parseFactor rest; pure (Expr.neg e, rest')
  | ...
end
```

### 02-json-parser — Fuel 模式的互递归解析器

用 `Nat` 作为 fuel 防止无限递归，配合 `Validate` 累积解析错误：

```lean
def parseValueFuel (fuel : Nat) (st : ParseState) : Validate String (Nat × JValue) :=
  match fuel with
  | 0 => .errors ["Exceeded fuel limit"]
  | fuel + 1 =>
    let i := skipWS st st.pos
    match peek? st i with
    | some '{' => parseObjectFuel fuel {st with pos := i}
    | some '[' => parseArrayFuel fuel {st with pos := i}
    | some '"' => match parseStringLit st i with
      | .ok (j, s) => .ok (j, JValue.string s) | .errors e => .errors e
    | ...
```

## 相关概念

- [[monads]] — Parser Monad 的基础
- [[functor-applicative-monad]] — Validate（Applicative 累积错误）
- [[list-type]] — `String.toList` 字符列表处理
- [[string-type]] — 字符串操作
- [[datatypes-pattern-matching]] — 归纳类型定义 AST
