---
title: 尾递归
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [tail-recursion, recursion, performance]
sources: [book/FPLean/ProgramsProofs/TailRecursion.lean, book/FPLean/ProgramsProofs/TailRecursionProofs.lean]
---

# 尾递归（Tail Recursion）

## 核心思想

尾递归是递归调用在函数**最后一步**（尾位置）的递归形式。编译器可将其优化为跳转指令（尾调用消除），避免栈溢出。非尾递归函数通过**累加器传递风格**转换为尾递归。

## 非尾递归 → 尾递归

```lean
-- 非尾递归：递归调用在 + 的参数位置
def NonTail.sum : List Nat → Nat
  | [] => 0
  | x :: xs => x + sum xs

-- 尾递归：递归调用在尾位置，使用累加器
def Tail.sumHelper (soFar : Nat) : List Nat → Nat
  | [] => soFar
  | x :: xs => sumHelper (x + soFar) xs

def Tail.sum (xs : List Nat) : Nat := Tail.sumHelper 0 xs
```

## 证明等价性

尾递归版本和原版必须产生相同结果——这需要证明：

```lean
theorem non_tail_sum_eq_tail_sum : NonTail.sum = Tail.sum := by
  funext xs
```

### 关键辅助定理

累加器与结果的关系（允许任意初始累加器值）：

```lean
theorem non_tail_sum_eq_helper_accum (xs : List Nat) :
    (n : Nat) → n + NonTail.sum xs = Tail.sumHelper n xs := by
  fun_induction Tail.sumHelper <;> grind [NonTail.sum]
```

使用 `Nat.add_assoc`、`Nat.add_comm`、`Nat.zero_add` 等标准库定理。

## 注意事项

- **列表操作方向反转**：尾递归遍历方向与原版相反，某些操作（如构建列表）需要额外反转
- **多重递归**：树遍历等有多个递归调用的函数难以直接转为尾递归
- **函数外延性**：证明两个函数相等需要 `funext` 策略

## 相关概念

- [[datatypes-pattern-matching]] — 结构递归与终止性
- [[tactics-and-induction]] — 归纳证明策略
- [[termination-proofs]] — 终止性证明
