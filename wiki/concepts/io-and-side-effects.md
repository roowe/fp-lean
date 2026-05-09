---
title: IO 与副作用
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [io, monad, error-handling]
sources: [book/FPLean/HelloWorld/RunningAProgram.lean, book/FPLean/HelloWorld/StepByStep.lean, book/FPLean/Monads/IO.lean]
---

# IO 与副作用

## 核心思想

Lean 是纯函数式语言——**程序不能有副作用，除非类型签名声明了这一点**。IO monad 将副作用（文件读写、网络等）编码为纯函数。

## 求值 vs 执行

| 概念 | 说明 |
|------|------|
| **求值（evaluation）** | 纯粹的数学表达式替换与化简，无副作用 |
| **执行（execution）** | 由运行时系统执行 IO 动作，产生实际副作用 |

## Hello World

```lean
def main : IO Unit := IO.println "Hello, world!"
```

`IO.println "Hello, world!"` 的类型是 `IO Unit`——一个**描述**了打印动作的值，不是打印动作本身。

## do 记法

do 块将多个 IO 动作组合为更大的程序：

```lean
def main : IO Unit := do
  let stdin ← IO.getStdin
  let stdout ← IO.getStdout
  stdout.putStrLn "How would you like to be addressed?"
  let input ← stdin.getLine
  let name := input.dropRightWhile Char.isWhitespace
  stdout.putStrLn s!"Hello, {name}!"
```

### do 块中三种绑定

| 语法 | 含义 |
|------|------|
| `let x ← action` | 执行 action，绑定结果 |
| `let x := expr` | 纯表达式绑定 |
| `action` | 执行 action，丢弃结果 |

### 嵌套动作

```lean
(← IO.getStdout).write buf
-- 等价于：
-- let tmp ← IO.getStdout
-- tmp.write buf
```

注意：**不能在 `if` 分支中使用嵌套动作**（会在条件判断前执行所有分支）。

## IO 动作是一等值

可以存储在数据结构中、作为参数传递，而不立即执行：

```lean
def countdown : Nat → List (IO Unit)
  | 0 => [IO.println "Blast off!"]
  | n + 1 => IO.println s!"{n + 1}" :: countdown n
```

## IO 的内部实现

```lean
def IO : Type → Type := EIO IO.Error
def EIO : Type → Type → Type := fun ε α => EST ε IO.RealWorld α
```

IO 基于 **EST**（Exceptional State Transformer），通过传递"世界状态"标记确保副作用安全排序。本质是同时编码了状态传递和异常处理。

## main 函数的三种签名

```lean
def main : IO Unit                                    -- 无参数，无退出码
def main : IO UInt32                                  -- 无参数，有退出码
def main (args : List String) : IO UInt32             -- 有参数，有退出码
```

## 常用 IO 操作

| 操作 | 说明 |
|------|------|
| `IO.println s` | 打印字符串（带换行） |
| `IO.getStdin` | 获取标准输入流 |
| `IO.FS.readFile ⟨path⟩` | 读取文件内容为 `String` |
| `IO.userError s` | 创建用户错误（配合 `throw` 使用） |

## #guard 编译期断言

```lean
#guard (some 5).get! == 5           -- 编译时检查，false 则编译失败
#guard parseArgs ["-i", "hi"] = ok ...
```

`#guard expr` 在编译时求值表达式，`true` 通过，`false` 编译报错。比 `#eval` 更严格——不只是看看输出，而是断言结果。

## 相关概念

- [[monads]] — IO 是 monad 的实例
- [[functor-applicative-monad]] — Monad 层级
- [[monad-transformers]] — 组合多种效果
