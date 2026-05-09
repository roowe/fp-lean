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

## 相关概念

- [[list-type]] — 链表类型
- [[termination-proofs]] — `Fin` 安全索引
- [[datatypes-pattern-matching]] — 模式匹配
