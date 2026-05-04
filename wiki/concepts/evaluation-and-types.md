---
title: 求值与类型
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [lean-syntax, types, getting-started]
sources: [book/FPLean/GettingToKnow/Evaluating.lean, book/FPLean/GettingToKnow/Types.lean]
---

# 求值与类型

## 核心思想

Lean 是**严格的纯函数式语言**。求值（evaluation）类似数学算术——变量一旦绑定不可更改，没有副作用。

## 表达式求值

`#eval` 命令计算表达式并显示结果：

```lean
#eval 1 + 2                              -- 3
#eval String.append "Hello, " "Lean!"    -- "Hello, Lean!"
```

`if/then/else` 是**表达式**（非语句），可以嵌套使用：

```lean
String.append "it is " (if 1 > 2 then "yes" else "no")
-- 求值为 "it is no"
```

### 关键区别

| 概念 | 命令式语言 | Lean |
|------|-----------|------|
| 变量 | 可修改 | 不可变 |
| 函数调用 | `f(x, y)` | `f x y`（空格分隔） |
| 副作用 | 普遍存在 | 类型系统管控（[[io-and-side-effects]]） |
| 部分应用 | 通常不支持 | `f x` 返回等待剩余参数的函数 |

## 类型系统

类型对程序可计算的值进行分类。Lean 的类型系统远超常规语言——可以编码强规范甚至数学定理。

```lean
#eval (1 + 2 : Nat)    -- Nat 自然数，任意精度
#eval (1 - 2 : Nat)    -- 0（Nat 不表示负数）
#eval (1 - 2 : Int)    -- -1
```

`#check` 只检查类型不求值：

```lean
#check (1 - 2 : Int)   -- 报告 "1 - 2 : Int"
```

### 类型层级

```
Nat : Type       -- Nat 的类型是 Type
Int : Type       -- Int 的类型也是 Type
Type : Type 1    -- Type 本身也有类型（避免 [[universes]] 悖论）
```

## 相关概念

- [[functions-and-definitions]] — 函数定义与柯里化
- [[datatypes-pattern-matching]] — 归纳数据类型
- [[polymorphism]] — 类型参数化
- [[structures]] — 结构体
