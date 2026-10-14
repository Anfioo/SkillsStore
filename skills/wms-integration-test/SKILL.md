---
name: wms-integration-test
description: 执行 WMS、WMS-API 和 wms-other-run 的最小化联调测试。根据 MASC 文档形成封闭清单，自动选择 API、页面、定时器或自动验证路径，并以业务证据判定结果。
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
# WMS 联调测试

## 路径解析规则

当用户引用 `.masc-specs/<feature>/` 时，按以下顺序读取：

1. 从用户指定的 `.masc-specs/<feature>/` 目录读取实际存在的 MASC 文档：`requirements.md`、`design.md`、`tasks.md`、`verification.md`、`test-runbook.md`、`test-cases.md`、`ai-test-runbook.md`、`worklog.md` 和 `evidence/`。
2. 需要本 Skill 的执行规则时，从固定目录 `D:\workSpring\.agents\skills\wms-integration-test\references\` 读取对应文件：
   - `references/masc-scope.md`
   - `references/api-page-automation.md`
   - `references/timer.md`
   - `references/evidence-stop.md`
3. `references/*.md` 的路径始终相对于本 Skill 目录解析；MASC 功能目录仅作为业务文档来源。

MASC 功能目录缺少某个文档时，继续读取其他实际存在的文档；本 Skill 参考目录缺少当前路径所需的参考文件时，按“引用读取门禁”停止并报告。

## 不可覆盖的边界

- MASC 必测场景以 `requirements.md` 的验收标准和“待验证场景”为最高优先级；`verification.md` 只能记录执行结果，不得反向扩充需求范围。
- `verification.md` 中超出需求验收边界的历史场景必须降级为“测试建议”，不得继承为必测项或计入门禁。
- 源码、改动、日志和数据库只用于定位入口及验证，禁止据此增加本次场景。
- 源码发现的必要补充测试只能在本次结论和文档同步完成后，以“未执行，待用户决定”提示；未经批准不得执行或写入清单。
- 自动读取 `local-config.md`，默认使用其中的 `baseUrl` 和 `supplierId=1001`；已有默认值或能自动判断测试类型时禁止追问。
- 一个场景只选择一条能判定需求断言的最小路径，不默认串行执行 API、页面、数据库、日志和定时器全流程。
- 纯请求组装、过滤、参数映射或空分支拦截优先做调用边界验证；除非需求明确要求下游状态变化，不得升级成真实跨系统写回调。
- 外部写调用只有在用户或需求明确要求真实联调，且存在可安全推进的唯一测试数据时执行；否则使用 Mock、已有隔离入口或无副作用验证。
- 不启动无关服务或浏览器，不等待真实定时器周期，不把编译成功、页面提示或异步入口返回当作业务通过。
- 不新增事务、回滚或补偿机制；不输出密码、Cookie、Token 或完整连接凭据。

## Maven 定时器测试前置规则

当本次目标是测试 `wms-run` 或 `wms-other-run` 中的 Timer、定时任务或定时器调用链时，执行 Maven 测试前必须先检查对应模块的 `pom.xml`：

1. 定位 `maven-surefire-plugin` 的 `<skipTests>` 配置；
2. 先记录对应 `pom.xml` 中 `<skipTests>` 的原始文本和值；
3. 如果发现硬编码的 `<skipTests>true</skipTests>`，先临时改为 `<skipTests>false</skipTests>`，再编译和执行测试；
4. 不得只依赖 `-DskipTests=false`，因为 POM 硬编码值可能不会被该命令行参数覆盖；
5. 执行命令必须显式保留 `-DskipTests=false`，并从 Maven 输出确认存在 `Tests run:`；
6. 出现 `Tests are skipped`、没有测试报告或没有 `Tests run:` 时，不得判定测试已执行，必须先修正跳过配置；
7. 该规则只适用于本次确实测试 `wms-run`、`wms-other-run` 定时器或其测试类的场景，不修改无关模块的测试配置；
8. 测试执行、报告读取和结论记录完成后，必须把对应 `pom.xml` 恢复为修改前的原始文本；无论测试通过、失败、阻塞或中途停止，都必须恢复；
9. 恢复后必须重新读取对应 `pom.xml`，确认 `<skipTests>` 与测试前记录的值一致，未完成恢复不得结束本轮联调；
10. 修改后必须读取 Surefire 报告并核对 `Failures: 0`、`Errors: 0`，不得以 `BUILD SUCCESS` 单独判定测试通过。

该前置检查属于 S3 至 S5 的固定动作，恢复检查属于 S7 的固定收尾动作，后续执行 `/wms-integration-test` 时不得省略。

## 唯一执行状态机

前一阶段未完成不得进入后一阶段，引用文件只能补充分支细节，不能改变顺序。

```text
S0 接收 Skill + 目标/MASC 目录
→ S1 读取 MASC 引用并锁定文档范围
→ S2 形成封闭测试清单
→ S3 自动选择一条最小测试路径并读取对应分支引用
→ S4 加载 local-config.md 并检查唯一阻塞
→ S5 按清单逐项单次执行
→ S6 读取证据引用并逐项取证
→ S7 输出结论并同步相关文档
→ S8 提示未执行的必要测试建议，等待用户决定
```

最小有效输入：

```text
/wms-integration-test 开始测试 @.masc-specs/029-cargo-transit-box-spec-binding-safety-change
```

用户未填写测试类型、`supplierId`、`baseUrl` 或具体文档名时，直接进入状态机，不因此追问。

## 引用读取门禁

引用文件是分支操作的唯一细节来源。进入对应阶段前必须读取；读取失败立即停止并报告，禁止凭经验继续。

| 进入条件 | 必须先读取 |
| --- | --- |
| 用户引用 `.masc-specs/<feature>/` | 本 Skill 目录下的 `references/masc-scope.md` |
| API、页面或自动验证路径 | 本 Skill 目录下的 `references/api-page-automation.md` |
| 定时器单次或调度冒烟路径 | 本 Skill 目录下的 `references/timer.md` |
| 开始业务执行、取证、判定或处理阻塞 | 本 Skill 目录下的 `references/evidence-stop.md` |

同一引用文件本次已完整读取时不重复读取。普通非 MASC 测试只验证用户明确目标，不读取 `masc-scope.md`。

## 分支决策树

```text
目标属于 MASC？
├─ 是：读取 masc-scope → 形成文档清单
└─ 否：以用户明确目标形成清单

清单需要什么触发？
├─ 已知接口或后端逻辑 → API
├─ 必须验证页面交互 → 页面
├─ wms-other-run Timer 业务 → 定时器单次
├─ 只证明 @Scheduled 注册 → 定时器调度冒烟
└─ 文档只要求已有测试命令 → 自动验证
```

- 能自动判断时禁止询问测试类型。
- 日志和数据库只是文档断言或业务证据，不独立扩展成新场景。
- 同一场景无必要地同时走 API 和页面路径属于违规。

## 顺序门禁

发生以下任一情况，立即停止当前动作并退回对应阶段：

- 未锁定 MASC 范围就分析或执行测试。
- 未形成封闭清单就触发业务操作。
- 根据源码自行增加 MASC 文档外场景。
- 已有默认配置却重复询问 `supplierId`、`baseUrl` 或测试类型。
- 未读取对应引用就进入分支执行或判定结果。
- 启动无关服务、浏览器、定时器或扫描无关数据。

## 结论硬门禁

- 没有业务结果证据不得判定通过。
- MASC 文档场景未执行、测试被跳过、退出码非零或异步下游未完成，不得宣称全部通过。
- `verification.md` 必须按 `references/masc-scope.md` 分类必测未执行、准备、阻塞、排除范围和建议，并写入 `MASC 推进门禁`。
- 只有 `MASC 推进门禁` 明确为“门禁结论：通过”“可进入下一步：是”，后续 MASC Skill 才能继续；本 Skill 不直接执行后续推进。
- 不直接修改 `finish.status`，不通过修改 `INDEX.md` 冒充完成，不删除失败证据。
- S8 建议不属于本次范围，不影响本次通过率；只有用户批准后才能作为下一轮清单重新执行 S1-S7。

## 输出骨架

```markdown
## 测试结论
- 结果：通过 / 未通过 / 阻塞
- 类型：API / 页面 / 定时器单次 / 定时器调度冒烟 / 自动验证
- 环境：本地或测试环境
- 测试数据：supplierId、任务标识（不输出凭据）

## 关键证据
- 触发：调用方式和是否单次
- 结果：接口、日志、异步状态和数据库关键变化
- 外部副作用：实际发送 / 已 Mock / 未触发

## 问题定位
- 仅填写未通过或阻塞的最窄根因

## 测试建议（未执行，待用户决定）
- 仅在确有必要时填写建议场景、风险和原因
- 不属于本次测试范围，不影响本次结论
```