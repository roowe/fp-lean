---
title: 终止性证明
created: 2026-05-04
updated: 2026-05-09
type: concept
tags: [termination, proof, array, recursion]
sources: [book/FPLean/ProgramsProofs/ArraysTermination.lean, book/FPLean/ProgramsProofs/Inequalities.lean, book/FPLean/ProgramsProofs/Fin.lean, book/FPLean/ProgramsProofs/InsertionSort.lean, book/FPLean/ProgramsProofs/SpecialTypes.lean]
---

# 终止性证明与安全数组

## 核心思想

Lean 必须证明所有递归函数终止。结构递归自动通过，但遍历数组等非结构递归需要显式终止度量。安全数组操作还需要索引越界的证明。

## termination_by 声明

```lean
def arrayMapHelper (f : α → β) (arr : Array α) (soFar : Array β) (i : Nat) : Array β :=
  if inBounds : i < arr.size then
    arrayMapHelper f arr (soFar.push (f arr[i])) (i + 1)
  else soFar
termination_by arr.size - i
```

`termination_by` 声明一个在每次递归调用时严格递减的自然数度量。`termination_by?` 让 Lean 建议度量。

### decreasing_by

`termination_by` 声明度量，`decreasing_by` 手动证明度量递减（当 Lean 无法自动推断时）：

```lean
def insertSortLoop [Ord α] (arr : Array α) (i : Nat) (h : i ≤ arr.size) : Array α :=
  if h' : i < arr.size then
    insertSortLoop (insertSorted arr ⟨i, h'⟩) (i + 1) (by grind [insertSorted_size])
  else arr
termination_by arr.size - i
decreasing_by grind [insertSorted_size]
```

大多数情况下 `grind` 或 `simp` 就够用。复杂场景需要手动提供引理。

## have 表达式

在递归函数中嵌入局部证明：

```lean
def mergeSort [Ord α] (xs : List α) : List α :=
  if h : xs.length < 2 then match xs with | [] => [] | [x] => [x]
  else
    let halves := splitList xs
    have : xs.length ≥ 2 := by grind
    have : halves.fst.length < xs.length := by apply splitList_shorter_fst; assumption
    have : halves.snd.length < xs.length := by apply splitList_shorter_snd; assumption
    merge (mergeSort halves.fst) (mergeSort halves.snd)
termination_by xs.length
```

## Fin 类型 — 安全数组索引

```lean
structure Fin (n : Nat) where
  val  : Nat
  isLt : LT.lt val n    -- 编译期证明 val < n
```

`Fin n` 表示严格小于 `n` 的自然数。用 `Fin` 保证数组索引安全：

```lean
def insertSorted [Ord α] (arr : Array α) (i : Fin arr.size) : Array α :=
  match i with
  | ⟨0, _⟩ => arr
  | ⟨i' + 1, _⟩ =>
    have : i' < arr.size := by grind
    match Ord.compare arr[i'] arr[i] with
    | .lt | .eq => arr
    | .gt => insertSorted (arr.swap i' i) ⟨i', by simp [*]⟩
```

## sorry — 临时占位

`sorry` 是临时占位证明，允许编译通过但不提供实际证明。用于快速原型开发：

```lean
have : halves.fst.length < xs.length := by sorry
```

## 运行时特殊类型

Lean 在运行时对某些类型有特殊表示（高效实现）：

| 类型 | 运行时表示 |
|------|-----------|
| `Nat` | 高效大整数 |
| `String` | UTF-8 字符串 |
| `Array α` | 动态数组 |
| `UInt32` 等 | 机器字 |

**类型和证明在运行时被擦除**——零性能开销。单字段结构体的构造器消失，无数据字段的构造器变为常量值。

## 项目实践

### 05-safe-sort — 用 Fin 和 have 证明数组操作安全

`insertSorted` 用 `Fin arr.size` 保证索引合法，`termination_by` 声明递归度量，`have` 嵌入局部证明：

```lean
def insertSorted [Ord α] (arr : Array α) (i : Fin arr.size) : Array α :=
  match i with
  | ⟨0, _⟩ => arr
  | ⟨i' + 1, _⟩ =>
    have : i' < arr.size := by grind
    match Ord.compare arr[i'] arr[i] with
    | .lt | .eq => arr
    | .gt => insertSorted (arr.swap i' i) ⟨i', by simp [*]⟩
termination_by i.val
```

`termination_by i.val` 告诉 Lean：每次递归调用 `i.val` 严格递减，因此函数终止。`⟨i', by simp [*]⟩` 在递归调用处构造新的 `Fin` 证明。

## 相关概念

- [[datatypes-pattern-matching]] — 结构递归
- [[tail-recursion]] — 尾递归优化
- [[tactics-and-induction]] — 证明策略
- [[dependent-types]] — 依赖类型（Fin 的基础）
