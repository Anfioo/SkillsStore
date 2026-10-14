---
name: check-vue-i18n
description: Manually run WMS Vue i18n check/fix hook on .vue files. Executes .cursor/hooks/check-vue-i18n.js (auto-fix $t paths, add missing cn.js keys). Use when the user /check-vue-i18n, asks to check/fix Vue internationalization, run i18n hook manually, or provides a .vue file path for i18n review.
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
# Vue i18n 手动检查（check-vue-i18n）

Hook 唯一维护位置：`.cursor/hooks/check-vue-i18n.js`（monorepo 根目录）。

**手动触发**：在 Agent 聊天输入 `/check-vue-i18n`（斜杠菜单选 Skill），不要误运行 hook 的 `.js` 或 README。

## 一条命令安装

在仓库根目录：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File ".cursor/skills/check-vue-i18n/scripts/install.ps1"
```

安装后重启 Cursor。详见 `README.md`。

## 执行方式（必须跑脚本，不要手写检查逻辑）

工作区根目录为 `d:\workSpring`。用 Shell 调用：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File ".cursor/skills/check-vue-i18n/scripts/run.ps1" -Path "<vue路径>"
```

### 路径参数

| 输入 | 示例 |
|------|------|
| 单个文件 | `-Path "wms/src/views/seller/MyCargotransitOut.vue"` |
| 多个文件 | `-Path "a.vue","b.vue"` |
| 整个目录 | `-Path "wms/src/views/logistics"`（递归所有 `.vue`） |

支持相对路径（相对工作区根）或绝对路径。用户未给路径时询问，不要猜测。

## 输出处理

1. 脚本 stdout 为 hook 返回的 JSON（`allowed`、`additional_context`）
2. 将 `additional_context` 原样整理给用户（含自动修复、新增 key、多匹配警告、裸文本提示）
3. `additional_context` 为空 → 报告「检查通过，无问题」
4. exit code 2 表示有 `additional_context`（可能已自动改文件），需人工复核

## 重要警告（必须告知用户）

Hook **会修改文件**：

- 自动修正 Vue 中错误的 `$t()` 路径（`this.$t('中文')` 与模板 `$t('中文')` 均支持）
- 孤儿 `$t('中文')`（cn.js 无此 key）→ 自动加最后一个 Extend + 修正为 `$t('ExtendN.中文')`
- 硬编码中文会写入 `wms/src/i18n/langs/cn.js` **最后一个 Extend 末尾**并改 Vue 引用
- **行数滚动**：cn.js 总行数到达 `N*1000`（如 11000+）时，自动新建 `Extend{N}`（如 Extend11）再追加，不再无限堆到 Extend10（见 `wms-i18n-guidelines.mdc` §1.3/§1.4）
- 模板属性 `title="中文"` → `:title="$t('ExtendN.中文')"`

**仅维护 `cn.js`，不自动写 `en.js`。**

手动跑之前提醒用户：先保存/提交，或确认可接受自动改动。

## Hook 能力一览

| 类型 | 行为 |
|------|------|
| 含中文的完整 `$t()` 路径（`ExtendN.字段` / `模块.字段`） | **精确检查**；存在则不改 |
| 完整 `$t()` 路径不存在、末级字段已有 | **自动修正到已有路径**，不新增 key |
| 完整 `$t()` 路径与末级字段均不存在 | **仅追加 cn.js 最后一个 Extend + 修正引用** |
| `$t()` 路径错误（唯一匹配） | **自动修复** |
| `$t()` 多 Extend/命名空间匹配 | **自动修复**，采用 findKeyPath 第一个路径 |
| 孤儿 `$t('中文')`（cn.js 无 key、无 `.` 前缀） | **自动加 cn.js + 修正路径** |
| 孤儿 `$t()` 在 `==`/`===` 比较中 | **仅警告**，不自动改 |
| cn.js 总行数到达 `N*1000`（如 11000+） | **自动新建 `Extend{N}`** 再追加，不堆到旧 Extend |
| script/template 引号内中文（唯一路径或新 key） | **自动加 cn.js + 替换** |
| 模板属性 `title`/`label` 等硬编码中文 | **自动改为 `:attr="$t(...)"`** |
| 模板裸文本（`>中文<`、标签前中文、独立行中文） | **仅警告**，建议 `{{ $t('...') }}` |
| 纯英文、`//` 注释、`<!-- -->` | **跳过** |
| URL（`http://`/`https://`） | **跳过** |
| `v-if`/`v-show`/`v-for` 行 | **跳过** |
| `==`/`===`/`!=`/`!==` 比较中的中文 | **跳过**（业务常量） |
| `console.log` 等调试行 | **跳过** |
| 含 `+` 拼接、反引号模板字符串、HTML 标签的字符串 | **跳过** |
| 调试文案（`请查看控制台`、`父组件接收到的数据`） | **跳过** |
| **cn.js 非最后 Extend 出现新 key**（对比 baseline） | **自动迁移到最后 Extend + 修正 Vue 引用** |
| **Vue 误写 `$t('ExtendN.xxx')`（N 非最后 Extend）** | **自动迁移 cn.js + 修正为最后 Extend 路径** |
| **AI 禁止手动改 cn.js Extend** | 见 `.cursor/rules/ai-i18n-no-manual-cnjs.mdc` |

## AI 新增 i18n 正确姿势

**禁止** Agent 手动 `StrReplace` cn.js 的 Extend 块或手写 `$t('Extend10.xxx')`。

| ✅ 只做 | ❌ 禁止 |
|--------|--------|
| Vue 硬编码中文 | 手动往 Extend10/11 插 key |
| Vue `$t('中文')` 无前缀 | AI 指定 Extend 编号 |

Hook + baseline（`.cursor/i18n/cn-extend-baseline.json`）会自动兜底错放。

## 规则速查（与 hook 一致）

- 主对象优先：`确定`、`取消` 等用 `$t('确定')`，不加 Extend 前缀
- 纯英文不翻译：`Ok`、`Cancel`、`search...` 保持硬编码
- 拼接只译中文部分：`"4.85*2.5CM" + $t('Extend3.标签')`
- 模板裸文本：必须 `{{ $t('ExtendN.xxx') }}`，禁止裸中文
- 完整路径精确检查：所有含中文的 `$t('ExtendN.字段')` / `$t('模块.字段')` 都按点分段校验；真实存在才保持不变
- 完整路径回退：路径不存在但末级字段已有 → 自动修到 `findKeyPath` 返回的已有路径，不重复新增 key
- 完整路径补键：路径与末级字段均不存在 → 仅追加到 cn.js 最后一个 Extend 并修正引用；比较上下文仅提示
- 多匹配默认第一个：`$t('xxx')` 匹配多个 Extend/命名空间 → 自动用 findKeyPath 返回的第一个路径；主对象同名 key 不修复
- 孤儿 `$t()`：`$t('中文')` 在 cn.js 不存在 → 自动加最后一个 Extend 并修正路径
- 业务常量不翻译：`==` / `===` / `!=` / `!==` 比较中的中文字面量或 orphan `$t()` → Hook 跳过或仅警告
- **闭合 `},` 单独成行**：往 Extend 追加 key 后，Extend 的闭合 `}` 必须另起一行、4 空格缩进，**禁止**写成 `'最后一个key': 'value',},` 跟在最后一个 key 后面。手写或改 hook 插入逻辑时都要保证：

```js
// ✅ 正确
Extend10: {
    'key1': 'value1',
    'key2': 'value2',
},

// ❌ 错误（闭合 }, 跟在最后一个 key 后面）
Extend10: {
    'key1': 'value1',
    'key2': 'value2',},
```

> hook 插入逻辑（`addKeyToCnJs`）已修正：插入新 key 时会剥掉 `}` 前空白并补 `\n    ` 让 `}` 单独成行。

详细规范见 `wms/.cursor/rules/wms-i18n-guidelines.mdc`。

## Hook 回归测试

在仓库根目录执行：

```sh
node .cursor/hooks/check-vue-i18n.test.js
```

测试使用临时 cn.js/Vue fixture，不修改真实 `src/i18n/langs/cn.js`。

## 故障排查

| 现象 | 处理 |
|------|------|
| 找不到 hook | 确认在 `d:\workSpring` 根目录执行 |
| 找不到 cn.js | 根 hook 路径为 `wms/src/i18n/langs/cn.js` |
| 工作区只开了 wms | 仍用根目录 hook：`d:\workSpring\.cursor\hooks\check-vue-i18n.js` |
| 误改了调试文案 | 检查是否含 `请查看控制台`；应被跳过 |

## 与 afterFileEdit hook 的关系

- **Hook（自动）**：Agent 编辑 Vue 时 `hooks.json` 自动触发
- **本 Skill（手动）**：用户主动 `/check-vue-i18n` 时触发

两者共用同一份 `.cursor/hooks/check-vue-i18n.js`。