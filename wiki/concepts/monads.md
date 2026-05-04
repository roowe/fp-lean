---
title: 单子（Monads）
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [monad, functor, applicative]
sources: [book/FPLean/Monads/Arithmetic.lean, book/FPLean/Monads/Class.lean, book/FPLean/Monads/Do.lean, book/FPLean/Monads/IO.lean]
---

# 单子（Monads）

## 核心思想

单子是编码**计算上下文**（副作用）的抽象接口。纯函数式语言通过单子将可变状态、异常、非确定性等效果编码为纯函数。类型签名诚实声明程序的效果。

## Monad 类型类

```lean
class Monad (m : Type → Type) where
  pure : α → m α
  bind : m α → (α → m β) → m β
```

- `pure`：将纯值注入单子上下文
- `bind`（写作 `>>=`）：链式组合单子计算

### Monad 合约

1. **左单位元**：`pure x >>= f = f x`
2. **右单位元**：`v >>= pure = v`
3. **结合律**：`(v >>= f) >>= g = v >>= (fun x => f x >>= g)`

## 常见 Monad 实例

| Monad | 编码的效果 | 典型用途 |
|-------|-----------|---------|
| `Option` | 可能失败 | 查找、验证 |
| `Except ε` | 异常（带错误信息） | 错误处理 |
| `State σ` | 可变状态 | 状态机 |
| `Reader ρ` | 只读环境 | 配置传递 |
| `Many` | 非确定性搜索 | 枚举解 |
| `IO` | 真实世界副作用 | 文件、网络 |
| `WithLog` | 日志记录 | 调试、审计 |

## 多态的力量

同一套求值器代码，通过不同的 Monad 实例，可以处理不同效果：

```lean
def evaluateM [Monad m]
    (applySpecial : special → Int → Int → m Int) :
    Expr (Prim special) → m Int
  | Expr.const i => pure i
  | Expr.prim p e1 e2 =>
    evaluateM applySpecial e1 >>= fun v1 =>
    evaluateM applySpecial e2 >>= fun v2 =>
    applyPrim applySpecial p v1 v2
```

## do 记法

do 记法是 Monad 的语法糖，将 `>>=` 链转为类似命令式的顺序语句：

```lean
-- 等价于 bind 链
def firstThirdFifthSeventh [Monad m] (lookup : List α → Nat → m α)
    (xs : List α) : m (α × α × α × α) := do
  let first ← lookup xs 0
  let third ← lookup xs 2
  let fifth ← lookup xs 4
  let seventh ← lookup xs 6
  pure (first, third, fifth, seventh)
```

### 脱糖规则

| do 写法 | 脱糖结果 |
|---------|---------|
| `let x ← E` | `E >>= fun x => ...` |
| `let x := E` | `let x := E; ...` |
| `E` | `E >>= fun _ => ...` |

## mapM

monadic 版本的 map：

```lean
def mapM [Monad m] (f : α → m β) : List α → m (List β)
  | [] => pure []
  | x :: xs => do
    pure ((← f x) :: (← mapM f xs))
```

## 相关概念

- [[functor-applicative-monad]] — Functor/Applicative/Monad 层级
- [[io-and-side-effects]] — IO monad 详解
- [[monad-transformers]] — 组合多个 Monad
