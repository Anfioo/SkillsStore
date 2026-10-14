---
name: wos-sql-check
description: Calls WOS workflow SQL audit API POST /api/v1/workflow/sqlcheck/ with full_sql, instance_id, and db_name; parses Inception-style rows (errlevel, errormessage, affected_rows). Use when validating DML/DDL before submitting a WOS ticket, when the user mentions sqlcheck、SQL 检测、工单预检、或 Archery 审核 affected_rows；与 wos-sql-query（只读 /query/）不同。
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
# WOS SQL 检测（工单预检 `sqlcheck`）

在**不执行** SQL 的前提下，走与提单前相同的 **`/api/v1/workflow/sqlcheck/`** 审核，查看每条语句的 `errlevel`、`errormessage`、`affected_rows` 等。

## 与现有能力分工

| 能力 | 路由 | 用途 |
|------|------|------|
| **wos-sql-query** | `POST /query/` | Archery **在线只读**执行 `SELECT` |
| **本 skill** | `POST /api/v1/workflow/sqlcheck/` | 工单 **DML/DDL 预检**（预估影响行、规则告警） |
| **wos-submit-workflow** | `POST /api/v1/workflow/` | 正式 **提交工单**（内部可先调 sqlcheck） |

## 前置条件

1. `pip install requests`
2. 已用 **wos-login-simulate** 生成 `wos_login_state.pkl`（与查询/工单共用）

## 命令行

```bash
python scripts/wos_sql_check.py \
  --instance-id 46 \
  --db-name mabang_overSeasWarehouse \
  --sql "UPDATE t SET c=1 WHERE id=1"

# 从文件（可含多条语句，与浏览器工单一致）
python scripts/wos_sql_check.py \
  --instance-id 46 \
  --db-name mabang_overSeasWarehouse \
  --sql-file ./change.sql

# 登录态路径
python scripts/wos_sql_check.py -i 46 -d mabang_overSeasWarehouse -f ./change.sql \
  --login-state "$HOME/wos_login_state.pkl"

# 存在 errlevel>=1 时非零退出（用于脚本门禁）
python scripts/wos_sql_check.py -i 46 -d mabang_overSeasWarehouse -f ./change.sql --strict
```

## 请求体（与抓包一致）

`Content-Type: application/json`，JSON 字段：

- **`full_sql`**：完整 SQL 字符串（多条用分号分隔，末尾分号可有可无）
- **`instance_id`**：字符串，与工单页实例一致（如 `"46"`）
- **`db_name`**：目标库名（如 `mabang_overSeasWarehouse`）

## 响应解读（常见字段）

服务端返回体中含 **`rows`** 数组时，每条对应一段 SQL（含自动注入的 `use \`db\``）：

| 字段 | 含义 |
|------|------|
| `stage` / `stagestatus` | 如 `CHECKED`、`Audit Completed` |
| `errlevel` | `0` 正常；`>=1` 为规则告警或错误（以平台为准） |
| `errormessage` | 中文说明，如「预计一次更新(n行)超过 5000 行」 |
| `affected_rows` | **预估**影响行数（可能与在线 `COUNT` 不一致，勿盲信） |
| `error_count` / `warning_count` | 汇总计数（在顶层或内层，以实际 JSON 为准） |

脚本默认打印摘要；加 **`--raw`** 输出完整 JSON。

## 注意事项

- **凭证**：勿将 `wos_login_state.pkl`、账号密码写入仓库。
- **预估行数**：`sqlcheck` 的 `affected_rows` 受执行计划、**`OR` 条件重复匹配**等影响，可能与 `SELECT COUNT(*)` 不一致；高危变更应用 **主键 / 唯一条件** 或 **在线只读 COUNT** 交叉核对。
- **写操作**：`sqlcheck` 不执行 SQL；落库仍须走 **审批工单**（wos-submit-workflow）。

## 在代码中调用

```python
import os, sys
sys.path.insert(0, os.path.expanduser("~/.cursor/skills/m-logistics-skills/wos-sql-check/scripts"))
from wos_sql_check import load_session_from_state, run_sqlcheck

session, csrf = load_session_from_state(os.path.expanduser("~/wos_login_state.pkl"))
ok, msg, data = run_sqlcheck(session, csrf, "46", "mabang_overSeasWarehouse", "UPDATE ...")
```