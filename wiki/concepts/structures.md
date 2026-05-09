---
title: 结构体
created: 2026-05-04
updated: 2026-05-09
type: concept
tags: [structures, lean-syntax, getting-started]
sources: [book/FPLean/GettingToKnow/Structures.lean]
---

# 结构体（Structures）

## 核心思想

结构体将多个简单数据组合为一个整体，类似 C 的 struct。定义结构体引入**全新类型**。类型类本质上也是结构体。

## 定义与创建

```lean
structure Point where
  x : Float
  y : Float

def origin : Point := { x := 0.0, y := 0.0 }
```

## 访问字段

**点表示法**自动转换为函数调用：`origin.x` → `Point.x origin`

```lean
#eval origin.x    -- 0.0
#eval origin.y    -- 0.0
```

每个字段自动生成**访问器函数**（如 `Point.x : Point → Float`）。

## 函数式更新

`with` 关键字创建新值，**不修改原值**：

```lean
def zeroX (p : Point) : Point :=
  { p with x := 0 }
```

## 构造器

默认为 `StructureName.mk`，可用 `::` 自定义：

```lean
structure Point where
  x : Float
  y : Float
deriving Repr

-- 等价的创建方式
#eval { x := 1.0, y := 2.0 : Point }
#eval Point.mk 1.0 2.0
```

## 位置参数

用 `⟨1, 2⟩` 按位置传参（不等同于 `<` `>`）。

## 结构体继承

结构体可以继承其他结构体，这是 [[type-classes]] 继承机制的基础：

```lean
structure Monster extends MythicalCreature where
  vulnerability : String
```

详见 [[functor-applicative-monad]] 中关于类型类继承的讨论。

## 项目实践

结构体在以下项目中用于组织核心数据：

- **02-json-parser** — `ParseState` 封装解析位置
- **03-grep** — `Config` 封装命令行配置
- **06-typed-db** — `Column` 描述数据库列
- **08-type-checker** — `Context`/`Env` 管理类型环境

示例：03-grep 的 `Config`（`Config.lean`）：

```lean
structure Config where
  pattern : String
  ignoreCase : Bool := false
  showLineNumbers : Bool := false
  showCount : Bool := false
  filePaths : List String := []
deriving Repr, BEq
```

示例：02-json-parser 的 `ParseState`（`Parser.lean`）：

```lean
structure ParseState where
  src : Array Char
  pos : Nat := 0
deriving Repr
```

## 相关概念

- [[datatypes-pattern-matching]] — 归纳类型（结构体的推广）
- [[polymorphism]] — 多态结构体
- [[type-classes]] — 类型类（本质是结构体 + 实例搜索）
