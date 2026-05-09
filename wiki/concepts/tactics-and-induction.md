---
title: 策略与归纳证明
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [tactic, proof, induction]
sources: [book/FPLean/TacticsInductionProofs.lean]
---

# 策略与归纳证明

## 核心思想

策略（tactics）是构造证明的小程序。`by` 进入策略模式，策略操作**证明状态**（当前目标 + 可用假设），逐步将目标化简为 `rfl` 或已有假设。

## 归纳策略

```lean
theorem plusR_zero_left (k : Nat) : k = Nat.plusR 0 k := by
  induction k with
  | zero => rfl
  | succ n ih => unfold Nat.plusR; rw [←ih]
```

归纳产生两个子目标：
1. **基础情况**（base case）：`0 = Nat.plusR 0 0`
2. **归纳步骤**（induction step）：假设 `n = Nat.plusR 0 n`，证 `n + 1 = Nat.plusR 0 (n + 1)`

### 策略高尔夫（Tactic Golf）

极简证明——`<;>` 将策略应用到所有子目标：

```lean
theorem plusR_zero_left (k : Nat) : k = Nat.plusR 0 k := by
  induction k <;> simp [Nat.plusR] <;> assumption
```

## 常用策略速查

| 策略 | 作用 |
|------|------|
| `rfl` | 证明定义相等 |
| `decide` | 自动决策过程 |
| `simp` | 化简（可带参数：`simp [f, g]`） |
| `simp_all` | 化简目标 + 所有假设（比 `simp` 更强） |
| `grind` | SMT 风格自动证明（强于 simp） |
| `omega` | 自动求解线性算术（整数不等式、等式） |
| `exact` | 精确提供证明项 |
| `assumption` | 使用已有假设 |
| `intro` | 引入蕴含/全称的前提 |
| `apply` | 匹配引理结论，自动生成子目标 |
| `constructor` | 分解 And 等结构 |
| `cases` | 对假设做分情况讨论 |
| `split` | 对 if/match 做分情况讨论 |
| `unfold` | 展开定义 |
| `rw` | 重写目标（`rw [←h]` 反向重写） |
| `generalize` | 给中间表达式起名字，方便后续操作 |
| `trivial` | 证明 `True` 类型目标（或简单情况） |
| `induction` | 归纳证明 |
| `fun_induction` | 函数归纳（按函数结构归纳） |
| `skip` | 占位（不证明，用于调试） |

### 组合子

| 语法 | 作用 |
|------|------|
| `<;>` | 将后续策略应用到前一个策略产生的所有子目标 |
| `have := ...` | 引入局部证明 |

### 等式操作

```lean
-- congrArg：如果 a = b，那么 f a = f b
-- h : x = y → congrArg f h : f x = f y

-- .symm：翻转等式方向
-- h : a = b → h.symm : b = a

-- .trans：链接两个等式
-- h1 : a = b, h2 : b = c → h1.trans h2 : a = c
```

### 标准库引理

很多常见性质标准库已经证好了，`exact` 直接引用：

```lean
exact List.length_map f xs        -- map 不改变长度
exact List.length_filter_le p xs  -- filter 结果长度 ≤ 原长度
exact List.length_reverse xs      -- reverse 不改变长度
exact List.reverse_reverse xs     -- reverse 两次等于原列表
exact List.length_append xs ys    -- append 长度 = 两者之和
```

### 编译期测试（example）

```lean
example : typeCheck [] (.num 3) = some .nat := rfl
```

`example` 定义匿名定理。如果代码有 bug，`rfl` 编译失败——测试不通过就编译不过。

### match 作为证明策略

在 proof 中直接做 case 分析，不通过 `induction` 或 `cases`：

```lean
theorem eraseDups_length_le [BEq α] (xs : List α) :
    xs.eraseDups.length ≤ xs.length := by
  match xs with
  | [] => simp
  | x :: xs => ...
```

## grind 策略

基于 SMT 求解器技术，自动化程度最高：

```lean
theorem BinTree.mirror_count (t : BinTree α) :
    t.mirror.count = t.count := by
  induction t <;> grind [BinTree.mirror, BinTree.count]
```

## 函数归纳（fun_induction）

按函数的递归结构进行归纳，而非按数据类型的构造器：

```lean
theorem non_tail_sum_eq_helper_accum (xs : List Nat) :
    (n : Nat) → n + NonTail.sum xs = Tail.sumHelper n xs := by
  fun_induction Tail.sumHelper <;> grind [NonTail.sum]
```

详见 [[tail-recursion]]。

## 相关概念

- [[propositions-and-proofs]] — 命题与证明基础
- [[dependent-types]] — 依赖类型中的证明
- [[termination-proofs]] — 证明程序终止
- [[tail-recursion]] — 尾递归等价性证明
