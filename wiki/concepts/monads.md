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

Monad 做两件事：**包装**（`pure`）和**链接**（`bind`/`>>=`）。

```lean
-- 包装：同一个值，放进不同类型
pure 5             -- Option 里是 some 5，Except 里是 ok 5，List 里是 [5]

-- 链接：从包装里取值，处理后放回；如果"出问题了"就短路
some 5   >>= fun x => some (x + 1)    -- some 6
none     >>= fun x => some (x + 1)    -- none（后面的函数根本不执行）
```

不同类型的链接规则不同（Option 失败就停，Except 出错就停，IO 按顺序排），但写法统一。do 记法是 `>>=` 的语法糖，让你不用写 lambda。

## Monad 类型类

```lean
class Monad (m : Type → Type) where
  pure : α → m α
  bind : m α → (α → m β) → m β
```

- `pure`：将纯值注入单子上下文
- `bind`（写作 `>>=`）：链式组合单子计算

### Monad 合约

> `>>=` 读作"然后"。`pure` 加包装，`>>=` 拆包装喂给下一步，下一步自己包好还回来。

Monad 的实现必须遵守三条规矩，保证 `pure` 和 `>>=` 的行为不奇怪。

**1. 左单位元：`pure x >>= f` 等于 `f x`**

包装了立刻拆开喂给 `f`，等于直接把 `x` 给 `f`。`pure` 没有偷偷改东西。

```lean
-- 假设 f n := some (n + 1)
pure 5 >>= f     -- some 5，然后取出 5 给 f → some 6
f 5              -- 直接给 → some 6
-- 结果一样
```

**2. 右单位元：`v >>= pure` 等于 `v`**

从 `v` 里取值再原样包装回去，等于什么都没做。

```lean
some 5 >>= pure   -- 取出 5，pure 5 → some 5
some 5            -- 什么都没做
-- 结果一样
```

**3. 结合律：`(v >>= f) >>= g` 等于 `v >>= (fun x => f x >>= g)`**

先算前两步再算第三步，和一口气算完三步，结果一样。就像 `(1+2)+3` 等于 `1+(2+3)`。

```lean
(some 5 >>= f) >>= g           -- 先算 some 5 然后 f，再把结果给 g
some 5 >>= (fun x => f x >>= g)  -- some 5，然后一口气把 f 和 g 都接上
-- 结果一样
```

通俗理解：两边都是 `f` 先跑，区别只是括号打在哪。就像吃三道菜——左边是"前两道吃完歇一下，再吃第三道"，右边是"三道菜一口气吃完"。吃的东西和顺序完全一样，只是中间有没有歇脚。

三条合起来的意思：**`pure` 是空操作，`>>=` 是顺序组合，不会偷加逻辑。** 写 `do` 块时可以放心拆分或合并步骤，行为不变。

## 常见 Monad 实例

每个 Monad 的链接规则（`>>=` 的行为）不同，决定了"出问题时怎么办"。

**Option — 可能有，也可能没有**

```lean
-- 有值：正常往下走
some "hello" >>= fun s => some (s.length)   -- some 5

-- 没值：短路，后面的代码根本不执行
none >>= fun s => some (s.length)           -- none
```

用途：字典查找 `lookup`、列表取第 N 个 `index`，找不到就返回 `none`，整条链自动短路。

**Except — 成功或报错**

```lean
-- 成功：正常往下走
ok 10 >>= fun x => ok (x * 2)              -- ok 20

-- 报错：短路，错误信息带着走
error "除零" >>= fun x => ok (x * 2)       -- error "除零"
```

用途：和 `Option` 类似，但 `none` 换成了带错误信息的 `error`，知道哪里出了问题。

**State — 自动传递状态**

```lean
-- 不用手动传状态变量，>>= 帮你传
def tick : State Nat Nat := do
  let n ← get           -- 取当前状态
  modify (· + 1)        -- 改状态
  pure n                -- 返回旧值
```

用途：计数器、随机数生成器、编译器里的符号表——任何需要"边算边改一个变量"的场景。

**Reader — 自动传环境**

```lean
-- 所有步骤共享同一个只读配置，不用当参数传来传去
def greet : Reader String String := do
  let name ← ask         -- 读取环境
  pure ("你好, " ++ name)
```

用途：数据库连接、应用配置、当前用户——任何"很多函数都需要同一个值"的场景。

**Many — 分叉，每条路都走**

```lean
-- 不是选一条路，而是所有可能都走一遍
def headsOrTails : Many Unit String :=
  Many.cons "正面" (Many.cons "反面" (Many.zero))

headsOrTails >>= fun result => doSomething result
-- result 会依次是 "正面" 和 "反面"，两条路都走
```

用途：搜索所有解、枚举组合、语法歧义消解——需要"试遍所有可能"的场景。

**IO — 按顺序执行，不短路**

```lean
def main : IO Unit := do
  IO.println "第一行"    -- 真的打印
  IO.println "第二行"    -- 真的打印
  -- 不会跳过任何一步，顺序严格保证
```

用途：读写文件、网络请求、打印输出——和真实世界打交道的场景。IO 的 `>>=` 不短路，每步都执行。

**WithLog — 正常算，顺便记日志**

```lean
def compute : WithLog String Nat := do
  log "开始计算"
  let x := 3 + 5
  log "计算完毕"
  pure x
-- 结果：ok 8，同时带着日志 ["开始计算", "计算完毕"]
```

用途：调试、审计——不影响计算逻辑，但每步都留个痕迹。

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

do 记法只有三条转换规则，编译器把 do 块展开成 `>>=` 调用：

**`let x ← E` — 从 Monad 里取值**

```lean
-- do 写法                    脱糖结果
let x ← some 5             some 5 >>= fun x =>
some (x + 1)                  some (x + 1)
```

`←` 拆包装，`x` 拿到里面的值。最常用的写法。

**`let x := E` — 普通赋值，不涉及 Monad**

```lean
-- do 写法                    脱糖结果
let x := 5                 let x := 5
some x                        some x
```

没有 `←`，就是普通的 `let` 绑定，不拆包装。

**单独的 `E` — 执行但丢弃结果**

```lean
-- do 写法                    脱糖结果
IO.println "hi"            IO.println "hi" >>= fun _ =>
IO.println "bye"              IO.println "bye"
```

不需要结果时用。`fun _ =>` 表示忽略返回值，继续下一步。

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
