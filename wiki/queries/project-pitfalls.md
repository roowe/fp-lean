---
title: 项目难点与钻研指南
created: 2026-05-09
updated: 2026-05-09
type: query
tags: [getting-started, summary]
---

# 项目难点与钻研指南

> 基于 8 个实战项目的源码分析，按难度排序列出新手需要重点钻研的卡点。
> 依据项目：01-calc, 02-json-parser, 03-grep, 04-eval-prove, 05-safe-sort, 06-typed-db, 07-formal-verify, 08-type-checker

## 难度排序

| 顺序 | 项目 | 卡点 | 建议钻研时间 |
|------|------|------|------------|
| 1 | 01-calc | parseExprAux 左结合构建 | 1-2 天 |
| 2 | 02-json-parser | fuel 模式为什么替代 partial | 半天 |
| 3 | 02-json-parser | Validate 的 seq 错误累积 | 1 天 |
| 4 | 03-grep | ReaderT 展开与 .run 解包顺序 | 1 天 |
| 5 | 05-safe-sort | 函数体内的 have 证明 | 1 天 |
| 6 | 05-safe-sort | fun_induction vs induction | 半天 |
| 7 | 04-eval-prove | grind 不够用时手动 split | 1 天 |
| 8 | 07-formal-verify | match 替代 induction + 递归定理 | 1-2 天 |
| 9 | 06-typed-db | Universe Pattern 的必要性 | 1-2 天 |
| 10 | 08-type-checker | 依赖模式匹配的五路同步 | 2-3 天 |

---

## 01-calc — parseExprAux 左结合构建

```lean
partial def parseExprAux (lhs : Expr) : Parser Expr :=
  fun input =>
    match input with
    | '+' :: rest => do
        let (rhs, rest') ← parseTerm rest
        parseExprAux (Expr.add lhs rhs) rest'  -- lhs 累积传递
    | _ => pure (lhs, input)
```

新手直觉是"解析完一个再解析下一个"，但 `lhs` 是**累积的**——`1 + 2 + 3` 被解析为 `Expr.add (Expr.add 1 2) 3`（左结合）。理解"为什么 lhs 要当参数传递"需要画几棵语法树。

Parser 的类型签名 `List Char → Except ParseError (α × List Char)` 也是初见杀——"消费一部分输入，返回剩余输入"这个模式在书里没有正式教过。

## 02-json-parser — fuel 模式 vs partial

```lean
def parseValueFuel (fuel : Nat) (st : ParseState) : Validate String (Nat × JValue) :=
  match fuel with
  | 0 => .errors ["Exceeded fuel limit"]
  | fuel + 1 => ...
```

书里的 01-calc 解析器直接用 `partial`，但 JSON 解析器改用 fuel。原因：`partial` 函数不能被证明任何性质（`fun_induction` 不适用），fuel 模式让函数结构递归，Lean 接受它终止。

**什么时候用 fuel 什么时候用 partial**：需要证明性质时用 fuel，纯 IO 工具用 partial 即可。

## 02-json-parser — Validate 的 seq 错误累积

```lean
seq f x := match f with
  | .ok g => g <$> (x ())
  | .errors errs => match x () with
    | .ok _ => .errors errs
    | .errors errs' => .errors (errs ++ errs')  -- 合并两边的错误
```

`bind` 要求**先看到前步结果才能决定后步**，但错误累积要求**两步同时跑完再合并**——这两者矛盾。理解这一点需要亲手写一个 Validate 的 `bind` 试试，会发现写不出来。详见 [[monad-vs-applicative]]。

## 03-grep — ReaderT 展开与 .run 解包

```lean
-- ReaderT Config (ExceptT String Id) α
-- 展开 ReaderT：Config → ExceptT String Id α
-- 展开 ExceptT：Config → Except String α
-- 所以 searchLines 本质上是一个「接收 Config，返回 Except 结果」的函数
```

解包顺序是反直觉的：

```lean
(searchLines lines).run config |> ExceptT.run
-- .run config 传给 ReaderT（最外层）
-- |> ExceptT.run 剥掉 ExceptT（内层）
```

新手容易搞反——以为 `.run` 是从内到外，实际是从外到内。建议亲手把类型展开写在纸上。详见 [[monad-transformers]]。

## 05-safe-sort — 函数体内的 have 证明

```lean
| ⟨i' + 1, _⟩ =>
    have : i' < arr.size := by grind    -- 函数体里的证明
    match Ord.compare arr[i'] arr[i] with
```

Lean 的 **`have` 表达式**向上下文添加一个事实，后续的 `arr[i']` 需要这个事实才能通过索引检查。这个模式在书里只讲了一次，但实际编程中极常用。详见 [[termination-proofs]]。

## 05-safe-sort — fun_induction vs induction

```lean
theorem insertSorted_size ... := by
  fun_induction insertSorted <;> grind [Array.size_swap]
```

普通 `induction` 按数据类型的构造器归纳，但 `insertSorted` 的递归结构是对 `Fin` 的值递归。`fun_induction` 按函数自身的递归结构生成归纳原理。新手容易在 `induction` 上卡住半天，不知道该换 `fun_induction`。详见 [[tactics-and-induction]]。

终止度量也是卡点：为什么 `termination_by i.val` 是索引值递减而不是数组长度？因为每次递归 `i' = i - 1`，索引严格递减。**终止度量不一定是"数据结构变小"，任何严格递减的自然数都行。**

## 04-eval-prove — grind 不够用时手动 split

```lean
| add a b ih_a ih_b =>
    unfold simplify
    repeat' split <;> simp_all [eval] <;> omega
```

`mul` 分支一行 `grind` 搞定，但 `add` 需要 `unfold + split`。因为 `simplify` 的 add 分支有 4 个 match 分支（`num 0, sb` / `sa, num 0` / `num na, num nb` / `sa, sb`），`grind` 对某些情况不够强。**知道什么时候 grind 不够用、需要手动 split**，是这个项目最重要的经验。详见 [[propositions-and-proofs]]。

## 07-formal-verify — match 替代 induction + 递归定理

```lean
theorem eraseDups_length_le [BEq α] (xs : List α) :
    xs.eraseDups.length ≤ xs.length := by
  match xs with
  | [] => simp
  | x :: xs =>
    rw [List.eraseDups_cons]
    apply Nat.succ_le_succ
    exact Nat.le_trans (eraseDups_length_le ...) (List.length_filter_le ...)
termination_by xs.length
```

和书里教的 `induction xs` 不同——`match` 不会自动生成归纳假设，你需要**自己递归调用定理**。而且这个定理本身需要 `termination_by`——**证明也需要证明终止**。详见 [[tactics-and-induction]]。

## 06-typed-db — Universe Pattern 的必要性

```lean
inductive DBType where | int | string | bool
abbrev DBType.asType : DBType → Type
  | .int => Int | .string => String | .bool => Bool
```

为什么不直接用 `Type`？因为如果 Schema 里直接放 `Type`，那 `List (String × Type)` 的类型是 `Type 1`，后续所有定义都要升一级。用 DBType 作为编码，`asType` 作为解释函数，把宇宙层级压回 `Type`。详见 [[universes]] 和 [[dependent-types]]。

Row 的模式匹配也是卡点——为什么需要 `[col]` 单列分支？因为多列的 `×` 是右结合的，没有单列分支会导致两列时多出一个无意义的 `Unit`。

## 08-type-checker — 依赖模式匹配的五路同步

```lean
def eval : (ctx : Context) → (e : Expr) → (ty : Ty) →
    CheckResult ctx e ty → Env ctx → ty.asType
  | _, _, .nat, .num n, _ => n                    -- ty = .nat, proof = .num n
  | ctx, _, .nat, .add left right, env =>          -- ty = .nat, proof = .add
      eval ctx _ .nat left env + eval ctx _ .nat right env
  | ctx, _, retTy, .app f arg, env =>              -- retTy 由 f 的类型决定
      eval ctx _ _ f env (eval ctx _ _ arg env)
```

同时匹配**表达式**（`.add`）、**类型**（`.nat`）和**推导证据**（`CheckResult`）三样东西。返回类型 `ty.asType` 依赖于匹配到的 `ty`——`.nat` 时返回 `Nat`，`.bool` 时返回 `Bool`，`.fn a b` 时返回函数类型。

建议从 `.num` 和 `.bool` 两个简单分支开始理解，再看 `.add`（两路递归），最后看 `.app`（返回的是函数）。这是整个系列最陡的坡，理解了就完成从"会用依赖类型"到"会设计依赖类型系统"的升级。详见 [[dependent-types]]。

## 相关概念

- [[datatypes-pattern-matching]] — 归纳类型基础
- [[dependent-types]] — 依赖类型核心模式
- [[monads]] — Monad 使用
- [[functor-applicative-monad]] — Validate 的层级定位
- [[monad-transformers]] — ReaderT + ExceptT
- [[termination-proofs]] — Fin、have、termination_by
- [[tactics-and-induction]] — fun_induction、match 策略
- [[universes]] — 宇宙模式
