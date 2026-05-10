---
title: 策略与归纳证明
created: 2026-05-04
updated: 2026-05-10
type: concept
tags: [tactic, proof, induction]
sources: [book/FPLean/TacticsInductionProofs.lean]
---

# 策略与归纳证明

## 核心思想

策略（tactics）是构造证明的小程序。`by` 进入策略模式，策略操作**证明状态**（当前目标 + 可用假设），逐步将目标化简为 `rfl` 或已有假设。

可以把证明想象成一个游戏：
- **目标**（goal）：你要证的东西，比如 `xs.length ≥ 0`
- **假设**（hypotheses）：你手头已知的事实，比如 `ih : ys.length ≥ 0`
- **策略**：你打出的牌，每打一张，目标会变化（变简单、分裂成多个子目标、或者直接解决）

```lean
theorem list_length_nonneg (xs : List α) : xs.length ≥ 0 := by
  simp    -- 打出 simp，目标化简为 True，然后 True 自动解决
```

## 策略速查总览

按「自动化程度」从高到低排列——优先用最自动化的，搞不定再降级：

| 策略              | 一句话            | 什么时候用                    |
| --------------- | -------------- | ------------------------ |
| `grind`         | SMT 风格全自动推理    | 第一选择，能展开定义 + 推导不等式 + 用假设 |
| `simp`          | 按等式规则化简        | grind 搞不定时，精确控制展开哪些定义    |
| `omega`         | 解线性算术不等式       | 只涉及加减和大小比较               |
| `decide`        | 编译期穷举判定        | 有限的小情况                   |
| `rfl`           | 定义相等，直接搞定      | 等式两边计算结果一样               |
| `split`         | 对 if/match 分情况 | 目标里有 if 或 match          |
| `unfold`        | 展开一个函数定义       | 让 simp/grind 看到定义内部      |
| `rw`            | 用等式重写目标        | 手头有等式假设想代入               |
| `exact`         | 直接给出证明项        | 已经知道答案，不想让自动策略猜          |
| `apply`         | 用引理匹配目标        | 引理结论和目标吻合，但需要提供前提        |
| `intro`         | 引入变量或前提        | 目标是 `∀ x, ...` 或 `A → B` |
| `induction`     | 结构归纳           | 需要按数据类型递归证明              |
| `fun_induction` | 函数归纳           | 按递归函数的结构归纳               |
| `have`          | 先证一个中间结论       | 需要分步走，先建立事实              |
| `cases`         | 对假设分情况         | 手头有个假设需要拆开               |

---

## 详细解释

### `rfl` — 定义相等，秒杀

等式两边在**定义上**完全一样，不需要任何推理。

```lean
-- 04-eval-prove：num 的化简结果就是它自己
| num n => rfl

-- 08-type-checker：编译期测试
example : typeCheck [] (.num 3) = some .nat := rfl

-- 08-type-checker：闭包求值结果就是 6
example : evalClosed appSuccFiveProof = (6 : Nat) := rfl
```

**注意**：`rfl` 不需要 `by` 前缀，可以直接写。但它只能处理「计算一下就相等」的情况。如果需要逻辑推理（比如归纳），就得用别的策略。

### `simp` — 化简器

Lean 最经典的自助策略。把目标里的表达式一步步化简：展开简单定义、应用等式、去掉显然的部分。

```lean
-- 07-formal-verify：列表长度 ≥ 0
theorem list_length_nonneg (xs : List α) : xs.length ≥ 0 := by
  simp

-- 05-safe-sort：simp [*] 表示"用所有可用假设来化简"
insertSorted (arr.swap i' i) ⟨i', by simp [*]⟩
```

**带参数的 simp**：`simp [f, g, lemma]` 告诉 simp "你可以展开 f 和 g 的定义，也可以用 lemma 来重写"。不写参数时 simp 只用自己的内置规则。

```lean
-- 07-formal-verify：指定展开哪些定义和引理
split <;> simp [ih, Nat.add_comm, Nat.add_left_comm]
```

**simp 的局限**：它不会"思考"，只是机械地应用等式。遇到需要反向推理或构造的情况就无能为力。

### `simp_all` — 化简目标 + 所有假设

比 `simp` 更激进：不仅化简目标，还化简上下文中所有**假设**，并尝试用假设来化简目标。

```lean
-- 04-eval-prove：cases 之后对每种情况全面化简
cases sa <;> simp_all [eval]
```

### `grind` — SMT 风格全自动推理

Lean 4 的新策略，能同时做：展开定义、推导不等式、利用假设、反向推理。是项目中**最常用的自动化策略**。

```lean
-- 05-safe-sort：自动推导 i' < arr.size
have : i' < arr.size := by grind

-- 04-eval-prove：带参数告诉 grind 展开哪些定义
grind [simplify, eval]

-- 05-safe-sort：与 fun_induction 组合，一键证明
fun_induction insertSorted <;> grind [Array.size_swap]
```

**grind vs simp**：
- `grind` 更聪明，会做反向推理和假设传播
- `simp` 更可控，只做你指定的化简
- **优先用 grind，搞不定再降级到 simp**

**grind 的局限**：某些复杂的 match 分支（比如 04-eval-prove 的 add 分支有 4 个子情况），grind 可能在某个分支卡住。这时需要先 `split` 拆开，再对每个分支用 grind。

### `omega` — 线性算术自动求解

专门处理整数/自然数的加减和大小比较。

```lean
-- 04-eval-prove：处理化简后的算术不等式
repeat' split <;> simp_all [eval] <;> omega
```

**什么时候用 omega**：目标只涉及 `+`、`-`、`≤`、`<`、`≥`、`>`、`=` 和自然数/整数变量。比如 `n + 2 ≤ m + 3 → n < m + 2`。

**omega 不能做什么**：乘法、除法、对数等非线性运算。

### `decide` — 编译期穷举

对于有限的命题，Lean 可以在编译时穷举所有情况来判定真假。

```lean
-- 简单例子
example : (3 : Nat) < 5 := by decide

-- 也可以不写 by
example : (3 : Nat) < 5 := decide
```

**注意**：只能用于有限的、可判定的命题。命题涉及的域太大（比如"所有自然数"）就不能用。

### `unfold` — 展开定义

把函数调用替换成函数体。让后续策略看到定义的内部。

```lean
-- 04-eval-prove：先展开 simplify，再处理内部结构
| add a b ih_a ih_b =>
    unfold simplify      -- 把 simplify (Expr.add a b) 展开成 match ...
    repeat' split <;> simp_all [eval] <;> omega

-- 05-safe-sort：展开后接 split
unfold insertSort
split <;> grind [insertSortLoop_size]
```

**什么时候用**：`simp [f]` 有时不够精确，或者你想控制展开的时机。`unfold f` 只展开一个定义，更精确。

### `rw` — 重写目标

用手头的等式（假设或引理）替换目标中的表达式。

```lean
-- 07-formal-verify：用库引理 List.eraseDups_cons 重写目标
rw [List.eraseDups_cons]

-- 反向重写：← 表示从右往左替换
-- 如果 h : a = b，rw [h] 把目标中的 a 替换为 b
-- 如果 h : a = b，rw [←h] 把目标中的 b 替换为 a
```

**和 simp 的区别**：`rw` 精确地做一次替换，`simp` 会反复化简。

### `exact` — 直接给出证明项

你已经知道答案，直接喂给 Lean。

```lean
-- 07-formal-verify：引用标准库定理
exact List.length_map f xs              -- map 不改变长度
exact List.length_filter_le p xs        -- filter 结果长度 ≤ 原长度
exact Nat.add_comm n m                  -- 加法交换律

-- 07-formal-verify：构造项级证明
exact (congrArg String.reverse h).symm.trans h.symm
```

**什么时候用 exact**：当你能直接写出证明项时（通常是引用已有定理）。比 `apply` 更直接。

### `apply` — 用引理匹配目标

反向推理：拿一个引理 `A → B → C`，如果目标正好是 `C`，就生成两个子目标 `A` 和 `B`。

```lean
-- 07-formal-verify：目标从 n+1 ≤ m+1 缩减为 n ≤ m
apply Nat.succ_le_succ
-- 现在子目标变成 n ≤ m
```

**apply vs exact**：
- `apply` 匹配结论，自动生成前提作为新目标
- `exact` 必须提供完整证明，不生成新目标

### `intro` — 引入变量或前提

目标从 `∀ x, P x` 变成 `P x`，或者从 `A → B` 变成 `B`（同时把 `A` 加入假设）。

```lean
-- 04-eval-prove：引入两个变量
theorem eval_simplify (env : Env) (e : Expr) : eval env (simplify e) = eval env e := by
  intro env e    -- 其实这里不需要 intro，因为定理参数已经在签名里了

-- 07-formal-verify：引入蕴涵前提
theorem reverse_preserves_length (s : String) :
    s.toList.reverse.length = s.toList.length := by
  exact List.length_reverse s.toList

-- 更典型的 intro 用法：forall
theorem forall_length_pos : ∀ (xs : List α), xs.length ≥ 0 := by
  intro xs    -- 引入 xs，目标变成 xs.length ≥ 0
  simp
```

### `split` — 对 if/match 分情况

目标中有 `if p then A else B` 或 `match x with | ... => ...` 时，按每个分支拆成子目标。

```lean
-- 07-formal-verify：insert 的 match 拆成两个分支
split <;> simp [ih, Nat.add_comm, Nat.add_left_comm]

-- 05-safe-sort：拆开后每个分支用 grind
split <;> grind [insertSortLoop_size]
```

**什么时候用 split**：`grind` 或 `simp` 卡住时，往往是因为目标里有个 match 或 if。先 `split` 拆开，再分别处理每个分支。

### `cases` — 对假设分情况

和 `split` 类似，但操作的是**假设**而非目标。

```lean
-- 04-eval-prove：对化简结果 sa 做情况分析
-- hsa : a.simplify = sa 在上下文中
cases sa <;> simp_all [eval]
```

**cases vs split**：
- `cases h` 拆开一个假设 `h`
- `split` 拆开目标中的 if/match
- `induction` 拆开一个假设**并生成归纳假设**

### `induction` — 结构归纳

对归纳类型（如 `Nat`、`List`、自定义类型）做归纳证明。每个构造器产生一个子目标，递归构造器会生成归纳假设 `ih`。

```lean
-- 04-eval-prove：对表达式结构归纳
induction e with
| num n => rfl                           -- 基础情况：数字
| var x => rfl                           -- 基础情况：变量
| add a b ih_a ih_b => ...               -- 递归情况：加法，有两个归纳假设

-- 07-formal-verify：对列表结构归纳
induction xs with
| nil => rfl                             -- 空列表
| cons y ys ih => ...                    -- 非空列表，ih 是对 ys 的归纳假设
```

**归纳假设 `ih` 是什么**：假设你要证 `∀ n, P n`，归纳到 `n + 1` 时，`ih : P n` 是"对更小的 n 已经成立"这个事实。你需要用它来证明 `P (n + 1)`。

### `fun_induction` — 函数归纳

Lean 4 特有。按**递归函数的结构**归纳，而不是按数据类型的构造器。

```lean
-- 05-safe-sort：insertSorted 的递归结构是对 Fin 值递减
theorem insertSorted_size [Ord α] (arr : Array α) (i : Fin arr.size) :
    (insertSorted arr i).size = arr.size := by
  fun_induction insertSorted <;> grind [Array.size_swap]
```

**什么时候用 fun_induction**：
- 函数的递归结构不是简单的结构递归（比如对 Fin 索引递减）
- `induction` 生成的子目标和函数的递归调用不匹配
- 配合 `termination_by` 使用效果最好

### `have` — 先证中间结论

在证明过程中引入一个辅助事实。相当于"先证一个小引理，再用它证大定理"。

```lean
-- 05-safe-sort：先证 i' < arr.size，后面的 arr[i'] 才能通过类型检查
have : i' < arr.size := by grind
match Ord.compare arr[i'] arr[i] with
  | .lt | .eq => ...

-- 05-safe-sort：直接引用已有定理
have : (arr.swap i i).size = arr.size := Array.size_swap
```

**`have` 在函数体内也能用**：不仅限于 `by` 块里，普通函数定义中也能用 `have` 向上下文添加事实。

### `generalize` — 给中间表达式起名字

把目标中的复杂子表达式替换为一个新变量，同时添加一个等式假设。

```lean
-- 04-eval-prove：把 a.simplify 的结果命名为 sa
generalize hsa : a.simplify = sa
-- 现在目标中 a.simplify 被替换为 sa
-- 上下文中多了 hsa : a.simplify = sa
```

**什么时候用**：想对一个中间结果做情况分析（`cases sa`），但目标里直接写的是表达式而不是变量。

### `sorry` — 占位，跳过证明

暂时不证，留个坑。文件能编译通过，但 Lean 会警告。

```lean
-- 05-safe-sort：未完成的证明
theorem insertSorted_sorted ... := by
  sorry
```

**注意**：`sorry` 只用于开发中临时跳过。发布前必须全部替换成真正的证明。

---

## 组合子

策略可以像管道一样组合：

### `<;>` — 对所有子目标应用

前一个策略产生多个子目标时，`<;>` 把后续策略应用到**每一个**子目标上。

```lean
-- induction 产生多个子目标，每个都用 simp + assumption
induction k <;> simp [Nat.plusR] <;> assumption

-- fun_induction 产生多个子目标，每个都用 grind
fun_induction insertSorted <;> grind [Array.size_swap]

-- split 产生多个子目标，每个都用 grind
split <;> grind [insertSortLoop_size]
```

### `repeat'` — 重复直到失败

反复应用一个策略，直到它不再产生变化。

```lean
-- 不断 split 直到没有更多 if/match
repeat' split <;> simp_all [eval] <;> omega
```

---

## 等式操作（项级）

不进入策略模式，直接构造证明项：

```lean
-- congrArg：如果 a = b，那么 f a = f b
-- h : x = y → congrArg f h : f x = f y

-- .symm：翻转等式方向
-- h : a = b → h.symm : b = a

-- .trans：链接两个等式
-- h1 : a = b, h2 : b = c → h1.trans h2 : a = c
```

项目实例（07-formal-verify）：

```lean
-- 用 congrArg、.symm、.trans 构造项级证明
exact (congrArg String.reverse h).symm.trans h.symm
```

---

## 标准库常用引理

很多常见性质标准库已经证好了，`exact` 直接引用：

| 引理 | 含义 | 项目出处 |
|------|------|---------|
| `List.length_map f xs` | `map` 不改变长度 | 07-formal-verify |
| `List.length_filter_le p xs` | `filter` 结果长度 ≤ 原长度 | 07-formal-verify |
| `List.length_reverse xs` | `reverse` 不改变长度 | 07-formal-verify |
| `List.reverse_reverse xs` | `reverse` 两次等于原列表 | 07-formal-verify |
| `List.length_append xs ys` | `append` 长度 = 两者之和 | 07-formal-verify |
| `Array.size_swap arr i j` | `swap` 不改变数组大小 | 05-safe-sort |
| `Nat.add_comm n m` | 加法交换律 | 07-formal-verify |
| `Nat.le_trans` | ≤ 的传递性 | 07-formal-verify |
| `Nat.succ_le_succ` | `n ≤ m → n+1 ≤ m+1` | 07-formal-verify |

---

## match 作为证明策略

在 proof 中直接做 case 分析，不通过 `induction` 或 `cases`：

```lean
-- 07-formal-verify：直接 match，不生成归纳假设
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

**和 induction 的区别**：`match` 不自动生成归纳假设。你需要**自己递归调用定理**来获得等价于归纳假设的能力。

---

## 证明工作流

按这个顺序尝试：

```
目标出现
  │
  ├─ 能直接算出来？ → rfl
  │
  ├─ 是有限情况？ → decide
  │
  ├─ 一眼就能搞定？ → grind [相关定义]
  │
  ├─ grind 搞不定 → simp [相关定义]
  │
  ├─ simp 也搞不定 → unfold 暴露结构
  │                    │
  │                    ├─ 有 match/if → split <;> grind/simp
  │                    │
  │                    ├─ 有假设要拆 → cases h
  │                    │
  │                    └─ 需要归纳 → induction / fun_induction
  │
  └─ 每一步都需要精细控制 → rw / apply / exact 手动构造
```

---

## 编译期测试（example）

```lean
example : typeCheck [] (.num 3) = some .nat := rfl
```

`example` 定义匿名定理。如果代码有 bug，`rfl` 编译失败——测试不通过就编译不过。

---

## 项目实践

策略与归纳证明在以下项目中大量使用：

- **04-eval-prove** — `induction`、`grind`、`unfold`、`split`、`omega`、`generalize`
- **05-safe-sort** — `fun_induction`、`grind`、`have`（函数体内证明）
- **07-formal-verify** — `induction`、`simp`、`exact`、`rw`、`apply`、`match` 风格证明

01-calc、02-json-parser、03-grep、06-typed-db 是纯编程项目，不涉及策略证明。08-type-checker 通过依赖类型在类型层面保证安全性，仅用 `rfl` 做编译期测试。

示例：04-eval-prove 的 grind 不够用时的手动策略链（`Proofs.lean`）：

```lean
| add a b ih_a ih_b =>
    unfold simplify
    repeat' split <;> simp_all [eval] <;> omega
```

`mul` 分支一行 `grind` 搞定，但 `add` 需要手动 `unfold + split`。因为 `simplify` 的 add 分支有 4 个 match 分支，`grind` 对某些情况不够强。

## 相关概念

- [[propositions-and-proofs]] — 命题与证明基础
- [[dependent-types]] — 依赖类型中的证明
- [[termination-proofs]] — 证明程序终止
- [[tail-recursion]] — 尾递归等价性证明
