---
title: String 类型与字符操作
created: 2026-05-09
updated: 2026-05-09
type: entity
tags: [lean-syntax, types, string, char]
---

# String 类型与字符操作

## 核心思想

Lean 的 `String` 是 UTF-8 编码的不可变字符串。逐字符处理时通常先转成 `List Char`，处理完再转回来。

## 创建与基本操作

```lean
def s := "hello"
#eval s.length          -- 5
#eval s ++ " world"     -- "hello world"（拼接）
#eval s.push '!'        -- "hello!"（高效追加单个字符）
```

## 转换

| 操作 | 方向 |
|------|------|
| `String.toList s` | String → `List Char` |
| `String.ofList cs` | `List Char` → String |
| `s.splitOn "\n"` | 按分隔符切割为 `List String` |

逐字符处理的标准模式：`String.toList` → 处理 → `String.ofList`。

## 查找与判断

| 方法 | 类型 | 说明 |
|------|------|------|
| `s.contains c` | `Char → Bool` | 是否包含字符 `c` |
| `s.startsWith prefix` | `String → Bool` | 是否以 `prefix` 开头 |
| `s.toLower` | `String` | 转小写 |
| `s.toUpper` | `String` | 转大写 |

```lean
#eval "Hello".contains 'e'          -- true
#eval "--help".startsWith "--"      -- true
#eval "Hello".toLower               -- "hello"
```

## 连接

```lean
", ".intercalate ["a", "b", "c"]    -- "a, b, c"
```

用分隔符连接字符串列表。

## Char 类型

`Char` 是 Unicode 码点（`UInt32` 的子集）。

| 操作 | 说明 |
|------|------|
| `c.isDigit` | 是否是数字字符 |
| `c.isWhitespace` | 是否是空白字符 |
| `c.toNat` | 字符 → 码点（Nat） |
| `Char.ofNat n` | 码点 → 字符 |

## 转义序列

字符串字面量中支持转义：

```lean
"hello\nworld"       -- 换行
"tab\there"          -- 制表符
"quote\""            -- 双引号
"backslash\\"        -- 反斜杠
"\u0048\u0069"       -- Unicode 转义（Hi）
```

## 字符串 → 数字

```lean
"123".toNat?         -- some 123 : Option Nat
```

`String.toNat?` 返回 `Option Nat`，解析失败返回 `none`。

## 相关概念

- [[list-type]] — `String.toList` 转为 List 处理
- [[array-type]] — `s.splitOn "\n" |>.toArray` 转 Array
- [[polymorphism]] — 多态类型
