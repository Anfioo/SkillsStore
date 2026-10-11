---
name: wos-sql-query
description: WOS 生产库只读查询 Skill。Use when querying WOS /query/、Archery 在线 SELECT、prod-tms-xjp、prod-tms-xjp-log、XJP/TMS 生产数据、日志归档数据、按 supplierId 企业维度分表查询、沉淀表结构/业务关联/失败问题。与 wos-login-simulate、wos-sql-check、wos-submit-workflow 配合使用。
applies_to: []
tags: []
scope:
- project
agents:
- claude-code
- cursor
- windsurf
- copilot
- codex
- opencode
- gemini
- antigravity
- cline
---
# WOS SQL 查询（XJP/TMS）

通过 WOS `POST /query/` 执行**只读 SELECT 排查**。本 Skill 只负责查询与知识沉淀；登录复用 `wos-login-simulate`，写操作预检/提单继续使用 `wos-sql-check` / `wos-submit-workflow`。

## 固定实例

| 目的 | instance_name | 说明 |
|------|---------------|------|
| 主业务数据 | `prod-tms-xjp` | 主实例，优先查当前业务状态 |
| 日志和归档数据 | `prod-tms-xjp-log` | 日志、归档、历史轨迹；主实例查不到状态流转时再查 |

`db_name` 以 WOS 下拉或报错提示为准。未知时先用 `SHOW DATABASES LIKE '%xjp%'` / `SHOW TABLES LIKE ...` 缩小范围，不要猜大范围查询。

## 查询前置流程

1. 已执行 `wos-login-simulate` 并生成登录态：`wos_login_state.pkl`。
2. 先读本 Skill 的沉淀文件：
   - `learned/schema.jsonl`：表结构字段沉淀
   - `learned/relations.jsonl`：业务关联沉淀
   - `learned/errors.jsonl`：失败问题沉淀
3. 只写 `SELECT` / `SHOW` / `DESC` / `EXPLAIN`。禁止用 `/query/` 执行 DML/DDL。
4. 查询必须控制范围：优先带 `supplierId` / `supplier_id` / `supplierid` 和业务单号、主键、时间范围，默认 `LIMIT 100`。

## 常用业务口径

| 查询目标 | 库 | 表 | 时间字段 | 说明 |
|----------|----|----|----------|------|
| 企业某天推单量 | `wms_order` + `wms_order_history` | `tb_task_order{supplierId % 100}` + `tb_task_order_history{supplierId % 100}` | `createTime` | `createTime` 是订单创建时间；推单量要同时统计未发货库和发货库后相加 |
| 企业某天发货量 | `wms_order_history` | `tb_task_order_history{supplierId % 100}` | `shipTime` | `wms_order` 是未发货库，不能用于发货量；发货量以历史库 `shipTime` 为准 |

推单量模板：

```sql
SELECT SUM(cnt) AS push_order_count
FROM (
  SELECT COUNT(*) AS cnt
  FROM wms_order.tb_task_order{suffix}
  WHERE supplierId = <supplierId>
    AND createTime >= '<date> 00:00:00'
    AND createTime < DATE_ADD('<date> 00:00:00', INTERVAL 1 DAY)
  UNION ALL
  SELECT COUNT(*) AS cnt
  FROM wms_order_history.tb_task_order_history{suffix}
  WHERE supplierId = <supplierId>
    AND createTime >= '<date> 00:00:00'
    AND createTime < DATE_ADD('<date> 00:00:00', INTERVAL 1 DAY)
) t;
```

发货量模板：

```sql
SELECT COUNT(*) AS ship_order_count
FROM wms_order_history.tb_task_order_history{suffix}
WHERE supplierId = <supplierId>
  AND shipTime >= '<date> 00:00:00'
  AND shipTime < DATE_ADD('<date> 00:00:00', INTERVAL 1 DAY);
```

## 企业维度与分表规则

企业 ID 统一称为 `supplierId`。表字段可能叫：

- `supplierId`
- `supplier_id`
- `supplierid`

分表规则：**按企业 ID 取模**。默认物理表后缀为：

```text
{{逻辑表}} + (supplierId % N)
```

其中 `N` 默认 `100`，可通过 `--shard-mod` 指定；如果实际表是下划线形式，用 `--shard-style underscore` 得到 `逻辑表_后缀`。

查询时优先使用占位符，让脚本展开：

```bash
python scripts/wos_sql_query.py \
  --instance-name prod-tms-xjp \
  --db-name <db_name> \
  --supplier-id 216389 \
  --sql "SELECT * FROM {{Order}} WHERE supplierId = 216389 LIMIT 10"
```

脚本会把 `{{Order}}` 展开为 `Order89`（`216389 % 100 = 89`）。

如果字段名不确定，先查结构：

```bash
python scripts/wos_sql_query.py -i prod-tms-xjp -d <db_name> --describe Order --supplier-id 216389
```

并把识别到的企业字段沉淀到 `learned/schema.jsonl`。

## 自动沉淀规则

### 表结构

当执行 `--describe`、`DESC table`、`SHOW COLUMNS` 且成功后，必须把以下信息沉淀到 `learned/schema.jsonl`：

```json
{"table":"Order89","logical_table":"Order","supplier_field":"supplierId","fields":["id","supplierId","..."],"instance_name":"prod-tms-xjp","db_name":"..."}
```

### 业务关联

当从一次排查中确认表之间的关联关系时，使用 `--learn-relation` 写入：

```bash
python scripts/wos_sql_query.py --learn-relation \
  --from-table Order \
  --to-table OrderLog \
  --on "Order.orderNo = OrderLog.orderNo" \
  --meaning "订单主表到日志表" \
  --evidence "本次通过 orderNo 查询命中"
```

### 失败问题

每次查询失败，脚本默认写入 `learned/errors.jsonl`，包括：实例、库、SQL 片段、错误分类、错误信息、时间。

当本轮修正了失败原因，必须执行：

```bash
python scripts/wos_sql_query.py --resolve-error <error_id> --fix "正确实例/库/表/字段/分表规则说明"
```

原则：失败不能只停留在聊天里，必须落到 Skill 的 `learned/errors.jsonl`，避免下次重复踩坑。

## 常用命令

```bash
# 主实例查询
python scripts/wos_sql_query.py -i prod-tms-xjp -d <db_name> \
  --supplier-id <supplierId> \
  -s "SELECT * FROM {{逻辑表}} WHERE supplierId = <supplierId> LIMIT 20"

# 日志/归档实例查询
python scripts/wos_sql_query.py -i prod-tms-xjp-log -d <db_name> \
  --supplier-id <supplierId> \
  -s "SELECT * FROM {{日志逻辑表}} WHERE supplierId = <supplierId> ORDER BY id DESC LIMIT 20"

# 打印分表后缀
python scripts/wos_sql_query.py --print-shard --supplier-id <supplierId>

# 结构学习
python scripts/wos_sql_query.py -i prod-tms-xjp -d <db_name> --describe <逻辑表或物理表> --supplier-id <supplierId>

# 查看未解决失败
python scripts/wos_sql_query.py --list-open-errors
```

## 查询节奏

1. 主实例先查当前状态。
2. 主实例无法解释状态变化，再查 `prod-tms-xjp-log`。
3. 如果表不存在，先调整分表后缀/下划线风格，再调整实例/库。
4. 如果字段不存在，优先从 `supplierId` / `supplier_id` / `supplierid` 三种企业字段和 `DESC` 结果中确认。
5. 证据链要窄：一次只改变实例、库、表名、字段名、分表后缀中的一个变量。

## 输出要求

给用户回复时只输出：

- 查了哪个实例/库/表
- 命中的关键记录或没有命中的结论
- 本次沉淀了什么结构/关联/失败修正

不要暴露登录态、Cookie、CSRF、密码或完整敏感数据。