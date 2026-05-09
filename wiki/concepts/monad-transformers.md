---
title: 单子变换器（Monad Transformers）
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [monad-transformer, monad]
sources: [book/FPLean/MonadTransformers/ReaderIO.lean, book/FPLean/MonadTransformers/Transformers.lean, book/FPLean/MonadTransformers/Order.lean, book/FPLean/MonadTransformers/Do.lean]
---

# 单子变换器（Monad Transformers）

## 核心思想

单子变换器是"Monad 工具箱"——向任意 Monad 添加特定效果（失败、异常、状态、环境）。通过堆叠变换器组合多种效果，而不需要为每种组合手写 Monad。

## 核心变换器

| 变换器 | 添加的效果 | 定义 |
|--------|-----------|------|
| `OptionT m` | 失败 | `m (Option α)` |
| `ExceptT ε m` | 异常 | `m (Except ε α)` |
| `StateT σ m` | 可变状态 | `σ → m (α × σ)` |
| `ReaderT ρ m` | 只读环境 | `ρ → m α` |

使用 `abbrev` 定义组合类型：

```lean
abbrev ConfigIO (α : Type) : Type := ReaderT Config IO α
```

## .run 解包变换器

变换器堆叠后，用 `.run` 逐层剥开得到最终结果：

```lean
-- SearchM := ReaderT Config (ExceptT String Id)
let result := (searchLines lines).run config |> ExceptT.run
-- .run config : ExceptT String Id (Array (Nat × String))
-- |> ExceptT.run : Except String (Array (Nat × String))
```

从外到内：`ReaderT.run` 传入配置，`ExceptT.run` 取出 `Except` 值。

## MonadLift — 单子提升

`monadLift` 将内层 Monad 的动作提升到变换后的 Monad：

```lean
-- IO 动作可以直接在 ReaderT Config IO 中使用
-- 通过 MonadLift 自动提升
```

## 堆叠顺序很重要

单子变换器**不可交换**——不同顺序产生不同行为：

```lean
-- 顺序1：异常时状态回滚
abbrev M1 := StateT LetterCounts (ExceptT Err Id)

-- 顺序2：异常时状态保留
abbrev M2 := ExceptT Err (StateT LetterCounts Id)
```

展开 M1：`LetterCounts → Except Err (α × LetterCounts)` — 异常丢弃状态
展开 M2：`LetterCounts → (Except Err α × LetterCounts)` — 异常保留状态

## do 中的命令式特性

单子变换器为 do 记法引入了更多命令式特性：

### Early Return

```lean
def List.find? (p : α → Bool) (xs : List α) : Option α := do
  for x in xs do
    if p x then return x
  failure
```

背后使用 `ExceptT` 实现。

### 局部可变变量

```lean
def two : Nat := Id.run do
  let mut x := 0
  x := x + 1
  x := x + 1
  return x    -- 2
```

背后使用 `StateT` 实现。

### 循环

```lean
-- for 循环
for x in xs do
  ...

-- while 循环
while not buf.isEmpty do
  stdout.write buf
  buf ← stream.read bufsize

-- repeat 循环（不需要 partial）
repeat IO.println "Spam!"
```

## 管道操作符

```lean
5 |> times3 |> toString |> ("It is " ++ ·)
-- 等价于 ("It is " ++ ·) (toString (times3 5))
```

| 操作符 | 方向 |
|--------|------|
| `|>` | 正向管道 |
| `<|` | 反向管道 |
| `|>.method` | 管道点记法 |

## 相关概念

- [[monads]] — Monad 基础
- [[functor-applicative-monad]] — 层级关系
- [[io-and-side-effects]] — IO monad
