---
title: StarRocks MV 改写失败排障实录
tags:
  - starrocks
  - 物化视图
  - 排障
date: 2026-05-25
description: 用 TRACE LOGS MV 命令定位 View Delta 改写失败的真实案例
---

# StarRocks MV 改写失败排障实录

## 背景

按 [[12-month-kernel-plan|12 个月学习计划]] 走到第 1 步:多表 JOIN 物化视图。建好 MV 后,查询少 JOIN 一张表时,本该走 View Delta 改写,但没命中。

## 现象

MV 定义:`orders ⋈ users ⋈ products`,GROUP BY (dt, category, user_level)
查询只 JOIN 两张:`orders ⋈ products`

EXPLAIN 显示走的还是 base 表的 HASH JOIN,**MV 没命中**。

## 诊断:`TRACE LOGS MV`

排障的关键命令。直接看优化器为什么不改写:

```sql
TRACE LOGS MV
SELECT p.category, SUM(o.amount)
FROM orders o JOIN products p ON o.product_id = p.product_id
WHERE o.dt='2026-05-22' GROUP BY p.category;
```

输出关键行:

```
[mv_sales_with_dim] FKs [user_id] are not totally not-null in child table orders
[mv_sales_with_dim] FKs [product_id] are not totally not-null in child table orders
Rewrite ViewDelta failed: cannot compensate query by using PK/FK constraints
```

## 根因

**View Delta 改写要求 FK 列必须 NOT NULL**。

理由:如果 `orders.user_id` 可以是 NULL,`JOIN users` 会过滤掉 NULL 行,JOIN 不是无损的,优化器不能安全消除多余 JOIN。

## View Delta 改写"四要素"

| 要素 | 检查方式 |
|---|---|
| 维度表声明 `unique_constraints` | `ALTER TABLE users SET ('unique_constraints' = 'user_id')` |
| 事实表声明 `foreign_key_constraints` | `ALTER TABLE orders SET ('foreign_key_constraints' = '(user_id) REFERENCES users(user_id)')` |
| **FK 列必须 NOT NULL** | 建表时 `BIGINT NOT NULL`,事后改不了 |
| Session 开关 | `enable_materialized_view_view_delta_rewrite = true`(默认开) |

## 教训

1. 多表 MV **建表前**就要规划 NOT NULL,不是事后加
2. MV 改写出问题永远先 `TRACE LOGS MV`,不要靠猜
3. `is_active=true` + `query_rewrite_status=VALID` 并不保证一定会被改写 — 还要看具体规则的前置条件

## 相关代码位置

`fe-core/src/main/java/com/starrocks/sql/optimizer/rule/transformation/materialization/`
