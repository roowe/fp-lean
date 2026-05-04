---
title: 定义相等 vs 命题相等
created: 2026-05-04
updated: 2026-05-04
type: comparison
tags: [dependent-types, proof, comparison]
sources: [book/FPLean/DependentTypes/Pitfalls.lean]
---

# 定义相等 vs 命题相等

## 核心区别

| 维度 | 定义相等（Definitional） | 命题相等（Propositional） |
|------|------------------------|------------------------|
| 判定者 | 编译器自动 | 需要显式证明 |
| 使用方式 | 隐式，无需操作 | 通过 `▸` 或 `rw` 部署 |
| 灵活性 | 仅限定义层面的化简 | 任意等式关系 |
| 典型例子 | `1 + 1` 和 `2` | `k = Nat.plusR 0 k` |

## 为什么重要

在 [[dependent-types]] 编程中，类型中出现的表达式必须能被编译器化简。如果化简"卡住"（变量处无法继续），就需要命题相等性证明来"解卡"。

```lean
-- plusL 能用在类型中（定义相等可化简）
def Nat.plusL | 0, k => k | n+1, k => plusL n k + 1
-- Vect α (n.plusL 0) = Vect α n  ← 编译器能化简

-- plusR 用在类型中会卡住！
def Nat.plusR | n, 0 => n | n, k+1 => plusR n k + 1
-- Vect α (n.plusR 0) = Vect α ???  ← 卡住在变量 n 上
```

## 解卡方法

用命题相等性证明 + `▸` 操作符：

```lean
theorem plusR_zero_left (k : Nat) : k = Nat.plusR 0 k := by
  induction k <;> simp [Nat.plusR] <;> assumption

-- ▸ 将目标类型中的 k 替换为 Nat.plusR 0 k
def appendR : Vect α n → Vect α k → Vect α (n.plusR k)
  | .nil, ys => plusR_zero_left _ ▸ ys
```

## 经验法则

- 优先选择对**正确参数**做模式匹配的函数（如 `plusL`）
- 类型中的表达式应尽量选择能让编译器自动化简的方向
- 只在必要时用命题相等性证明

## 相关概念

- [[dependent-types]] — 依赖类型编程
- [[propositions-and-proofs]] — 命题与证明
- [[tactics-and-induction]] — 证明策略
