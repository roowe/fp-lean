---
title: 命题与证明
created: 2026-05-04
updated: 2026-05-04
type: concept
tags: [proposition, proof, tactic]
sources: [book/FPLean/PropsProofsIndexing.lean]
---

# 命题与证明

## 核心思想

在 Lean 中，**命题即类型**（Propositions as Types）。命题是 `Prop` 类型的类型，证明是该类型的一个值。

## 最简单的证明

```lean
def onePlusOneIsTwo : 1 + 1 = 2 := rfl
```

`rfl`（reflexivity）证明两边计算结果相同。

约定：用 `theorem` 关键字声明已被证明的命题。

## 策略模式（Tactic Mode）

`by` 进入策略模式，用小型程序构造证明：

```lean
theorem onePlusOneIsTwo : 1 + 1 = 2 := by
  decide
```

### 常用策略

| 策略 | 说明 |
|------|------|
| `rfl` | 自反性（两边定义相等） |
| `decide` | 自动决策过程（适用于具体值） |
| `simp` | 化简器 |
| `grind` | SMT 风格自动证明 |
| `exact` | 精确提供证明项 |
| `assumption` | 使用已有假设 |

## 逻辑连接词

| 连接词 | 记法 | 说明 |
|--------|------|------|
| And | `A ∧ B` | 证据是包含双方证据的对 |
| Or | `A ∨ B` | 证据是任意一方的证据 |
| True | `True` | 平凡成立 |
| False | `False` | 无证据（不可达） |
| Not | `¬A` | `A → False` 的函数 |
| 蕴含 | `A → B` | 将 A 的证据转换为 B 的证据的函数 |

```lean
theorem andImpliesOr : A ∧ B → A ∨ B :=
  fun andEvidence =>
    match andEvidence with
    | And.intro a b => Or.inl a
```

## 证据作为参数

安全索引——将越界不可能性的证据作为参数：

```lean
def third (xs : List α) (ok : xs.length > 2) : α := xs[2]

#eval third woodlandCritters (by decide)
```

`ok` 是命题 `xs.length > 2` 为真的证据，由调用者负责提供。

## 无证据索引

| 语法 | 越界时行为 |
|------|-----------|
| `xs[i]` | 编译期需要证据（最安全） |
| `xs[i]?` | 返回 `Option`（`none` 表示越界） |
| `xs[i]!` | 运行时 panic |

## 相关概念

- [[tactics-and-induction]] — 归纳证明策略
- [[dependent-types]] — 类型中包含命题
- [[type-classes]] — `Decidable` 类型类
- [[termination-proofs]] — 证明程序终止
