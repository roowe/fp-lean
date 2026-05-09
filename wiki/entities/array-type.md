---
title: Array 类型
created: 2026-05-09
updated: 2026-05-09
type: entity
tags: [lean-syntax, types, array]
---

# Array 类型

## 核心思想

`Array α` 是 Lean 的动态数组——连续内存存储，按索引随机访问，O(1) 取值。和 [[list-type|List]] 相比，Array 更适合索引密集的操作，但函数式风格下不可变。

## 创建

```lean
def a : Array Nat := #[]           -- 空数组
def b : Array Nat := #[1, 2, 3]   -- 字面量
```

## 常用操作

| 操作 | 说明 |
|------|------|
| `arr[i]` | 按索引取值（需要越界证明，见 [[termination-proofs|Fin]]） |
| `arr.size` | 长度 |
| `arr.push x` | 末尾追加元素，返回新数组 |
| `arr.swap i j` | 交换两个位置的元素，返回新数组 |
| `arr.set! i x` | 设置指定位置的值（越界则 panic） |
| `arr.toList` | 转 List |
| `List.toArray` | List 转 Array |

```lean
#eval #[3, 1, 4].push 5       -- #[3, 1, 4, 5]
#eval #[3, 1, 4].swap 0 2     -- #[4, 1, 3]
#eval #[3, 1, 4].size          -- 3
```

## for 循环

Array 支持 `for ... in` 循环，可以带证明：

```lean
-- 普通循环
for line in lines do
  IO.println line

-- 带索引的循环（h 是 i < lines.size 的证明）
for h : i in [:lines.size] do
  let line := lines[i]    -- 有 h 证明，索引安全
  ...
```

## Array vs List

| 维度 | Array | List |
|------|-------|------|
| 索引访问 | O(1) | O(n) |
| 头部插入 | O(n) | O(1) |
| 不可变操作 | `push`、`swap` 返回新数组 | `::` 头部插入 |
| for 循环 | 支持（带证明） | 需要转 Array 或用递归 |
| 适用场景 | 索引密集、排序 | 模式匹配、递归处理 |

## 项目实践

> Array 在 05-safe-sort 项目中是核心数据结构，展示了 `Fin` 安全索引和 `swap` 原地操作。

```lean
-- 05-safe-sort/SafeSort/Basic.lean — 用 Fin 保证索引安全的插入排序
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

03-grep 项目中用 `Array String` 存储文件行，通过带证明的 `for` 循环安全访问：

```lean
-- 03-grep/MiniGrep/Search.lean
for h : i in [:lines.size] do
  let line := lines[i]  -- h 证明 i < lines.size
  if (← matchLine line) then results := results.push (i + 1, line)
```

| 项目 | Array 用途 |
|------|-----------|
| 02-json-parser | `Array Char` 作为解析输入源（ParseState.src） |
| 03-grep | `Array String` 存文件行，带索引循环 |
| 05-safe-sort | `Array α` 排序核心，`Fin` 索引 + `swap` |

## 相关概念

- [[list-type]] — 链表类型
- [[termination-proofs]] — `Fin` 安全索引
- [[datatypes-pattern-matching]] — 模式匹配
