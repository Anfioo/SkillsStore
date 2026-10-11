---
name: tms-wms-frontend
description: Use when developing or refactoring the wms-new/wms Vue 2 frontend, especially pages, shared WMS components, routing, i18n, tables, filters, drawers, PDA, order, shipping, picking, inbound, warehouse, and api_*.js. Applies the project's verified refactored-page styles and CommonTable, GridTable, CommonDrawerForm, DynamicFilterBar, and MultiSearchInput patterns.
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
# TMS 前端 Skill（wms）

## 仓库身份

- 路径：`wms-new/wms/`（工作区内；SVN 仓库名仍为 `wms`）
- 技术：Vue 2 + Element UI + Vuex + vue-i18n
- API：全部 `POST /tmsapi/<actionName>` → 代理 wms-api:14046

## 改代码前必读

1. `.cursor/rules/wms-refactor-redline.mdc`（业务逻辑、分支、字段、权限和最小 diff 红线）
2. `.cursor/rules/wms-frontend-guidelines.mdc`（UI、10px 主间距、表格、抽屉、滚动和边界）
3. `docs/TMS-已重构页面清单.md`（确认整页/局部状态，排除待修复页面）
4. `references/refactored-page-patterns.md`（从已重构页面选择同类布局和视觉模式）
5. `references/shared-component-playbook.md`（公共组件真实 API、slot、事件和滚动模式）
6. `references/pc-published-visual-baselines.md`（用户确认已上线的 PC 基准及验收矩阵）
7. `.masc-rosetta/guides/WMS-页面操作手册-详细版.md`（查路由操作）
8. `.masc-rosetta/guides/WMS-全页面业务与操作百科全书.md` 与 `.masc-rosetta/guides/_page-api-flows.json`（查业务上下游与页面 API 线索；最终以页面和 `src/api/` 源码为准）

## 页面重构取样流程（强制）

1. **盘点目标页**：记录全部字段、表格列、按钮、弹窗/抽屉、`v-if`/`reportType` 分支、权限、事件、API 参数和副作用。
2. **判断页面家族**：列表、自然滚动表格、扫码工作台、长表单/设置、看板、抽屉局部或 PDA。
3. **核验基准资格**：查询重构清单；优先“用户确认已上线”和“已验证”，局部模块只参考登记范围，禁止使用“待修复”页面。
4. **选择 1～2 个同类样本**：写清“参考什么、不复制什么”，先学习信息层级、区域边界、滚动责任、操作密度和状态反馈，再写 DOM/CSS。
5. **选择公共组件**：按 `shared-component-playbook.md` 使用真实 props/slots/events；公共组件是实现与基础样式基准，不复制一份仿制组件。
6. **确定唯一主滚动**：页面、卡片或表格只能有一个主滚动责任方；先画顶部、底部、左右和分页/footer 边界。
7. **最小 diff 实施**：只迁移同语义结构和样式，不删除、合并或弱化原业务逻辑。
8. **按页面家族验证**：PC 检查 1440×900、1366×768、1024×768；PDA 检查 390px、360px、340px，并更新重构清单的准确状态。

## UI 现代化重构

- **已重构页面决定项目样式**：同类型已上线/已验证页面优先于通用审美和网络模板。完整家族见 `references/refactored-page-patterns.md`。
- **公共组件决定实现骨架**：表格、筛选、批量搜索和抽屉优先使用 `CommonTable.vue`、`GridTable.vue`、`DynamicFilterBar.vue`、`MultiSearchInput.vue`、`CommonDrawerForm.vue`；选型和契约见 `references/shared-component-playbook.md`。
- **布局容器**：列表可使用 `wms-modern-page`，作业页可使用 `modern-wms-container`；这些 class 不保证全局存在，新页必须自带或明确导入完整布局 CSS。页面边缘和同级主区块默认 10px，内部使用有限间距尺度。
- **表格筛选页例外**：顶部筛选 + 主表 + 分页优先参考 `/historyorderlist` 的白底一体化，而不是机械套灰底卡片。列宽收敛到容器，表头单行，长文本省略 + hover；滚动条和末行必须停在分页上方。
- **滚动模式**：表格内部滚动使用 `CommonTable`；页面自然滚动、sticky 表头或自适应最大高度优先 `GridTable`。禁止同时制造页面、卡片和表格三层滚动，禁止复制其他页面的 `calc()`、sticky `top` 或 `bottomOffset` 常数。
- **筛选与批量搜索**：动态条件使用 `DynamicFilterBar`；单号/SKU 多项精确搜索使用 `MultiSearchInput`，根据 API 语义选择逗号串普通模式或保留换行的 `batchMode`。
- **抽屉**：统一使用 `CommonDrawerForm`；先核对真实 props 和 `body/footer` slots，再选择同业务/同复杂度参考。新增编辑 drawer footer 默认居中，普通 dialog footer 默认右对齐。
- **多选级联边界**：长标签必须同时处理 tags、搜索 input 和 suffix 空间，并在三种 PC 视口检查单选、多选 `+N`、清除和下拉展开。
- **克制设计**：禁止渐变大标题、玻璃拟态、发光重阴影、全页胶囊化、无业务意义统计卡和营销式留白。

## 高频页面

| 业务 | 路由 | API 模块 |
| ------ | ----------------- | --------------------------------- |
| 待发货 | `/shippinglist` | `api_order.js` |
| 配货 | `/picking_*` | `api_pick.js`, `api_ordertask.js` |
| 分拣 | `/shipping_split` | `api_ordertask.js` |
| 发货 | `/shipping_*` | `api_wms_order.js` |
| 签收 | `/package_sign` | `api_stockwarehouse*.js` |
| 上架 | `/upper_shelf` | 入库 task API |

## i18n 规则

- 模板/JS 用户可见字符串：`$t('中文原文')`
- 新增键先写 `src/i18n/langs/cn.js`，再补 en 等
- 禁止硬编码中文
- 详细规范：`.cursor/rules/wms-i18n-guidelines.mdc`
- **手动检查/修复**：`@check-vue-i18n` → `.cursor/skills/check-vue-i18n/SKILL.md`（会改 Vue + cn.js，仅自动写 cn.js）
- **自动检查**：Agent 编辑 `.vue` 后 `hooks.json` → `afterFileEdit` 触发同一份 `check-vue-i18n.js`

## 菜单 homeid

订单管理=7，出库作业=3，采购入库=2，库内=5。路由 `meta.homeid` 与后端菜单一致。

## 自检清单

- [ ] ESLint 无新增 error
- [ ] 新文案已 i18n
- [ ] API action 名与 wms-api Controller 一致
- [ ] 未破坏 orderStatus/status 流转
- [ ] 组件 destroy 时清理定时器/监听
- [ ] 新路由：父组件是否为 `Home.vue`（业务页）或顶级路由（独立预览）；**禁止** `Index.vue` 作父路由
- [ ] 模板新增变量已在 `data()` / `computed` 声明
- [ ] Vue2 语法/编译验证已执行：优先 `npm run build`；如沙箱系统权限失败，已用非沙箱权限重跑；不能只依赖肉眼或单一 grep
- [ ] 模板新增组件、变量、方法、事件、slot、动态属性均已核对存在且 Vue2 写法合法
- [ ] 运行 i18n hook 或批量替换后，已检查 JS/template 整体语法，并 grep `\.:[A-Za-z_$]`、`Extend\d+\.Extend\d+` 等 hook 过度修复风险模式
- [ ] 新页 layout CSS 已自带（`wms-modern-page` 等非全局）
- [ ] **（红线）旧页重构**：已对照重构前源码逐项核对表头/表体/展开行/筛选/操作/脚本，无字段与逻辑丢失
- [ ] **GridTable / CommonTable 服务端分页页**：已按 CustomerList 标准做「页面自然滚动 + 筛选滚走后表头吸顶 + wrap 内 sticky 分页」（未隐藏主滚动条）
- [ ] **完成状态**：已更新 `docs/TMS-已重构页面清单.md`，准确标注整页/局部范围；最后一次用户否定反馈、运行时受阻或未收敛的大 diff 均未误报为“已完成”
- [ ] **同类基准**：已记录参考的已上线 PC 路由及参考边界，没有机械复制不相干业务结构
- [ ] **Dialog / Drawer footer 区分**：`el-dialog` 操作按钮位于右下角；`CommonDrawerForm` 新增/编辑类 footer 默认居中；未混用对齐规则，按钮顺序、显隐、权限、事件和状态均未改变。
- [ ] **CommonDrawerForm 抽屉**：已先核对组件真实 props/slot/滚动结构，再按“同业务 → 同复杂度 → `/mycustomerlist` 兜底”选参考；body 单滚动、footer 安全距离及弹层边界已验证
- [ ] **边界矩阵**：已检查 1440×900、1366×768、1024×768 的主操作、换行、横向/双滚动、表格分页、弹层裁切、长文本、空状态和展开区空白
- [ ] **扫码链路**：作业页的 `v-focus`、`ref`、Enter、音频、清空、成功/异常后焦点恢复均未被样式重构破坏
- [ ] **反模板味**：未无依据增加渐变头图、巨型图标、玻璃拟态、发光重阴影、胶囊化或无业务意义统计卡

## UI 重构参考页（生成新页面必须先选同类基准）

完整用途、边界规则和验收矩阵见：`.cursor/skills/tms-wms-frontend/references/pc-published-visual-baselines.md`。

| 类型 | 路由 | 文件 |
| --- | --- | --- |
| 普通新增/编辑抽屉兜底 | `/mycustomerlist` | `views/customer/MyCustomerList.vue` |
| 窄幅单列仓库抽屉 | `/warehouselist` | `views/warehouse/WarehouseList.vue` |
| 宽幅货架/复杂配置抽屉 | `/updateWarehouse` | `views/warehouse/UpdateWarehouse.vue` |
| 商品多分区抽屉（仅已验证抽屉） | `/stockWarehouselist` | `views/stock/StockWarehouseList.vue` |
| 高级查询抽屉（仅已验证抽屉） | `/shippinglist` | `views/ordermanagement/components/OrderSearchDrawer.vue` |
| 多条件 + sticky 摘要抽屉（仅已验证抽屉） | `/logisticsrulelist` | `views/logistics/LogisticsruleList.vue` |
| 白底筛选列表 | `/historyorderlist` | `views/ordermanagement/HistoryOrderlist.vue` |
| CommonTable / GridTable / 吸顶列表 | `/customerlist`、`/transfergoods`、`/myallocatetask`、`/warehouselist` | 对应页面文件 |
| 首页数据看板 | `/index` | `views/Index.vue` |
| 入库扫码/签收/贴标 | `/package_sign`、`/purchasing-labeling`、`/fast_package_sign_and_ship`、`/pre_sign_basket` | `views/purchase/` 对应页面 |
| 出库称重/验货/波次/发货/配货 | `/order_weight`、`/Packaging_inspection`、`/AutoWaveStation`、`/shipping_single`、`/shipping_multiple`、`/picking_single` | `views/order/` 对应页面 |
| 退货工作台 | `/returnsReceiving`、`/scanReturn` | `views/ordermanagement/` 对应页面 |
| 仓库列表/编辑/调拨任务 | `/warehouselist`、`/updateWarehouse`、`/replenishTask` | `views/warehouse/`、`views/replenish/` 对应页面 |
| 系统配置 | `/system_config` | `views/system/SystemConfig.vue` |
| 入库任务列表 | `/stockwarehouse_task` | `views/task/StockWarehouseTask.vue` |
| 员工管理 | `/employee_management` | `views/system/EmployeeManagement.vue` |

以上新增 PC 基准由用户在 2026-07-24 明确确认为已上线页面。选择参考时忽略当前工作区未提交改动，不得把脏工作区状态误判为线上标准变化。

### CommonDrawerForm 抽屉参考选择（强制）

1. **先看公共组件**：`src/components/Common/CommonDrawerForm.vue` 是 props、`body/footer` slot、关闭流程、append-to-body、body 滚动与 footer 固定方式的结构权威；页面调用里组件未声明的属性不能当成稳定能力。
2. **再选最近业务参考**：只从已上线或登记为已验证/局部已验证的实际用法中选择；优先同业务，再看字段密度、是否长表单、是否有内嵌表格/上传/摘要、是否需要宽幅布局。
3. **无近似用法才兜底**：普通新增/编辑表单使用 `/mycustomerlist`；它不是所有抽屉的唯一模板。窄单列表单用 `/warehouselist`，宽幅复杂配置用 `/updateWarehouse`，商品多分区用 `/stockWarehouselist` 已验证抽屉，高级查询用 `/shippinglist` 的 `OrderSearchDrawer.vue`，大量条件与摘要用 `/logisticsrulelist` 已验证抽屉。
4. **只迁移适配部分**：学习标题/分区、标签位置、栅格回落、滚动归属、footer、边框和空白控制；不复制目标页不存在的步骤、摘要、上传、表格、供应商分支或保存逻辑。
5. **边界验收**：body 单滚动且 `min-height: 0`，footer `flex-shrink: 0` 并留底部安全距离；1024px 下长标签、校验、复合输入和上传列表可用；select/date/cascader/popover/tooltip 不被裁切；无内外双滚动、footer 遮挡、双边框、灰缝或大块空白。

公共组件：`CommonTable.vue`、`GridTable.vue`、`CommonDrawerForm.vue`、`DynamicFilterBar.vue`、`MultiSearchInput.vue`

### PDA 页面 UI 标准（强制参考）

重构 PDA 页面时，优先参考已完成页面：`pda_inventory_task.vue`(`/inventory_by_task_p`，**任务盘点/扫码任务卡验收基准**)、`pda_index.vue`、`pda_cargo_transit_checkout.vue`(`/cargo_transit_check_out_p`)、`pda_cargo_transit_sign_shelf.vue`(`/shelfByBox_p`)、`pda_boxlist.vue`、`pda_cargo_shelf_by_task.vue`(`/shelfByTask_p`)、`pda_wavePick.vue`、`pda_pre_sign_basket.vue`、`pda_show_other.vue`、`pda_return_order.vue`、`pda_returnsReceiving.vue`、`pda_returnBoxing.vue`、`pda_find_stock.vue`(`/stockWarehouselist_p`)、`pda_produceTask_delivery.vue`、`pda_replenishTask.vue`。

- **统一风格**：灰底 `#f0f2f5/#f5f7fa` + 白色卡片 + 轻阴影，页面根容器/主内容 **padding: 10px**，卡片间距 **margin-bottom/gap: 10px**；PDA 页面字体最小 **14px**（图标、装饰线除外），不要再使用大面积蓝色渐变头图。
- **顶部操作区**：使用 `top-card/header-section/top-bar` 一类白色卡片承载扫描输入、筛选和主要按钮；扫描输入要占满宽度、带 `el-icon-full-screen/search` 前缀，关键按钮右侧或下一行，PDA 小屏可纵向堆叠。
- **列表卡片**：使用 `modern-card/task-card/wms-card/replenish-card` 风格，圆角 8~12px、1px 浅边框、轻阴影；标题行放主编号/SKU/批次，详情放灰底信息块或 label/value 行，数量/状态用 chip 或浅色 tag。
- **空状态/提示**：用居中 icon + 简短提示，错误/当前扫描提示用浅色 `tips-bar/empty-hint`，不要只留裸文本。
- **重构红线**：只换布局和 class，不删 `v-model/ref/@keyup.enter/@click/v-if/v-show`，改完逐项核对原字段、按钮、弹窗、API 入参。

#### 任务盘点页验收基准（`pda_inventory_task.vue`）

- **顶部两行结构**：第一行任务图标/标题/数量 + 固定宽主按钮；第二行自适应扫码框 + 固定宽操作提示。340px 下不隐藏标题、不换行、不让按钮吞掉文字。
- **扫码输入**：只保留一个扫描图标和一个输入壳；NutUI 必须关闭自带图标/搜索按钮并清理内部重复边框、背景和 padding。
- **主题色**：普通主操作统一 `#4697f3`；浅蓝用于辅助操作和提示；红色只用于删除、库存盘0等危险操作。
- **任务卡字段**：图片、库存 SKU、卖家 SKU、商品名称、虚拟 SKU、备注、仓库、仓位、库存数量、盘点数量、已配货量及权限操作必须明确可见；不能用无标签省略导致字段视觉消失。
- **步骤提示**：静态说明不用 `nut-sidenavbar`；节点/数字/连接线共享固定 30px 轨道，数字居中、所有节点严格同一垂直线，长文案不挤压轨道。
- **否决项**：红色主题残留、双图标/双输入壳、按钮文字换行或挤掉标题、步骤节点/连线错位、原业务字段或事件不可见，任一出现均不得验收。
- **验证**：至少检查 390px、360px、340px；核对 `ref/v-model/@keyup.enter/@click/v-if/v-show` 与原实现一致，并执行 `ReadLints`、`npm run build`。

Demo 预览（挂 Home 壳，有顶栏侧栏）：

- 表格筛选 `#/demo_filter_table` → `/home` children，`homeid: "1"` → `views/demo/DemoFilterTable.vue`
- 仓库工作台 `#/demo_warehouse_console` → `/purchase` children，`homeid: "2"` → `views/demo/DemoWarehouseConsole.vue`
- 列表 `#/demo_modern_list` → `/home` children，`homeid: "1"`
- 操作台 `#/demo_modern_console` → `/purchase` children，`homeid: "2"`

## 路由速查

```text
业务页  → Home.vue + children + meta.homeid + meta.parentpath
独立页  → 顶级 { path: '/xxx', component: ... }（同 print 系列）
错误示范 → Index.vue 作 parent（无 router-view，子页不渲染）
```

## 对话沉淀（Agent 每轮自动追加，勿重复提问）

- **PDA 查询页新增筛选条件时要同步查询入参和持久化链路**：只增加下拉框会出现选中后接口不带值，或刷新、退出登录后恢复不完整。正确：同时补齐筛选数据源、`Search` 字段、查询 API 参数、条件变更触发、localStorage 写入/恢复；如果退出流程会 `localStorage.clear()`，还要确认对应存储键在清理前后被显式保留。见 `src/views/pda/pda_find_stock.vue`、`src/App.vue`。
- **整批 FBA 发货参考装箱页时只能借壳，原表格列不得提升到摘要**：`FbaShip.vue` 参考 `FbaBoxing.vue` 重排时，曾因批次、仓库和状态当前读取父级 `fbaTask`，误判为每箱重复信息并移到表格外，破坏了未来逐行数据可能不同的列语义。正确：只参考单面板操作区、分组工具栏和无分页 `GridTable external-scroll` 自然滚动，不复制扫描、装箱、箱子增删等装箱业务；“补货批次/申报信息、发货仓库/目标仓库、箱子、状态”等所有原表格列即使当前绑定父级对象也必须逐行原位保留，状态与 PDF 条件继续位于状态列，并用专项测试保护列顺序、绑定和条件归属；无绑定值的历史孤立标签只有在用户明确授权后才可删除，本轮用户已授权删除“发货批次”，表格列仍必须完整保留。见 `src/views/fba/FbaShip.vue`、`src/views/fba/FbaBoxing.vue`。
- **批量订单号/运单号搜索的分页扩容不能覆盖用户选择**：页面使用 `isPageSizeManuallyChanged`，初始值为 `false`；首次及用户未手动修改页大小时允许按批量输入数量动态扩容 `pageSize`。分页 `size-change` 将其设为 `true`；此后的搜索、筛选、翻页和业务刷新均保持用户选择的页大小，当前页全选只遍历接口当前返回页。见 `src/views/ordermanagement/ReturnOrder.vue`、`tests/return-order-batch-page-size.test.mjs`。
- **全量接口的前端分页必须先筛选后切片**：若外层搜索只作用于当前页，用户会漏掉完整 API 结果中的匹配项；原因是完整源、筛选结果与当前页列表语义混用。正确：外层搜索先过滤完整源再分页切片，三类列表保持独立；展开子项搜索继续基于各行完整子源；分页事件只本地切页、不重请求后端，并触发表格布局重算。见 `src/views/logistics/LogisticsList.vue`、`tests/logistics-list-grid-table.test.mjs`。
- **白底筛选表格不要套卡片壳**：物流授权这类“顶部筛选 + 主表 + 分页”页面若给 table-wrap 增加四周边框、4px 圆角和阴影，会形成用户不需要的卡片感，并与白底一体化列表标准冲突。正确：表格外壳保持纯白、无外框、无圆角、无阴影；筛选区去卡片后同步检查 query 底部 padding 与 table wrapper 顶部 padding，避免两层留白叠加成明显断层；组合输入不能只设 `.el-input__inner`，还要统一外层高度、`.el-input-group__prepend`、line-height 与 box-sizing；只保留表头、数据行、展开内容和分页自身必要的浅分隔线；局部去卡片时不要顺带改业务结构。见 `src/views/logistics/LogisticsList.vue`。
- **表格末尾 sticky 分页器会被父容器带起**：`position:sticky; bottom:0` 只在父容器边界内吸底，滚到表格最后一行时 footer 回到文档流位置，视觉上会向上移动。正确：需要全程不动时改用 `position:fixed`，按页面根节点 `getBoundingClientRect()` 动态同步 left/width，并用 footer 实际 `offsetHeight` 给 table wrapper 增加等高底部占位；resize/ResizeObserver 更新，销毁时成对清理，避免末行被遮挡。见 `src/views/logistics/LogisticsList.vue`。

- **参考页必须匹配业务阶段，不能只看都是扫码页**：`FbaBoxing.vue` 把 `/pre_sign_basket` 的双阶段左右卡片骨架套到单一连续装箱后，顶部出现空白、窄摘要卡和碎片状态条，用户仍判定很丑。正确：先判断目标是“多阶段并行任务”还是“单一连续扫码”；FBA 分区装箱采用 `/package_sign` 的单面板层级，并参考 `/fast_package_sign_and_ship` 的工具密度，按“主扫描与任务操作 → 当前箱/批次/FNSKU 状态 → 批量与打印管理 → 表格”纵向组织；不增加页面标题、渐变卡或统计卡。见 `src/views/fba/FbaBoxing.vue`。

- **视觉重排不能把重复入口、单位或父级表头当冗余删除**：`FbaBoxing.vue` 迁移到 `GridTable` 时，曾把复制数量后的第二个 `copyTheBox(0)` 入口改成普通“个”文本、漏掉新增箱子的“指令”、去掉数量单位“个”，并用自定义 header 覆盖“商品”一级合并表头。原因是只按字段绑定和方法是否仍存在判断“逻辑保留”，没有逐节点核对可见文案、重复入口和多级表头。正确：对照 SVN 逐项保留每个可点击节点、完整文案、单位、空值语义、字号层级和父/子表头；只更换容器与样式，并用专项测试约束入口次数和表头层级。见 `src/views/fba/FbaBoxing.vue`、`tests/fba-boxing-ui.test.mjs`。

- **列设置图标与页面按钮不在同一行或迁移后反复拉伸**：`CommonTable` 的 toolbar 默认绝对定位；把整个 `vxe-toolbar` 搬进顶部固定宽 host，即使覆盖 flex 和宽高，仍会受内部 `buttons-wrapper/tools-operate` 布局影响而挤压图标。正确：按钮 flex 容器内预留 28px host，只从 `CommonTable` 根节点取真正的 `.vxe-custom--wrapper` 移入 host，不搬整个 toolbar；host 需保留 `vxe-toolbar` 上下文让依赖祖先的官方样式生效，同时显式清零 toolbar 默认 `.6em` padding、背景并设 `box-sizing:border-box`，否则仍会形成看似无内容的右侧空槽；wrapper 和直接子 `.vxe-button` 锁 28px 正方形，图标只设 `font-size:14px; line-height:1; width/height:auto; transform:none`，保留 wrapper 内列设置弹层；若业务明确要求某个操作按钮贴最右侧，host 应放在按钮组前面，让目标按钮成为最后一个 flex item，不能固定把 icon 排在末尾；mounted/updated 保持挂载。见 `src/views/logistics/LogisticsList.vue`。

- **旧 FBA 补货任务的筛选、表格与分页边界难统一，列设置接入后又出现表头上方巨大空白**：使用 `ul/li` 表格 + `scroll-float-fixed` + `footer-float-fixed` 时，筛选换行、列对齐、吸顶和末行截止难统一；迁移表格壳后若直接开启内置 toolbar，再给 wrapper 写 `padding-top: 34px`，会在操作区与表头之间制造明显硬占位，公共 `.vxe-cell` 的默认大 padding 也会让整表过松。正确：筛选 label + 控件使用语义 `filter-item` 防分离，主表迁移为 `GridTable` 百分比列，隐藏 vxe expand 列并与原 `item.ischecked` 同步，筛选滚走后仅表头 fixed，分页放在 table wrap 内 sticky；要与 `BoxlingList.vue` 一样少量数据时仍稳定在页面底部，必须同时让页面根节点 `min-height:100%; display:flex; flex-direction:column`、table wrap `flex:1; min-height:0`，footer 使用 `margin-top:auto` 和稳定最小高度；否则仅写 `position:sticky; bottom:0` 会紧跟短表出现在上方，并在父容器边界处被向上带动；列设置对齐 `StockWarehouseTask.vue` / `TransferGoods.vue`，把按钮放在独立或现有操作行最右侧，表格 wrapper 不做顶部硬占位，并用页面限定的足够 specificity + `!important` 覆盖 GridTable 默认 cell padding 到紧凑密度；顶部短筛选项可把外置 label 放入已有语义的 placeholder 降低拥挤，但任务状态、商品状态等状态分组 label 必须保留；vxe 展开内容若背景消失、贴边且内外表头同色，原因通常是错误依赖 `.vxe-table--expanded .vxe-body--expanded-cell` 层级选择器，而真实 DOM 需直接匹配 `.vxe-body--expanded-column` / `.vxe-body--expanded-cell`，同时 GridTable 还会覆盖展开区字体与单元格。正确：使用页面局部的直接选择器配合 `!important` 设置展开承载区背景和左右 padding，内部 cell 清除 padding，每个详情块使用白底轻边框，内层表头使用区别于主表的轻蓝灰；筛选控件仅设置 `.el-input__inner` 不能保证同高，因为 Element mini 的外壳、组合 prepend、select/cascader 后缀图标、range input/icon；页面以 `/boxlingList` 为同类基准时，普通控件采用统一 32px 契约；Seller SKU 这类 Element input-group 必须直接复用原生 `el-input size=small + slot=prepend el-select`，只给 select 保留业务宽度，禁止页面重画 prepend 边框、清除内层边框或分别锁外壳/inner/suffix 高度，否则即使数值都是 32px，inline-table/table-cell 的实际边框和基线仍会错位；日期区也不能残留 28/26px 混合高度；多行筛选区的上下间隔必须由唯一父容器负责，参考 `BoxlingList.vue` 使用 `padding:8px 10px; column-gap:10px; row-gap:8px`，不要再给 filter item 加 margin-bottom、给状态/操作行加补偿 padding，否则分隔线独占行后会形成多套间距叠加/separator 和按钮会被项目全局规则分别控制。正确：在页面局部成套统一控件外壳、inner、图标、日期内部元素和按钮高度，并对容易被全局覆盖的节点使用必要的高 specificity / `!important`；响应式筛选不能用“按钮固定 68px + 独立 flex item”凑总宽，这会裁切多语言按钮，并让主输入与操作拆行。正确：把强关联的关键字 + 搜索作为不可拆 flex 组，按钮使用 `width: auto` 和按内容设置的 `min-width`，时间类型 + 日期也保持不可拆；在 1024px 内容宽度下按控件、gap 与 padding 的真实总宽局部收紧前排控件，让次级物流和标签自然落到第二行。Element `el-input` 的 prepend 内嵌 select 时，默认或全局 `.el-input-group__prepend .el-select { margin: 0 -20px }` 与 prepend padding 会把前置区撑宽，造成箭头偏位和交界双边框。正确：给内嵌 select 增加页面局部 class，以更高 specificity 和必要的 `!important` 重置 margin、width、padding，保持 input-group 的 `inline-table` 布局，并让 prepend 与主 input 只保留一条中间边线；静态总宽与 CSS 优先级断言只能证明理论成立，不能替代实际浏览器边界验收；只有静态证据时必须明确“待运行时复核”，不得声称实际页面已修好；flex 筛选区在全宽状态行、操作行之间插入 `hr` 时，主容器 `row-gap` 会在分隔线前后各计算一次，再叠加全局 `.line-style-solid` 的 margin / height，造成按钮区异常大间隔。正确：主容器将 `row-gap` 归零，由筛选控件自身 margin 维持换行节奏；`hr` 明确将 height、margin、border 归零后只保留单边线，状态行与操作行使用显式 padding 控制纵向间距；vxe 主表百分比列总和不足 100% 时不会自动均分，最右列后会留下相应比例空白。正确：迁移后静态解析所有可见列，排除 `width="0"` 的隐藏 expand 列并断言总和为 100%，宽度增量按业务列均衡分配，不能全部堆到最后一列；静态总和通过后实际视觉仍须浏览器复核；FBA 补货/装箱这类服务端分页、页面自然滚动、含展开行、列设置和吸顶表头的页面统一使用 `GridTable`，配置 `:external-scroll="true"`、`height="auto"`、`:scroll-y="{ enabled: false }"`；`CommonTable` → `GridTable` 迁移可保留同一 ref，因为 `GridTable` 等价暴露 `getInstance()`、`recalculate()`、`setRowExpand()`、`setAllRowExpand()`；只替换表格壳和配置，不改列、展开、API 或分页。迁移前后用 `v-model`、`click`、`change`、`v-if`、`v-show`、API 调用计数及精确绑定审计，保留所有 Dialog 和 API 数据形状，并运行专项测试 `tests/replenishment-plan-grid-table.test.mjs`。见 `src/views/fba/ReplenishmentPlan.vue`。

- **展开明细重复相同表头或误把不同表头强行合并**：`ReplenishmentPlan.vue` 若把 `v-for="(child,index1) in item.fbaTaskItemList"` 放在包含表头和数据行的外层，会形成“表头/数据、表头/数据”的重复组；反过来若仅依据父任务的 `pickedNumCount` 渲染“已配”列，又会把本应不同列结构的子记录强行放进一张表。正确：按子记录实际列结构分组，例如用 `Number(child.pickedNum) > 0` 形成“有已配列/无已配列”两类；每类只渲染一个 `.replenishment-expand-row` 和一个表头 `ul`，数据 `ul` 循环该组 items，不同结构分别展示。展开 `ul` 使用 `align-items:center` 做单元格垂直居中。GridTable 复合表头保留原 `title` 元数据，并通过 `#header` 插槽逐项换行，页面搜索按钮放在全部筛选控件之后、分隔线之前，作为最后一个筛选控件随前项自适应排列；改后运行 `tests/replenishment-plan-expand-grouping.test.mjs` 与 `tests/replenishment-plan-grid-table.test.mjs`。见 `src/views/fba/ReplenishmentPlan.vue`。

- **主表与展开明细操作入口视觉杂乱或样式完全失效**：内层操作列过窄会把复制、编辑、停用和配置操作挤成随机两行；重组为两层 DOM 后若容器从 `logistics-action-links` 改名，却仍依赖 `.logistics-action-links .logistics-action-button` 选择器，所有轻按钮样式会失效并退回裸文字链接。正确：保留所有按钮、权限和事件；内层操作列按真实数量提高到约 27%～30% 且总比例保持 100%；第一层主操作 flex、第二层配置操作左对齐；重组 class 时同步审计模板与 CSS 选择器命中，复用通用按钮视觉必须保留 `logistics-action-links` 基类，再叠 `logistics-channel-actions` 专项布局；按钮 23～24px、4px 圆角，普通浅蓝、危险浅红、启用浅绿、配置浅灰。见 `src/views/logistics/LogisticsList.vue`。

- **整批作业表格不应为统一外观强加分页**：`FbaBoxing.vue` 的箱子数据由分区任务一次返回，批量修改、删除和打印针对整批箱子；无服务端分页契约时新增前端分页会改变全选与作业可见范围。正确：使用 `GridTable external-scroll` 保持整批自然滚动，不渲染分页器；操作区滚走后仅表头吸顶，保留原 `checkFlag/checkTaskid`、扫码焦点、事件和 API payload。见 `src/views/fba/FbaBoxing.vue`。

- **操作日志完整标准化不能只换表格外壳**：旧页依赖 `&nbsp;`、`ul/li + pct*`、`$globals.calculateScrollFloat()` 与全局 `footer-float-fixed` 时，筛选对齐、列宽、吸顶和分页截止会同时失控；迁成 `CommonTable` 后只统一控件 `height` 仍不够，`.el-form-item__content` 行高和按钮 inline 基线会让按钮底部错位；把 label 搬进 placeholder 时若继续拼「请输入条件/根据姓名查询」，又会造成冗余长提示；仅关闭 vxe hover 配置也可能被公共样式覆盖。正确：使用白底一体化筛选区与百分比六列表格，保留原 4 个筛选字段、6 列、API 参数和分页规格；筛选项与 content 统一 flex、height、line-height，按钮用 inline-flex，短字段 placeholder 只留字段名、日期保留开始/结束语义；不需要行 hover 时同时设 `row-config.isHover=false` 和页面局部白底兜底；监听 `.main-topmenu-content-wrapper`，筛选滚走后仅 fixed 表头，分页置于 wrap 内 sticky，并成对清理监听/body class。交付必须实际尝试 1440×900、1366×768、1024×768 浏览器走查；若登录/滑块阻断，保存截图和 `runtime-status.json`，状态保持待运行时复核，不能以 build/静态扫描冒充视觉验收。见 `src/views/fba/OperationLog.vue`、`tests/operation-log-standardization.test.mjs`。

- **列表操作列按最复杂行固定大比例会形成大片空白**：`LogisticsList.vue` 的操作列为容纳已授权行多个操作设成 40%，但大量未授权行只有“授权”一个入口，右侧长期空白，同时物流公司名称被 20% 列宽提前截断；调整百分比时若业务列仍合计 100%，再叠加固定 40px 展开列，主表会变成 `100% + 40px` 并出现横向滚动。正确：列宽按主要信息优先级分配，名称字段获得更多宽度，操作列只保留复杂行可自然换行的合理比例；有固定列时业务百分比总和需预留固定列空间（本页收敛到 95%），不能靠隐藏滚动条；不删按钮、不改权限和事件。见 `src/views/logistics/LogisticsList.vue`。

- **vxe 展开最后一行时多行同时展开**：主表使用 `row-key="id"`，但未授权记录的业务 `id` 可能同时为空或为 0，vxe 会把相同 key 的记录当成同一行并联动展开。正确：接口数据进入表格前为每条记录生成稳定且唯一的前端 row key；若对象可能进入保存/API 参数，使用 `Object.defineProperty(..., { enumerable:false })` 避免临时字段改变 payload，再把 `row-key` 指向该字段。见 `src/views/logistics/LogisticsList.vue`。
- **唯一 row-key 修复后，行级异步状态也必须同步换 key**：主表改用 `__logisticsRowKey` 后，若 `loading/loaded/renderLimit` 映射仍以可能为空或重复的业务 `id` 索引，点击一行会让所有同 id 行一起显示 loading，并共享加载缓存和分批数量。正确：抽一个 `getLogisticsChannelStateKey(row)`，三组前端状态统一使用稳定唯一 row key；接口参数仍保留原 `id/logisticsId`，不要改变后端契约。见 `src/views/logistics/LogisticsList.vue`。
- **物流公司语义标签不能低对比小字**：`镖局` 属于识别物流公司类型的重要标签，12px 浅蓝字叠浅蓝底会在列表密集场景中难以辨认。正确：仅增强 `.logistics-company-tag`，使用 13px/600、22px 高、较深蓝字 `#1677d2`、浅蓝底 `#e6f2ff` 和 `#b3d8ff` 边框、4px 圆角；不要顺带改变“复制”标签或 `row.isBiaoju` 判断。见 `src/views/logistics/LogisticsList.vue`。
- **展开 Grid 小屏序号表头被拆字**：展开渠道表使用全百分比列时，序号列 4% 再扣左右 16px padding 后不足以容纳两个中文字符，`word-break:break-word` 会把“序号”拆成两行。正确：序号/渠道 ID 使用 52px/72px 固定基础列，其余列用 `minmax(0, fr)` 分配剩余空间；表头 `white-space:nowrap`，不要只禁换行却保留过窄百分比造成溢出。见 `src/views/logistics/LogisticsList.vue`。
- **物流授权省略文本必须可查看全文**：主表物流公司名称和展开渠道的名称、别名、仓库、编码、黑白名单使用 ellipsis 后，仅原生 `title` 提示样式不统一且黑白名单无提示。正确：用 `el-tooltip` 包裹实际 `.channel-cell-text/.logistics-company-name`，空值禁用，统一 popper class；黑白名单展示与 tooltip 复用 `getChannelCompanyListText(channel)`，避免两份条件表达式漂移；仅调整展示，不改字段或接口参数。见 `src/views/logistics/LogisticsList.vue`。
- **给 v-for 行加 Tooltip 时严禁误替换表头同名 class**：展开表头和数据行都含 `.channel-name/.channel-alias`，按第一个字符串区间批量替换会把 `channel.*` 放到 `v-for` 外，触发 `Property channel is not defined` 并中断整个展开 render，看起来像数据丢失。正确：先锁定 `v-for` marker 后的行区间再替换，表头保持 `$t()`；改后 grep 确认所有 `channel.*` 都位于行作用域，并跑 build。见 `src/views/logistics/LogisticsList.vue`。

- **CommonTable 大数据异步展开闪白、无限下滚或长期空白**：点击后预展开空内容、接口返回后再写明细会触发两次高度计算；通过无返回值包装方法后立即 `recalculate()` 还会与 vxe 内部渲染竞争；即使时序正确，一次 `v-for` 创建数百/上千条含多列权限组件的明细，仍会长时间阻塞主线程并撑大展开行。正确：首次展开先请求明细，通过 `CommonTable.getInstance()` 调 vxe 实例 `setRowExpand()`，等待 Promise 后再重算；大明细先渲染固定批次（如 50 条），由用户按批加载更多，筛选后重置批次；loading、已加载空数组、筛选无结果必须使用独立状态，不能都由 `list.length` 推断。见 `src/views/logistics/LogisticsList.vue`。
- **物流授权自然滚动主表组件选型**：`/logisticslist` 这类服务端分页、页面自然滚动、含异步展开和列设置的主表若使用 `CommonTable` 再手动拼 `height:null + scrollY:false`，虽可运行但与项目标准不一致。正确：使用 `GridTable` 的 `external-scroll=true + height=auto + scroll-y.enabled=false`，保留稳定 row-key、expand-config、vxe columns、toolbar、`getInstance()/recalculate()` 和全部按钮权限事件；迁移只替换等价表格壳，不改展开数据、API、分页或业务字段。见 `src/views/logistics/LogisticsList.vue`、`tests/logistics-list-grid-table.test.mjs`。
- **主表与展开明细双层表头吸顶**：展开明细使用自定义 CSS Grid 表头时，仅主 vxe 表头吸顶会导致长渠道列表滚动后失去列语义；父层 `overflow:hidden` 还会直接截断 fixed/sticky 表头。正确：继续监听 `.main-topmenu-content-wrapper`，主表头吸顶后把展开表头固定在主表头下沿；仅当明细 table 顶部已越过吸顶线且底部仍容得下表头时启用，插入等高 placeholder 防内容跳动，按明细 rect 同步 left/width，切换展开行、resize、离开明细底部及销毁时恢复并清理；承载层改 `overflow:visible`。主表表头下方不需要灰线时同时覆盖 header-wrapper 和 header column 的 `border-bottom`，避免 GridTable 公共 `!important` 边框残留。见 `src/views/logistics/LogisticsList.vue`。
- **物流授权渠道别名 placeholder 必须使用完整业务文案**：用户要求的是「渠道别名搜索」时，拼接 `$t('Extend4.渠道别名') + $t('搜索')` 或误用「别名搜索」都会造成文案不符合预期。正确：优先复用已有 key `$t('Extend4.渠道别名') + $t('搜索')`，只有 cn.js 中不存在可组合的既有 key 时才新增完整文案；调用 Hook 前注意老页面可能批量修复历史孤儿 key，需审计并回退无关语言包/引用变更。见 `src/views/logistics/LogisticsList.vue`。

- **布尔开关控制危险 API 分流时调用方必须显式传参**：页面某个分支漏传称重开关后，封装中的 `if(val)` 会把 `undefined` 当成关闭状态并落入直接发货/扣库存接口。正确：审计同页所有包装方法调用并显式传入 `this.ifShipTask`（或该分支计算出的开关），同时用 AST 回归测试约束每个调用至少两个参数，并验证真值走 `onlyUpdateOrderStatus`、假值走 `updateOrderAndManualShiped`；不要通过修改 API 默认值掩盖漏参。见 `src/views/purchase/FastPackageSignAndShip.vue`、`src/api/api_order.js`、`tests/fast-package-sign-weighing.test.mjs`。

- **批量人员选择弹窗不应固定窄列宽**：多语种员工姓名在固定 `100px` checkbox 中会相互挤压，人数多时无边界平铺也会显得凌乱。正确：短流程继续用 `el-dialog`，按桌面约 900px 宽、使用四列等宽无框选项网格；姓名保持单行省略并通过 tooltip 展示全文，避免按字符数猜测跨格导致英文姓名碎裂换行，同时保持 checkbox、行高和纵向列线稳定；checkbox 与姓名整体垂直居中，仅用轻微 hover/选中底色反馈，人员区设置最大高度并独立纵向滚动，底部操作固定在 dialog footer 右下角；800px 回落三列、520px 回落两列，只改展示壳并保留原选择数组、追加/删除分支、锁定提示和接口参数。见 `src/views/warehouse/UpdateWarehouse.vue`。

- **旧式表格去方框时不能直接删 border**：数据行边框参与表头/表体有效宽度补偿，直接改成 `border: none` 会让行宽增加并再次破坏列对齐。正确：保留同宽透明边框占位，去掉圆角、hover 边框和阴影，仅用浅背景表达悬停；不改列定义和业务内容。见 `src/views/purchase/UpperShelf.vue`。
- **PDA 图片快捷入口同时支持拍照与相册**：给移动端图片上传 input 添加 `capture` 会倾向强制直接打开摄像头，无法稳定让用户选择设备相册；只做图标入口但复用弹窗临时 `imageList` 又不会立即落到退货单；快捷上传若提交数据对象里的旧备注，刷新后会覆盖编辑框未保存内容，上传接口成功码但空 URL 还会把退货单错误推进。正确：快捷入口使用 `el-upload accept="image/*"` 且不设置 `capture`，让设备选择器提供拍照/相册；上传响应必须同时满足成功码与 URL 非空，保存时提交当前编辑框绑定值；原弹窗保存方法在请求全程持有 loading 锁，拒绝重复点击；上传成功后把 URL 追加到当前退货单已有图片列表，再调用原 `uploadDescription` 自动保存，并沿用原格式、大小和最多 5 张校验。整份图片列表覆盖的跨终端竞争不能靠前端锁解决，需后端原子追加或版本冲突校验。见 `src/views/pda/pda_return_order.vue`、`tests/pda-return-order-upload-safety.test.mjs`。
- **旧式 `tableulli` 表头与卡片行横向错位**：表头直接占满容器、表体有左右 padding，数据行再带 border，同时表头和表体单元格使用不同横向 padding/box-sizing 时，两边 `table-layout: fixed` 的有效宽度与逐列盒模型都不一致；未设宽的自适应列会吸收差值，固定列的差值还会逐列累积，导致日期列及后续列整体偏移。正确：保持表头与数据行完全相同的列定义和单元格 `box-sizing`/横向 padding，并让表头左右内边距等于“表体内边距 + 数据行边框”，使两侧参与列宽计算的内部宽度一致；只改局部 CSS，不改字段、条件和事件。见 `src/views/purchase/UpperShelf.vue`。
- **PDA 扫码值尾部不可见字符导致精确匹配失败**：扫码枪可能在 SKU、仓位、篮子号后附带全角空格、Tab/回车或零宽字符，普通 `trim()` 又不能覆盖全部零宽字符，页面随后按 SKU/仓位精确比较就会误报不存在。正确：在扫码处理方法入口统一清理首尾空白、控制后缀和零宽字符，只处理首尾，不删除 SKU 中间合法空格；用真实样例 `OSL-253-07149　` 做回归测试，并保持原 Enter、接口参数与焦点流程。见 `src/utils/normalizeScannerInput.js`、`src/views/pda/pda_shelf.vue`。
- **GridTable 表头全选后勾选图标变灰**：公共选择器给所有 `.vxe-checkbox--icon` 统一设灰色，覆盖了 checked/indeterminate 状态。正确做法：未选与 disabled 保持灰色，`.is--checked`/`.is--indeterminate` 在基础规则之后显式使用 VXETable primary 变量和 `#409eff` fallback。见 `src/components/Common/GridTable.vue`。
- **PDA 通知 API 不可凭包源码或框架记忆推断**：`src/main.js` 同时安装 Element UI 与 NutUI；即使包源码可能暴露 `warn`，部署运行时已证明 `this.$notify.warn(...)` 不可用，并在 `src/views/pda/pda_transfergoods.vue` 尺寸校验分支触发 `TypeError`。正确：新增或修改 PDA 警告/错误分支时，先核对同页/同模块调用并验证实际运行时 API，优先使用已确认稳定的 `this.$message({ showClose: true, message, type: 'warning' })` / `type: 'error'`；构建或 lint 通过不代表方法存在，必须运行测试精确覆盖改动分支或显式验证 callable。见 `src/main.js`、`src/views/pda/pda_transfergoods.vue`。
- **i18n Hook 将完整路径文案重复套 Extend**：JS 中误写硬编码 `'Extend2.获取虚拟SKU异常'` 时，若 Hook 把整串当中文 key，会生成 `Extend11.Extend2.获取虚拟SKU异常` 和同名脏 key。正确：硬编码字符串若本身精确命中 cn.js 完整路径，直接替换为 `this.$t('Extend2.获取虚拟SKU异常')`；已有完整 `$t()` 路径保持不动；并用隔离语言包 fixture 覆盖“已有路径、错误命名空间、重复 Extend”回归。见 `.cursor/hooks/check-vue-i18n.js`、`src/views/pda/pda_shelf.vue`。
- **抽屉复合字段不要依赖 Element 浮动栅格自动补位**：同一行字段高度不同会让后续 `el-col` 卡在半行后开始，纸张尺寸的单选与宽高输入若再加重复说明文字会割裂阅读，帮助图标也会因父级 `flex-wrap` 掉到下一行。正确：基础短字段按 3/2/1 列响应式收敛；运费模板统一顶部 label + 等宽控件；发货设置使用显式 CSS Grid 按 DOM 顺序排布；自定义尺寸紧跟“自定义”单选，仅展示宽、×、高、单位；高度压缩下拉和问号放在 `inline-flex; flex-wrap:nowrap` 的局部组内。见 `src/views/logistics/PredictionChannel.vue`。
- **授权步骤抽屉避免横贯式大步骤条**：在窄抽屉内直接使用默认 `el-steps` 并给各步骤内容固定高度，会出现连线横跨整栏、标题分散、内容区大片留白。正确：使用 `CommonDrawerForm`，步骤导航改为紧凑的数字圆点 + 短连线，当前/完成态统一主题蓝；每步内容放入同一轻量 `section-block`，用图标、标题、说明、操作按钮形成居中层级，授权码输入单独紧凑横排；保留原 `authFpxStep` 状态、API 方法和成功关闭流程。见 `src/views/logistics/LogisticsList.vue`。
- **CommonDrawerForm 只换外壳仍会错位重叠**：旧抽屉内部继续混用 `ul/li` 百分比列、`line-heiht30` 固定高度、inline 固定宽度时，在 70% 抽屉中会出现标签与控件左右漂移、纸张尺寸/重量/发票字段重叠，旧地址虚线表也无法获得标准表单观感。正确：以 `MyCustomerList.vue` 为参考，把普通字段、条件运费模板和发货设置完整迁到 `el-form + el-row/el-col + el-form-item`；复合尺寸/重量使用局部 flex，标签开关和地址信息使用独立响应式卡片，并逐项保留全部 `v-model`、事件、supplierId 条件和保存入参；交付前必须实际截图复验，不能只凭编译通过判定视觉达标。见 `src/views/logistics/PredictionChannel.vue`、`src/views/customer/MyCustomerList.vue`。
- **Agent 局部改 Vue 禁止整文件格式化**：局部替换不会自动触发编辑器“保存时格式化”，但对历史大文件执行 `prettier --write <file>` 会重排未修改代码并产生几千行无关 diff。正确：只手工或用可靠的 range formatter 格式化本轮新增/修改代码块；工具不能可靠限制范围时不得退化为整文件格式化。完成后用 SVN/Git diff 确认 hunk 仅落在本轮改动区域，再执行 `ReadLints` 和 `npm run build`。见 `src/views/logistics/LogisticsruleList.vue`、`PredictionChannel.vue`、`AddressConfig.vue`。
- **普通配置表单抽屉参考页选错**：地址配置等普通表单抽屉若只因页面使用 `CommonDrawerForm` 就机械照搬业务特殊的 `LogisticsList` 授权抽屉，会带入不适合普通字段录入的标签列和授权信息布局。原因是公共组件只提供抽屉壳，不代表不同业务抽屉应共享同一内容结构。正确做法：以 `src/views/customer/MyCustomerList.vue` 的成品样式为主要参考，使用 `label-position="top"`、`section-block/section-title`、15px gutter、响应式多列、详细地址整行和居中 footer，并逐项保留原字段、条件与事件；见 `src/views/logistics/AddressConfig.vue`、`src/views/customer/MyCustomerList.vue`。
- **超长条件型抽屉要改变主体交互而非只换折叠皮肤**：物流规则抽屉即使换成 `CommonDrawerForm`，若主体仍是数十个 `el-collapse-item`，视觉和查找效率都与旧版接近。正确：对齐 `MyCustomerList.vue` 的 `CommonDrawerForm + label-position=top + section-block/section-title + 固定居中 footer`，基本信息用响应式网格；条件区按业务拆成左侧分类导航 + 当前分类双列条件卡片 + 右侧 sticky 已选摘要，小屏导航横向滚动、卡片单列；迁移时逐项核对条件卡数量，保留全部 `v-if/v-show`、supplierId 特例、控件绑定、编辑/复制回填和保存参数。见 `src/views/logistics/LogisticsruleList.vue`、`src/views/customer/MyCustomerList.vue`。
- **旧授权抽屉结构升级**：原生 `el-drawer` 内用固定高度、`ul/li` 百分比和 inline 宽度堆授权字段时，标题、滚动区、底部按钮与窄屏排版容易失控；迁移后若标签列继续按 20%~22% 或固定 96px 分配，`refresh_Token/access_Token` 这类英文 token 会被拆行，手写 `<el-link type="danger">*</el-link>` 还会让必填星号脱离 label 基线。正确：使用 `CommonDrawerForm`，业务相关的物流公司与令牌归入同一基础信息 `section-block`，联系信息独立分区；授权 token 这类单列表单使用顶部 label + 下方控件的纵向结构，必填星号用普通 `span.token-required-marker` 随 label inline 对齐，输入/按钮/提示在下一行全宽自然换行；动态 token 的 `token.name` 过滤必须写在整行 `ul` 上，不能只给内部 `li` 写 `v-if`，否则不匹配 token 会留下大块空白行；footer 居中固定；必须逐项保留所有物流商 `v-show/v-if` 分支、token 绑定、协议、按钮事件和保存方法，并核对 section 标题 `$t()` 的真实完整路径，避免界面显示 key。见 `src/views/logistics/LogisticsList.vue`。
- **PDA 顶部语言沿用旧值与英文底栏裁切**：`router/index.js` 定义阶段的 `i18n.t()` 会把启动时语言固化进 `route.name`，登录页随后切换 locale 也不会更新子页面顶部；底栏按是否含非中文字符切到 18px 英文分支，再叠加固定 60px 高度，会把 `outbound/Inbound` 等文字裁掉。正确：保持原 `route.name` 语义不动，PDA 登录成功后用当前 hash 目标整页重载，使 Router 从已持久化的本次语言重新初始化；`App.vue` 对四个 PDA 根路径实时 `$t()`，底栏各语言统一 12px 单行省略，并按 path/homeid 判断选中态。见 `src/views/pda/pda_login.vue`、`src/App.vue`、`src/components/tabbar_index.vue`。
- **Packing 四路由模式现代化不可按单页改造**：`Packing.vue` 同一组件通过 `meta.pickingType` / `singleFlag` 服务四种路由模式，若只按当前入口重排 UI，容易遗漏其他模式的条件分支、supplier 门禁、扫码 refs/事件或商品行关键语义。正确：四模式共用中等密度扫码作业台壳，逐项保留全部条件/supplier gates/scanner refs/events；商品行可缩小图片并用响应式 flex，但必须保留临时 SKU、库存、已配/应配、异常/重新配货和备注；视觉参考 `PackageSign.vue`、`PreSign.vue`、`Resolve.vue`，同时以 `Packing.vue` 的远距离可读性作为语义护栏。见 `src/views/order/Packing.vue`、`src/views/purchase/PackageSign.vue`、`src/views/purchase/PreSign.vue`、`src/views/order/Resolve.vue`。
- **退货附件列文件类型图标与图片预览**：退货附件列曾使用固定占位图片；改为统一 tooltip 后又造成原图片 hover 大图预览回归，上传按钮另起一行或使用高饱和大色块也会破坏紧凑表格节奏。正确做法：复用 `src/assets/icons/file-types/index.js` 的 `getFileTypeIcon` 与 `normalizeFileType`，所有附件显示紧凑 SVG 图标；图片使用 `el-popover` + `el-image fit=contain` 展示原图，非图片使用 tooltip 展示文件名；附件图标与上传入口放在同一 `flex-wrap` 工具组并统一为 24px，上传入口默认轻量、仅 hover 高亮；工具组点击打开附件时，上传入口必须使用 `@click.stop` 防止冒泡；并保留 `openFileUrls`、上传权限及原事件等业务逻辑。见 `src/views/ordermanagement/ReturnOrder.vue`。
- **ReturnOrder 旧筛选区最小间距优化**：历史筛选控件混用 `ml10` 与 inline `margin-left` 时会拥挤且无规律换行，直接重写筛选 DOM 又容易丢字段、事件和权限条件。正确：只给现有筛选容器、基础行和状态行增加页面局部 class，用限定在 `.return-order-page` 下的 CSS `flex-wrap`、gap 和 `!important` 覆盖散落 margin；Element 内部高度用 scoped `::v-deep` 局部统一，不迁移组件、不改 `v-model`、事件、条件或查询逻辑。见 `src/views/ordermanagement/ReturnOrder.vue`。
- **卖家端复用仓库页的菜单入口**：卖家菜单由 `findCustomerLeftMenu` 返回，单独注册 `CustomerHome.vue` 子路由不会显示入口；需要按指定 `employee.supplierId` 前端灰度时，应在卖家菜单标准化完成后向对应 `homeid` 的 child 注入独立卖家 path，并去重，组件继续复用原仓库页，避免复用仓库 path 导致 `Home.vue` 壳和菜单高亮错误。见 `src/views/CustomerHome.vue`、`src/router/index.js` 的 `/myallorderlist`。
- **PDA 移动端调试工具误认**：移动端调试工具是 vConsole（不是 vnode） → 若在入口无条件引入并实例化，会让生产包加载依赖并显示调试面板。正确做法：项目从 `src/main.js` 接入，仅在 `process.env.NODE_ENV === 'development'` 条件中动态 `require('vconsole')` 并实例化；依赖使用 `package.json` 中的 `vconsole`。见 `src/main.js`。
- **PDA 订单状态详情页不能按旧截图继续堆叠信息**：订单状态、订单标识、物流信息、订单扩展字段和商品明细若都使用同一灰白底与弱分隔线，会造成信息层级混乱、标签与状态脱节；正确：保留原字段/条件/API/扫码回焦不变，按“查询输入 → 订单状态头 → 标识 → 物流/订单分区 → 商品 → 备注/时间/更多字段”建立单一白卡层级，状态只用浅色语义标签，390px 以下把头部标签回落到状态下方，340px 检查编号和商品信息可读性。见 `src/views/pda/pda_order_status_query.vue`。

- **PDA `$confirm` 小屏按钮被裁切**：Element UI 的 `this.$confirm()` 生成 `.el-message-box` 并挂到 `body`，页面 `<style scoped>` 无法覆盖它；固定 `width:80%` 或默认按钮 margin 在窄屏会让「确定」按钮贴边/超出视口。正确做法：在 `src/assets/css/common.css` 的 `body.pda-mode-body` 下限制 `width:calc(100vw - 24px)`、`max-width`，并让 `.el-message-box__btns` 使用可换行 flex；保留确认方法、文案和取消分支不变。见 `src/views/pda/pda_inventory.vue`。

- **PDA 任务盘点删除弹窗需统一**：`pda_inventory_task.vue` 删除入口若仍使用原生 `confirm()`，会与手工盘点页出现两套弹窗体验，且浏览器弹窗不受 PDA 小屏布局控制。正确做法：改用 `this.$confirm(...).then(...).catch(() => {})`，复用 `body.pda-mode-body .el-message-box` 的全局窄屏样式；保留 supplierId 显示条件、`deleteInventoryList` 入参和删除成功后的本地移除逻辑。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 按仓位盘点扫描后列表与历史不一致**：扫描仓位后，历史页同时展示两组数据：已扫描并计数的 `inventoryNewList`，以及尚未扫描的 `inventorylist`；扫描 SKU 时只是把命中的记录从旧集合移动到新集合，不是把另一组列表隐藏。若用 `v-if/v-else` 互斥渲染，会让扫描一个 SKU 后剩余库存行消失、列表数量与历史不一致。正确做法：保留两组连续 `v-for`，列表总数使用两组长度之和；按 SKU 仍只使用一个 `skuInventoryList` `v-for`，不同 `gridId` 的记录不能去重。见 `src/views/pda/pda_inventory.vue`。

- **PDA 页面 scoped 样式覆盖子组件失效**：将 `pda_inventory.vue` 的非 scoped `<style>` 改为 `<style scoped>` 后，直接写 NutUI/Element 子组件内部选择器会因 Vue2 scoped 属性隔离而不生效。正确：页面自身选择器继续保留 `.pda_inventory` 前缀；需要穿透覆盖 NutUI/Element 子组件的选择器改用 Vue2 `::v-deep`，既保持页面样式隔离，又确保第三方组件内部样式可覆盖。见 `src/views/pda/pda_inventory.vue`。

- **PDA 任务盘点页加 scoped 不能只改 style 标签**：`pda_inventory_task.vue` 的扫码框、左滑容器、备注下拉和数量输入样式需要覆盖 NutUI/Element 内部节点；直接改成 `<style scoped>` 会导致这些样式失效。正确做法：页面自身选择器保持 `.pda-inventory-task-page`，第三方内部选择器改为 `.pda-inventory-task-page ::v-deep ...`，业务模板、按钮和接口逻辑不动。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 按仓位盘点操作区层级不清**：第一行客户选择与扫码操作混排，当前盘点仓位、隐藏无库存商品、结束盘点和统计集中在同一行，导致小屏下操作节奏拥挤、主次不明。正确：按仓位盘点操作区分三行排列——第一行当前盘点仓位与隐藏无库存商品，第二行结束盘点，第三行统计；保留所有原有 `v-model`、事件和方法，不改变业务链路。见 `src/views/pda/pda_inventory.vue`。

- **PDA 按仓位盘点字段与操作区样式不一致**：按仓位盘点的 SKU/卖家 SKU 直接由 `el-link` 包裹整行，当前盘点仓位又独占整行 grid，导致字段左对齐方式、操作区节奏和输入尺寸与按 SKU 盘点不统一。正确：SKU/卖家 SKU 改为与按 SKU 相同的左对齐字段行；当前盘点仓位与操作区保持同一行，输入高度统一为 40px、宽度 120px；需要覆盖 NutUI/Element 子组件时保持普通 `<style>`（非 scoped），并用 `.pda_inventory` 前缀隔离选择器。见 `src/views/pda/pda_inventory.vue`。

- **PDA 手工盘点页白屏**：`/inventory_by_handle_p` 白页 → Vue 模板中的复杂包裹标签采用跨行紧凑写法，导致外层 `span` 被 `vue-template-compiler` 判定未闭合；同时 `mounted` 无条件调用 NutUI 组件的 `ref.focus()`，组件未暴露 `focus` 时会抛运行时异常。正确：复杂包裹标签使用明确的开闭结构；第三方组件 ref 聚焦前先判空并检查 `focus` 是否为函数，否则回退到 `$el.querySelector('input')`，且对 `$el`、查询结果及其 `focus` 全程判空。见 `src/views/pda/pda_inventory.vue`。

- **StockWarehouseList 商品抽屉字段大片空白/单位不对齐**：普通字段与不同高度的整行复合块混在 Element UI `el-col` float 流中，会被前面块的高度卡住留下空洞；将 `el-row` 改为 CSS Grid 后，Element UI 的 `::before`/`::after` clearfix 伪元素还会作为匿名 grid item 占用两个格子，继续造成每行首尾空白。正确：仅在 `stock-product-modern-drawer` 内给商品抽屉普通/混合字段使用 CSS Grid，先禁用 `.stock-product-responsive-grid::before/::after`，再让直接子 `el-col` 取消 float 并按 viewport 断点自动补位；虚拟 SKU、普通图片复合块、商品单位、商品属性、动态属性和多属性标题/列表等多项或复合块统一 `grid-column: 1 / -1`；两个条件图片字段若都套整行 class 会各占一行浪费空间，应使用专用两列 image grid，同时可见时各占半行，任一条件字段隐藏时让可见图片显式跨满，760px 以下改单列；上传事件和 `src` 保持不动；申报区必须使用 section 专用 class 与独立 4/2/1 网格，四个基础字段在常见桌面宽度（含通用 1440px 三列断点）仍保持四列，1100px 以下两列、760px 以下单列；扩展信息使用专用 4/2/1 网格，品牌/目录在大屏各跨两列、两列断点各占一列并排、小屏各自整行，不影响是否质检、质检信息、质检图片、商品单位等后续字段和其他 section；业务配置同样使用仅作用于该 row 的专用 4/2/1 网格，默认及 1440px 仍为四列，商品状态占两列、包材占两列，商品用途与商品材质各占一列、产品销售链接占两列，出库/入库收费类型各占两列；1100px 以下各普通字段占一列，状态仍占两列，包材与销售链接各占一列，两个收费类型各占一列；760px 以下所有字段各自整行，且不得影响其他 section；商品属性轻分隔并整行展示；高级设置需区分两类裁切：长 label 自身的文字换行裁切可仅在 registration 专用 grid 内设置 `height:auto`、`min-height`、正常换行和 `overflow:visible`；但整列每行首字被左右裁切并非 label 固定高度导致，而是高级设置两层嵌套 `.stock-product-responsive-grid`（外层 advanced-grid 与内层 registration-grid）同时继承通用 `margin-left:-4.5px` 且 `overflow:hidden`。只清内层 margin 或调整 registration-fields padding 只会把裁切从左侧挪到右侧，不能根治；Element UI `<el-row :gutter="15">` 会生成 inline 左右负 margin，普通 CSS `margin: 0` 无法覆盖 inline style，因此必须仅将这两层专用 grid 的左右 margin 以 `!important` 清零，并保持 registration-fields 左右 padding 为 0，不得改动其他网格用户指定的 `-4.5px`；特货属性桌面端不应换行，在四列网格中使用专用字段占三格、是否申报占一格，并让 radio 保持 `nowrap`，仅在窄屏断点换行；HWC 动态申报字段若每项整行会拉长抽屉，应让父容器使用两列响应式网格，每项按 label 在上、控件在下排列，760px 以下改单列，同时保留动态 input/select/checkbox 绑定与接口组装，且不得影响其他 section；单位管理链接放进 `el-form-item` 的 label 行；复选项按内容自然宽度 flex 流会失去统一列基准并产生错列，应让两组 `.unit-options-row` 保持原顺序、改用响应式 CSS Grid（大屏四列、1440px 三列、1100px 两列、760px 一列），同时让 checkbox 占满格子、input 不收缩、label 可在本列内换行；状态+序列号这类复合字段不能按普通单列，应占两列，内部也使用两等列，第二项与下一行第二列字段左边界对齐，不能用 space-between 制造大空档；状态文字保持 nowrap，一列断点回到整行；长宽高+单位复合尺寸字段不要无条件占两列；与商品重量、折叠高度相邻时，应把名称、图片等复合块保持在独立 row，再用四列 metrics row：大屏重量占 1 列、尺寸占 2 列、折叠高度占 1 列；两列断点用 CSS 排为重量/折叠高度同行、尺寸下一行整行；760px 以下按重量→折叠高度→尺寸单列排列，尺寸内部按固定百分比配合 `min-width: 0` 收缩；商品抽屉内不要直接给 Element 圈形 glyph 叠圆底，否则会形成双层白圈；应仅在 `stock-product-modern-drawer` 下覆写 `el-icon-info/el-icon-question` 的 `::before` 为普通 Arial `i/?`，info 使用蓝底、question 使用警告黄底，保持静态无 hover，且不影响列表、其它弹窗和全局图标。用户明确允许合并视觉 section 壳时可以移除重复的 `section-block/section-title`，但各业务 row、字段、条件、绑定和事件仍须保持独立且原样保留。表单纵向过松时只压普通短字段的 label/item 间距，纵向可压缩；横向间距须区分外层/板块/字段三级：drawer body 保持 10px 外层间距，section 左右内边距 20px，字段列 gap 保持 15px；复合图片、虚拟 SKU、商品单位、动态属性和表格块保留必要空间，固定 footer 对应的 drawer body 底部 56px 安全 padding 不动。成对的商品中文名/英文名使用专用两列网格，各占 50%；警戒库存这类“输入框 + 长 checkbox”复合字段在四列网格中占两列（半行），小屏回到整行，不能按普通 1/4 列挤压；用户明确取消扩展信息折叠时，应同时删除折叠入口、`v-show`、data 状态与 method，并让扩展信息和高级设置默认展示；模块标题与内容间距使用 15px，普通字段仍保持紧凑。见 `src/views/stock/StockWarehouseList.vue`。

> 本节由 Agent 在踩坑后自动维护；新条目写在最上方，保留「现象 → 原因 → 做法」。

- **PC 首页无登录缓存时白屏**：未登录直接访问 `#/index` 会进入 `Home.vue`，默认“测试员”状态随后把缺失或损坏的 `localStorage.loginInfo` 传给 `updataUserData`，读取 `loginPagePath` 时抛异常，接口拦截器也来不及跳登录页。正确：`Home.vue` 恢复缓存前安全解析并校验 `loginInfo.token`；有效缓存恢复后刷新并 `return`，无效缓存使用 `router.replace('/login')` 并立即 `return`，禁止继续首页初始化。见 `src/views/Home.vue`。

- **带命名空间的 `$t()` 不能默认视为正确**：`$t('ExtendN.字段')` / `$t('模块.字段')` 过去遇到点号直接跳过，错误命名空间会长期漏检；原因是 Hook 只按无点 key 查找。正确：按点分段精确校验完整路径；路径不存在但末级字段已有时修到 `findKeyPath` 的已有路径，末级字段也不存在时仅追加 cn.js 最后一个 Extend；比较上下文仅提示，并用隔离 fixture 运行回归测试。见 `.cursor/hooks/check-vue-i18n.js`。
- **动态扩展字段复用保持父页业务链路**：后端扩展字段列表抽组件时若把保存校验和 JSON 组装一起迁入，容易改变既有接口语义；正确：公共组件只统一 `input/select/date` 渲染、必填星号和提示 Popover，使用 `input` 事件回传值，父页原回填、校验、`elementName` 键和 `hwc_extend_info` 组装原样保留。见 `src/components/Common/DynamicExtendInfoForm.vue`、`src/views/task/StockWarehouseTask.vue`。
- **复杂抽屉列表禁止横向滚动**：旧 `ul/li` 多列表若列宽不足，不能靠扩大抽屉、`min-width`、横向滚动容器或隐藏滚动条掩盖问题。旧 `ul/li` 表格不要保留 inline 深灰 header/body，局部用完整 4px border 壳、浅表头、白数据行和细分隔线；正确：抽屉保持合理业务宽度，仓库/清关/供应商列表均使用 `display:flex; width:100%; min-width:0`，列比例收敛到 100%，单元格 `min-width:0` + 内容级省略，并对关键长字段保留 `el-tooltip`；表格内 `el-input`/`el-select` 用 `width:100%; max-width:100%` 随列收缩；抽屉普通短字段按 `xs/sm/md/lg/xl` 响应式栅格收敛，避免大屏控件过宽。供应商这种 9 列密集表在桌面抽屉 65% 宽度下，即使列百分比合计 100% 也会出现文字挤压；若只要求供应商密集表去两侧留白，应使用供应商 section 专用 class 清零左右 padding，并给 title 补 20px margin，不可改通用 table-section 影响仓库/清关；同时保留普通表单 4/3/2/1 列响应式断点，以及仓库/清关/供应商三表 `width: 100%`、无横向滚动的约束；供应商密集表表头省略时每个表头必须用 tooltip 展示完整文案；65% 仅适用于该商品抽屉，不泛化为其他抽屉标准。只迁布局，不改任何 `v-model`、条件、ref、事件、supplierId 特例和保存入参。见 `src/views/stock/StockWarehouseList.vue`。

- **A4 配货单版式避免头重脚轻与大片空白**：A4 配货单如果使用超大条码/标题、零散 inline 定位、地址 `&nbsp;` 拼接、footer 大 `margin-top` 和孤立签字区，会造成头重脚轻、信息不对齐和大片空白。原因是各信息块脱离统一网格与连续文档流，间距依赖硬编码占位，横竖版和真实数据长度变化时无法稳定对齐。正确做法：顶部条码、标题、单号紧凑成组；基础信息与收件信息采用两至三列布局，地址独占整行并用明确分隔符连接字段；表格、统计、签收区域保持边框连续；保留全部字段及 `v-if`、横版/竖版逻辑，并用真实横竖任务执行打印预览。见 `src/views/print/printAllocateTaskNew.vue`。
- **订单高级查询抽屉大屏字段长短不一**：`waitlogisticslist`、`waitchecklist`、`allorderlist`、`waitshiplist`、`shippinglist`、`historyorderlist`、`banorderlist` 等共用 `OrderSearchDrawer`，若在各页 slot 里散写 `xs/lg/xl` 与固定 `width:300px`，大屏会出现同一组筛选项长短不一。正确：优先在 `src/views/ordermanagement/components/OrderSearchDrawer.vue` 统一约束 `el-col/el-form-item/el-input/el-select/el-cascader/el-date-editor` 宽度、`min-width:0` 与复合时间控件 flex 收缩；但不要给 Element UI 的 `.el-input-group` / `.input-with-select` 强行改 `display:flex`，否则 prepend 下拉和输入框会重叠。未复用该组件的 `ReturnOrder.vue` 再局部套同类容器样式，不改查询字段和入参。
- **OrderSearchDrawer 基本信息跨页面列宽漂移**：公共组件内置字段与各页面 slot 混用 `lg/xl=8、6、12`，且抽屉按 viewport breakpoint 计算，导致同一行控件在大屏长短不一；BanOrderlist 的固定 `width:300px` 日期控件还会放大溢出。正确：普通字段和页面 slot 统一使用 `xs/sm/md=12、lg/xl=6`，日期复合控件内部固定类型选择器、日期区 `flex:1; min-width:0`；只改栅格/CSS，不改字段、条件、绑定和查询入参。见 `src/views/ordermanagement/components/OrderSearchDrawer.vue`、`AllOrderlist.vue`、`BanOrderlist.vue`。
- **订单扫码结构化二维码兼容**：PDA/PC 扫码值可能是含顶层 `id` 的标准 JSON，也可能出现窄范围的 `"id"."..."` 点号畸形格式，若原样提交会导致订单识别失败。正确：仅当共享扫码输入进入订单查询/API 路径时，在调用 API 的边界解析标准 JSON 顶层 `id`，并仅对该点号格式做正则兜底；SKU/序列号验货阶段和独立物流单号输入不做解析，普通订单号/物流单号保持原值不变；订单扫码输入保持即时 `v-model`，若由 Enter 触发查询不得使用 `v-model.lazy`，Vue 2 的 Enter 监听必须写为 `@keyup.enter.native`；`<el-input>` 根节点禁止使用 Element 内部类 `el-input__inner`，页面视觉类放根节点，内部输入样式用 scoped `::v-deep .el-input__inner` 覆盖，避免双层边框。见 `src/views/pda/pda_scanorder.vue`、`src/views/pda/pda_packaging_inspection.vue`、`src/views/pda/pda_order_weight.vue`、`src/views/order/PackagingInspection.vue`、`src/views/order/OrderWeight.vue`。
- **PDA 分页箱列表全选状态不同步**：创建中转出库单第二步若用独立 `checkAll` 布尔值，单箱勾选、移除或切页后会与当前页真实选择状态脱节；全选取消还可能误删其他页已选箱子。正确：从当前页可选箱子和 `newBoxList` 派生全选/半选状态；勾选只追加当前页可选箱子，取消只按规范化字符串 ID 移除当前页箱子，保留其他页选择；批量加入复用单箱的 SKU 贴标初始化。见 `src/views/pda/pda_cargo_transit_out.vue`。
- **PDA 创建出库单选项初始化误报公司异常**：公司/国家/仓库共用 `Promise.all`、响应码严格比较数字 `200`，且打开入口不等待初始化时，任一辅助接口失败或字符串 `"200"` 都可能清空公司列表并误报「查询公司列表异常」。正确：created 与打开入口复用单一 in-flight Promise，打开时等待；三类选项独立处理，所有新增创建接口响应码先 `Number(data.code)`，公司状态与仓库当前 ID 也按规范化值比较；仅公司真实失败或无启用公司时阻止打开。见 `src/views/pda/pda_cargo_transit_out.vue`。
- **PDA 公司下拉不回显**：`el-option` 使用 `v-show` 隐藏失效公司且接口/表单 ID 类型不一致时，Element UI 可能只显示裸值或触发不了预期刷新。正确：先用 computed 过滤 `status !== 3` 的选项，并把公司选项 ID、默认值、`@change` 值和回显比较统一为 Number（非数字保留字符串）；切换时清空旧箱子/地址模板后并行刷新两类数据。见 `src/views/pda/pda_cargo_transit_out.vue`。

- **Vue 模板事件方法名大小写必须核对真实 methods**：`Shippinglist.vue` 高级查询里把 `search.operId` 的 `@change` 写成 `findOrderList`，但页面真实方法是 `findOrderlist/findOrderlist1`，Vue 渲染时会反复报 `Property or method is not defined`。正确：改模板事件后立即 grep 事件名是否在 `methods` 中存在；同一页面已有查询入口优先复用原方法名和行为，如需要重置页码/保存搜索条件时用 `findOrderlist1`。见 `src/views/ordermanagement/Shippinglist.vue`。

- **公共高级查询抽屉迁移后不要再读父组件 `$refs`**：`Shippinglist.vue` 把目录筛选迁到 `OrderSearchDrawer.vue` 后，原 `this.$refs.searchCascader.getCheckedNodes()` 在父组件作用域取不到子组件内部 cascader，导致 `categoryId` 不会随查询提交。正确：公共抽屉字段通过 `.sync` / 事件回传业务值，父页面组装参数时从 `selectCheckCategory` 等数据字段取末级 id，并跳过 `ALL_SELECT_CATEGORY` 这类前端全选标记；不要用旧 DOM ref 读取子组件内部控件。见 `src/views/ordermanagement/Shippinglist.vue`、`src/views/ordermanagement/components/OrderSearchDrawer.vue`。

- **Vue2 改动不能只查单一语法点**：`Shippinglist.vue` 里 i18n hook 曾把 `params.descr="中文"` 误替换成 `params.:descr="$t(...)"`，但风险不止 `.:`，任何 template/script 表达式、slot、动态属性、对象赋值、引号括号都可能被批量替换破坏。正确：Vue2 文件改动后优先跑 `npm run build` 做编译级验证；沙箱系统错误要非沙箱重跑；再配合 `ReadLints`、组件/变量/方法核对、风险 grep（如 `\.:[A-Za-z_$]`、`Extend\d+\.Extend\d+`），全部通过后再交付。见 `src/views/ordermanagement/Shippinglist.vue`。

- **HistoryOrderlist 表头提前盖住筛选区**：使用通用 `useUniversalSticky` 只按表格 wrapper 到顶判断，会在筛选/操作区还没滚走时把 vxe 表头 fixed 到顶部，造成表头压到筛选区。正确：对齐 `CustomerList.vue`，监听 `.main-topmenu-content-wrapper`，用 `querySectionRef.getBoundingClientRect().bottom <= getStickyTopOffset()` 作为触发条件，只固定 vxe 表头；不要隐藏主滚动条，也不要让整块表格 wrap `position: sticky`。见 `src/views/ordermanagement/HistoryOrderlist.vue` 对标 `src/views/customer/CustomerList.vue`。

- **Element 下拉回显裸数字**：订单高级查询时间筛选新增 `keytype2/keytypes` 后，第一次打开抽屉显示 `3` 而不是「打印时间」。原因是 `el-option value` 用字符串，历史缓存/接口回填可能是数字，Element 匹配不到 label 会直接显示原值。正确：从缓存/模板读取下拉值时统一 `String(value)`，缺省值回到页面原默认值，不要置空。见 `src/views/ordermanagement/HistoryOrderlist.vue`、`WaitShiplist.vue`、`Shippinglist.vue`。
- **待发货/发货中两组时间筛选勿混用公共默认值**：`WaitShiplist` 与 `Shippinglist` 都需要 `keytype2+operTime`、`keytypes+operTime1` 两组独立筛选，但业务选项不同；若一组由页面 slot 渲染、另一组走公共默认配置，会出现数量和类型不一致。正确：两组统一由 `OrderSearchDrawer` 渲染，页面通过 `searchConfig.timeFilterOptions` 明确传入选项；待发货仅付款/创建/审核/交运，发货中仅付款/创建/审核/通知发货/交运/打印，并保持后端编号不变。见 `src/views/ordermanagement/WaitShiplist.vue`、`Shippinglist.vue`。
- **时间范围筛选应占两列**：订单高级查询普通字段采用四列布局时，时间范围由选择器和日期区间组成，若仍占单列会被压缩且与视觉层级不符；正确：公共 `OrderSearchDrawer` 的 `keytype2/keytypes` 以及页面自定义时间 slot 使用 `xs/sm/md=24、lg/xl=12`，内部类型选择器固定宽度、日期区间 `flex:1; min-width:0`。见 `src/views/ordermanagement/components/OrderSearchDrawer.vue`、`HistoryOrderlist.vue`。
- **组合搜索框也应占两列**：订单号、SKU、备注模糊搜索等“类型下拉 + 输入框”复合控件若按普通单字段占一列，输入区会被压缩且与时间范围筛选宽度不一致；正确：公共 `OrderSearchDrawer` 的订单号/SKU/keyword 以及页面 slot 中同类复合控件统一使用 `xs/sm/md=24、lg/xl=12`，单个客户/平台/店铺下拉仍使用 `12/12/12/6/6`。见 `src/views/ordermanagement/components/OrderSearchDrawer.vue`、`HistoryOrderlist.vue`。
- **OrderSearchDrawer 使用页的自定义组合框也要统一**：公共组件内置组合框改宽后，页面 slot 中的“下拉 + 输入/数字框”（如订单号、SKU、模糊搜索、标签打印次数）若仍保留 `12/12/12/6/6`，会继续出现同页宽度不一致；正确：逐页检查所有 `OrderSearchDrawer` slot，自定义组合控件统一改为 `xs/sm/md=24、lg/xl=12`，单独下拉或单独输入保持普通列宽。见 `HistoryOrderlist.vue`、`AllOrderlist.vue`、`BanOrderlist.vue`、`Shippinglist.vue`。
- **基本信息三列布局需覆盖公共字段与 slot**：若只改公共组件、遗漏 `basic-info-*` 或 `shopIdsList` slot，复用页面仍会出现两列/四列混排；正确：仅在 `OrderSearchDrawer` 的 `block_basic` 及其基本信息 slot 统一 `xs/sm/md/lg/xl=8`，物流、详细参数、属性信息保持原布局，整行编辑块按业务需要保留 `span=24`。见 `OrderSearchDrawer.vue`、`HistoryOrderlist.vue`、`AllOrderlist.vue`、`BanOrderlist.vue`、`Shippinglist.vue`、`WaitShiplist.vue`、`WaitLogisticslist.vue`。

- **MyCustomerList 列表布局验收要求**：`MyCustomerList.vue` 最终效果重点不是仓库列 tooltip，而是列表布局必须对齐 `CustomerList.vue`：页面自然滚动，滚过筛选区后 vxe 表头吸顶；分页器使用表格 wrap 内 sticky 分页，位置和高度正常；主表必须一屏展示所有列，不能靠隐藏横向滚动条掩盖超宽；最后一行必须停在分页器上方，不能被分页器遮挡；禁止为修某列强行改主表固定百分比 `width`、表格 `min-height`、隐藏主滚动条或释放 `.main-topmenu-content-wrapper`，避免右侧留白、双滚动条、分页器错位。见 `src/views/customer/MyCustomerList.vue` 对标 `src/views/customer/CustomerList.vue`。

- **筛选型展开 Tab 无数据不要置灰**：`StockWarehouseTask.vue` 展开区新增“预签收篮子”时，若按筛选结果为空直接禁用 Tab，用户会觉得功能不可点、位置突兀。正确：Tab 保持可点击，列表层按现有数据前端筛选；无匹配数据时展示 `暂无数据` 空状态，不改接口参数和源数据。见 `src/views/task/StockWarehouseTask.vue`。

- **自然滚动页勿直接释放主滚动容器**：`MyCustomerList.vue` 从其他菜单跳入加载时出现表格和分页只占顶部、下方大片空白。原因是页面把 `body.warehouse-update-page-active .main-topmenu-content-wrapper` 改成 `height:auto; overflow:visible`，破坏了 `Home.vue` loading/content 外层的首屏高度。正确：对齐 `CustomerList.vue`，只释放 `.common-page-container` 与 `.content-loading-wrap`，表格容器需要首屏占位时用页面内 `min-height: calc(100vh - xxpx)`，不要直接改 `.main-topmenu-content-wrapper`。见 `src/views/customer/MyCustomerList.vue`。

- **CommonTable/vxe 展开行被裁切先对齐参考页结构**：`MyCustomerList.vue` 展开行问题反复后，单纯给 vxe/body/sub-table 叠 `width/max-width/overflow/table-layout` 会让主行被挤压、展开内容被遮挡或横向滚动假隐藏。正确：先按 `CustomerList.vue` 对齐外层 `history-order-table-wrap custom-expand-table`、`type="expand"` 紧跟展开图标列、展开内容仅在 `row.ischecked` 时渲染、展开/收起后调用 `recalculate()`；子表样式保持参考页的轻约束，不额外加 `width/max-width:100%` 压 vxe 计算。见 `src/views/customer/MyCustomerList.vue`。

- **PC 扫码作业台现代化要对齐已重构工作台风格**：`Purchasinglabeling.vue` 这类入库扫码页面若照搬外部“高端视觉”做大留白、灰底信息块、卡片套卡片，会显得和 `PreSign.vue` 等已重构页面冲突且高度变长。正确：以 `PreSign.vue` 为首选参考，保留灰底 `#f2f5f8` + 白卡 + 10px 间距 + 标题蓝色指示条；信息展示用细分隔行/表格节奏，少用大面积灰色禁用态块；扫码与本次录入动作保持首屏工作流优先。见 `src/views/purchase/Purchasinglabeling.vue`。
- **波次规则自动生成时间限制要前后同步**：`pickRule.vue` 添加/编辑波次规则时，抽屉内 `el-time-picker.selectableRange` 放开全天选择后，保存校验也必须从“上午 00-12 / 下午 13-23”同步改为通用 `HH:mm`；时间大小比较不能用 `Number('08:00')`（结果是 `NaN`），应去掉冒号按 `HHmm` 数值比较。见 `src/views/order/pickRule.vue`。
- **服务端分页页勿混用 `footer-float-fixed` 与 sticky 分页**：`WarehouseList` 现代化后若分页同时加 `footer-float-fixed`（全局 fixed 底栏）和局部 `position: sticky`，分页会悬浮错位、压住表格；`GridTable` 使用 `external-scroll` 时，若直接 `.vxe-table-wrapper` 用 `flex: 1` 吸收剩余高度，表体内容还会溢出 wrapper，而分页器按 wrapper 盒子提前排版，出现在数据行中间。正确：页面根和 table-wrap 用纵向 flex 撑满，直接 vxe wrapper 保持自然高度 `flex: none; display: block`，分页 footer 用 `margin-top: auto` + wrap 内 sticky 推到底部；服务端分页列表不要叠 `footer-float-fixed`。见 `src/views/warehouse/WarehouseList.vue`、`src/views/customer/MyCustomerList.vue`。
- **GridTable 迁移勿覆盖历史业务颜色**：旧 `el-table` 迁到 `GridTable/vxe-column` 后，若在页面级 `.xxx-table .vxe-body--column { color: ... }` 或 `.xxx-table .el-link { color: ... }` 统一设色，会吞掉模板里按状态写的历史颜色（如作废灰、配货中蓝、完成绿、过期红、批次号紫色）。正确：表体只调 padding、border、font-size、hover 背景等结构样式，不写全局 `color`；操作列颜色用 `.xxx-action-cell` 局部约束。见 `src/views/order/BatchManagement.vue`。
- **UpdateWarehouse 主保存按钮不明显**：仓库信息保存入口若只放首屏卡片头部，用户滚到货架列表后容易找不到保存或误以为只剩货架操作。做法：保留原权限条件，只保留一个页面内 `position: sticky` 的局部操作条，复用原 `returnWarehouselist` / `updateWarehouse` 方法；不要移动或改写保存入参。见 `src/views/warehouse/UpdateWarehouse.vue`。
- **MultiSearchInput 批量搜索限制按行数而不是字符数**：弹层原 `panel-maxlength` 只限制字符数，无法表达“最多 500 行”，且页面提交前可能仍按旧 `slice(0, 1000)` 处理。正确：公共组件只提供可选 `panelMaxLines` 行数截断能力，不改默认提示和默认行为；需要限制的页面单独传 `:panel-max-lines="500"` 与页面专用 `tooltip-text`，并在页面侧提交格式化同步 `slice(0, 500)` 兜底。见 `src/components/Common/MultiSearchInput.vue`、`src/views/ordermanagement/WaitChecklist.vue`。
- **vxe 自定义列面板显示 `...`**：从 `el-table-column` 迁到 `vxe-column` 时，只保留 `#header` 插槽但不写 `field/title`，列设置面板会拿不到可读列名，只显示省略号。正确：每个业务列都补稳定 `field` 和可读 `title`，复合表头用 `/` 拼接完整语义；勾选/操作等固定列用 `col_checkbox/col_action`。见 `src/views/order/BatchManagement.vue`。
- **GridTable 列设置弹层不要负定位进表格边界**：把 `common-table-toolbar` 用 `position:absolute; top:-xxpx` 拉到表格上方，容易被父级 `overflow:hidden` 或页面右侧边界裁切，列设置弹层会像卡在表格里。正确：列设置工具条放在表格 wrapper 内正常文档流右上角，父级允许 `overflow: visible`，右侧统计 chip 与表格保留安全间距。见 `src/views/order/BatchManagement.vue`。
- **Vue i18n hook 修复后要查对象属性赋值**：`check-vue-i18n.js` 可能把 `param.descr="中文"` 这类对象属性赋值误替换成 `param.:descr="$t(...)"`，导致语法错误。正确：hook 后立刻 grep `\.:[A-Za-z_$]`，手动改成 `param.descr = this.$t('ExtendN.xxx')`；同时再跑 i18n hook 和 lint。见 `src/views/order/BatchManagement.vue`。
- **波次批量打印报关单/发票字段勿混用旧队列参数**：`BatchManagement.vue` 的“批量打印”走公共 `WaveBatchPrintTemplate.vue` 调 `/mergeLabelPdf`，后端 `MergeLabelDto` 不接收顶层 `ifPrintCustoms/ifPrintDanfe`，这两个只属于旧 `printJson` 队列链路。正确：弹窗 checkbox 只控制前端是否在每条 `labelInfos` 里传 `customPdf`、`invoicePdf`；订单数据来自 `/findOrderAddressPdf` 的 `customPdf` 与 `invoice_info`/`invoicePdf`。见 `src/components/WaveBatchPrintTemplate.vue`、`wms-api` 的 `MergeLabelDto.java`。
- **服务端分页表格搜索要清空勾选状态**：`TransferGoods` 这类主表勾选若用 `checkTaskId/checkItem` 跨页保存，搜索后接口返回新列表仍会按旧 ID 回填 `row.checkFlag`，用户会看到旧勾选残留。做法：搜索/筛选入口发请求前调用页面已有的清空选择方法（如 `clearCheckSelection()`），同步清空表头全选、半选、行 `checkFlag` 和已选择数量；不要只在接口成功后改当前页行。见 `src/views/warehouse/TransferGoods.vue`。
- **PDA 签收页要按作业流整体统一**：预签收页若只局部美化商品卡、已签收卡或扫码区，最后仍会像几个控件拼起来，用户会继续反馈“还能改好看么”。原因是签收页同时承载扫描、辅助仓位/批次、商品识别、历史记录，视觉语言不统一时会割裂。做法：顶部做“主扫码 + 辅助工具区”，预签收商品做商品识别卡，已签收做历史票据卡，列表外层不要再套大白卡；小 PDA 必须补 `360px/340px` 断点，340px 以下辅助工具改单列、商品次级 chip 改上下排列、历史记录隐藏冗余 label；改后 grep 核对 `ref/v-model/@keyup.enter/@click` 和字段展示。见 `src/views/pda/pda_pre_sign.vue`。
- **PDA 历史/已签收卡片避免浪费空间**：预签收“已签收”列表若把卖家、时间、仓位、快递单号全部做成 label 上下堆叠，再外套白卡，会导致 375px PDA 首屏只能看到一两条，用户会反馈“浪费空间/丑”。原因是历史记录的首要目标是快速扫读批次、状态和关键字段，不是逐项填写表单。做法：去掉卡片套卡片，单条记录用紧凑摘要卡；批次和状态同一头部，短字段可两列，长时间/单号独占一行，摘要条轻量化；保持 `stockwarehouseList` 字段、空状态、查询入口和业务绑定不变。见 `src/views/pda/pda_pre_sign.vue`。
- **PDA 作业页不要像 PC 表单堆叠**：预签收这类扫码作业页若把“扫描/临时仓位/批次/商品/备注”都做成大标签 + 大输入框 + 卡片套卡片，会显得笨重、割裂，用户会直观反馈“丑”。原因是 PDA 视口窄，作业核心是扫码和快速识别商品，PC 表单式边框、重焦点蓝框、过大图片和大备注框会抢走注意力。做法：主扫码输入突出但少标题，辅助字段用轻分隔和紧凑输入；商品卡片用左图右核心信息，图片控制在 68~76px，SKU/采购量做明确层级，备注区低干扰，不要大面积表单框；重构前后必须 grep 核对 `ref/v-model/@keyup.enter/@click`。见 `src/views/pda/pda_pre_sign.vue`。
- **PDA Element Message 超出屏幕**：PDA 页面里 `this.$message()` / `Message` 提示可能右侧溢出屏幕，尤其 357px 左右设备上看起来只显示半截。原因是 Element UI `.el-message` 默认 `min-width: 380px`，且 Message 挂在 `body` 下，页面 scoped 样式压不到。做法：在全局 `common.css` 使用 `body.pda-mode-body .el-message` 覆盖 `min-width:0`、宽度 `calc(100vw - 20px)`，内容允许换行并保持图标/关闭按钮不收缩；不要在单个 PDA 页面里写 scoped 修复。见 `src/assets/css/common.css`、`src/router/index.js` 的 `pda-mode-body`。
- **PDA 进子页面顶部偶发被遮住**：从 PDA 菜单页滚动后点击菜单进入子页面，页面有时会继承旧滚动位置，导致顶部内容像是被往上滑掉。原因是 `pda_index.vue` 本身是 PDA 父级滚动容器（`.pda_index { overflow-y:auto; }`），路由切换时父容器 `scrollTop` 不会自动清零。做法：在 PDA 父级 `$route` watcher 内统一调用滚动复位，必要时 `$nextTick` 后再复位一次；不要在每个 PDA 子页面单独补 margin/padding。见 `src/views/pda/pda_index.vue`。
- **PDA 扫描区小屏挤压**：扫描、临时仓位、批次等多输入若继续按旧表单横向塞进一行，357px PDA 宽度会出现标签换行、按钮挤压、默认红色 checkbox 跳色；顶部 tab 若做成白色卡片，会和扫描卡形成“卡片套卡片”，显得笨重。做法：扫描区改成白色操作卡内的纵向分组，主输入占满宽度并使用轻量输入壳；批次输入和搜索按钮只在字段块内横排；NutUI checkbox 在页面内局部改为蓝色主题并避免文字截断；PDA 顶部 tab 用白底文字 + 下划线，不要另套卡片阴影；重排前后必须 grep 核对 `ref/v-model/@keyup.enter/@click`。见 `src/views/pda/pda_pre_sign.vue`。
- **MultiSearchInput 替换旧双击批量弹窗**：旧页 `@dblclick.native` 打开自定义 textarea 时，页面方法可能带有行数限制、特殊类型拦截、cookie 保存或指定查询入口；原因是公共组件默认模式会直接把多行内容格式化写回并触发 `search`，可能绕过页面原业务校验。做法：有页面特定逻辑时用 `batch-mode` + `@batch-search` 在页面内转换换行/截断/校验后再调用原查询方法；弹窗字数限制要用面板专用 props（如 `panel-maxlength` / `panel-show-word-limit`），不要把 `maxlength/show-word-limit` 透传到主输入框导致页面输入框也显示 `0/500`；仅普通输入可直接用 `@search`。见 `WaitChecklist.vue` 顶部订单号/SKU 批量搜索。
- **StockWarehouseList 打印 SKU 标签接口失败无提示**：弹窗内点击打印后接口返回非 200，页面看不到 message；原因是 `printSkuLabelNew()` 的失败 `else` 为空，`catch` 只写 `errormsg` 未展示。做法：接口失败分支必须 `this.$message.error(data.msg || this.$t('Extend10.打印异常'))`，请求异常也要弹出可见提示；不要只改全局 z-index。见 `StockWarehouseList.vue`。
- **表格筛选页分页与滚动截止**：`accountbill` 这类筛选 + 表格 + 分页页面若只给根容器或表格 wrapper 加 padding，容易出现两种坏状态：最后一行被 fixed 分页遮挡，或留白过大且滚动条穿到分页器后面。做法：优先让分页器参与布局或 sticky 在表格 wrap 内；必须 fixed 时，用分页器 `offsetHeight` 动态收缩 `.main-topmenu-content-wrapper` 或设置等效底部占位，使滚动条轨道、表格内容和最后一行都停在分页器上方；销毁页面时清掉写入滚动容器的 CSS 变量/内联样式。见 `AccountBill.vue`、`material.vue`。
- **表格筛选页统一标准**：`accountbill` 这类筛选 + 表格 + 分页页面若机械套 10px 灰底卡片，会出现横向滚动、分页高度漂移、筛选区与表格之间露灰、列宽浪费、操作列被挤窄等反复视觉问题。原因是通用“现代化卡片”规则不适合历史业务表格筛选页。做法：以 `HistoryOrderlist.vue` 为视觉基准，页面/筛选/表格白底一体化，去灰底、去外层 padding/margin、去卡片阴影圆角；主表用 `GridTable`/`CommonTable`，列宽百分比总和收敛到 100%，表头不换行，单元格内容省略并 hover 展示全文；自然滚动 + 表头吸顶；分页器保持历史高度，fixed/sticky 时必须让滚动条和表格末行停在分页器上方。见 `AccountBill.vue`、`HistoryOrderlist.vue`。
- **CargoTransitOut 抽屉 SKU 列排版**：出库任务管理抽屉的箱子信息表，SKU、贴标 checkbox、费用提示在同一列时最容易反复出问题。验收标准：14 寸 100% 缩放和小屏都不能横向滚动到看不见操作列；表头（SKU种类/SKU总数/操作）不换行；短 SKU（如 `PSS-002`）必须一行完整展示，长 SKU 才省略并 hover 完整展示；大屏不能让短 SKU 撑出大片中间空白导致费用文案截断；checkbox 勾选前后位置不能跳；费用/提示文案省略时 hover 到「checkbox + 贴标 + 文案」整组都要展示完整内容；抽屉底部固定按钮不能遮住箱子表最后一行。做法：箱子表用 `table-layout: fixed`，表头强制单行省略，非 SKU 列保持“不换行所需”的固定宽度，SKU 列吃剩余空间；SKU 行横向纯文本展示，不加背景块/胶囊/卡片；SKU 文本区设合理 `min-width` 保短码完整、长码省略，不能用单一百分比把小屏短码压成省略；贴标/费用区吃剩余宽度，checkbox 固定宽度，费用文案 `text-overflow: ellipsis`；用 `el-tooltip` 包裹 SKU 和整组贴标费用，不只用原生 `title`；滚动内容底部要给固定 footer 留安全距离。只改 class/CSS/展示 tooltip，不改 `detail.ifLabels`、`changeBoxSkuIfLabel(detail)` 和保存入参。见 `CargoTransitOut.vue`。
- **CargoTransitOut 创建抽屉贴标操作复用**：创建中转出库任务抽屉还未保存，SKU 贴标开关不能调用已存在明细的 `modifyIfLabels` 接口。做法同 `CargoTransitTask.vue` 新建：在 SKU 文本后放 `el-switch`，只更新本地 `detail.if_label/ifLabels`，随 `saveOrUpdateWebOutTask` 创建参数一起提交。见 `CargoTransitOut.vue`、`CargoTransitTask.vue`。
- **GridTable 自然滚动页表头误吸顶/刷新后不吸顶**：`TransferGoods` / `MyAllocateTask` 这类 `external-scroll=true` 页面，表头 fixed 只监听 `.main-topmenu-content-wrapper` 或只按表格顶部判断，可能出现筛选区还在页面内但表头已吸顶，刷新/keep-alive 后监听状态也可能丢失。做法：初始化先 `destroyStickyTableHeader()` 清旧 fixed/监听；同时监听主滚动容器和 `window` 捕获滚动；给筛选/操作区加 `querySectionRef`，`updateStickyTableHeader()` 必须同时满足筛选操作区已滚走、表格到顶、表格底部仍有内容才 fixed。见 `TransferGoods.vue`、`MyAllocateTask.vue`。
- **TransferGoods 操作列过窄**：主表 `ops` 列原 `width=68`，遇到「打回待审批 / 创建入库单」等较长中文操作会显得拥挤。做法：仅放宽 `field="ops"` 操作列（如 104px），不改 `external-scroll`、不挪动打印/发货等业务列，避免破坏表头吸顶和列设置。见 `TransferGoods.vue`。
- **StorageFee 新增规则行看不出**：新增规则插在列表中但没有视觉提示；该页保存会提交整个 `storageFeeList`。做法：用不可枚举 `__isNewStorageFee` 标记做浅黄高亮，避免临时字段进入接口参数；保存后 `findStorageFeeList()` 刷新恢复普通样式。见 `StorageFee.vue`。
- **OutOfWarehouseFee 新增收费行看不见**：数据多时新增行追加到底部，用户误以为没新增。做法：未保存行 `unshift` 到第一行，并用 `!id` 做前端高亮；不要新增会提交到后端的临时字段，保存后沿用 `findOutWarehouseFee1()` 刷新。见 `OutOfWarehouseFee.vue`。
- **StockWarehouseTask 主表列宽/裁切**：主表固定像素列宽（如 260/230/130/100）与百分比列混用，小屏会撑出横向滚动；后置 CSS 固定 `--stock-task-row-height` 并 `overflow:hidden` 会裁掉最后一行字段；入库编号行文本过长会挤掉剪刀后面的单据图标。做法：主表列宽统一百分比，移除行高固定裁切，改 `min-height + height:auto + overflow:visible`；文本与多个图标拆成可收缩文本区 + 固定图标区；抽屉内普通 `el-table` 不属于本次主表范围。见 `StockWarehouseTask.vue`。
- **CargoTransitTask 批量弹窗 Message 被遮住**：批量生成出库单先展示剔除原因 Message、随后打开 `BatchOutDialog`，Dialog 后获得更高 z-index，且仅 `$nextTick` 仍可能不够 → Message 仍在下层。做法：在全局入口 `src/assets/css/common.css` 给 `.el-message` 统一加 `z-index: 10000 !important`，不要依赖页面专用 `customClass`；若仍有可用任务先 `batchOutDrawer=true` 再提示。见 `CargoTransitTask.vue`。
- **库存三商品页左侧快速搜 SKU**：入口放 `Home.vue` 左侧菜单时，必须用当前路由白名单控制展示，并由解析接口返回 `pop/totalNum` 映射到 `/stockWarehouselist`、`/promoteStockWarehouselist`、`/comboSku`；目标页还要读取 `query.keyword/keytype/autoSearch` 触发现有搜索，否则只跳页不筛选。见 `Home.vue`、`StockWarehouseList.vue`、`promoteStockWarehouselist.vue`、`comboSku.vue`。
- **OutOfWarehouseFee SKU 自定义收费表**：去掉 hover；表头用 `scroll-float-fixed` + CSS sticky（吸在 `StockClassList` 外层 Tab 内容区顶部，即「出库类收费」下方），勿用 JS `position:fixed`（滚动容器是 `.el-tabs__content` 非 window）。见 `OutOfWarehouseFee.vue`、`StockClassList.vue`。

- **重构报错即回滚**：大规模改 Vue 模板后若 build 失败、div 标签不平衡、或用户反馈白屏/报错，**立即从 `.svn/pristine` 或 git 恢复该文件**，仅做「根 class + Tabs class + CSS」安全改造，禁止未验证前批量改 DOM/脚本。见 `OutOfWarehouseFee.vue` 2026-06-18 教训。

### 2026-07-02

- **CustomerHome 切换时区勿用接口返回覆盖登录态**：客户工作台退出仍跳 `/login`；`changeSellerTimeZone` 可能只返回部分用户信息，不能直接 `updataUserData(data.data, this)`，否则 token/loginPagePath 等登录态可能被覆盖，导致退出或重新登录链路异常。做法：接口成功后用当前 `employee` 合并 `timeZone`，写 Vuex/sessionStorage/localStorage，再刷新。见 `CustomerHome.vue`。
- **CustomerHome 切换时区需走客户专用接口**：客户首页不能只本地改 `employee.timeZone`/Vuex，否则后端时区不落库、刷新后丢失；也不要直接复用卖家首页的 `changeTimeZone` action。做法：`api_login.js` 增加 `changeSellerTimeZone`，`CustomerHome.vue` 保存时提交 `{ timeZone }`，成功后只合并当前 `employee.timeZone` 并写 Vuex/sessionStorage/localStorage，勿用接口返回覆盖完整登录态；卖家首页仍用 `Home.vue` 的 `changeTimeZone`。见 `CustomerHome.vue`。

### 2026-07-08

- **CommonTable 展开列放末尾导致轻微横向滚动**：`CommonTable.vue` 全局把 `.col_expand` 强制 `40px`；若 `type="expand"` 列放在百分比列**之后**，表格总宽 = 100% + 40px → 页面可左右滑一点。做法：与 `CustomerList.vue` 一致，展开列紧跟展开图标列（固定 `width="40"`）之后；并隐藏 `vxe-table--scroll-x-*` 虚拟滚动条。见 `MyCustomerList.vue`。
- **纯布局改列宽勿跑 i18n hook**：旧页只调 `min-width`/overflow 时，勿对整文件跑 `check-vue-i18n.js`，会批量改 `$t()` 路径并污染 `cn.js`。见 `TransferGoods.vue`、`MyCustomerList.vue`。
- **CommonDrawerForm 底部按钮遮挡内容**：组件有 footer 时，最后一块表单可能滚不到 footer 上方。正确：公共组件内给存在 `$slots.footer` 的 `.drawer-body` 加底部安全 padding，并保持 body `flex:1; overflow-y:auto`、footer `flex-shrink:0`；页面只保留宽度等个性化样式。见 `CommonDrawerForm.vue`、`WarehouseList.vue` 新增仓库抽屉。

### 2026-06-18

- **OutOfWarehouseFee 第三轮样式（CSS 驱动）**：根节点 `out-of-warehouse-fee-page`；内层 tabs 去 `border-card` 改 `fee-inner-tabs`；各 Tab 包 `fee-table-block` + 表头 `scroll-float-fixed`；提示改 `fee-tip-banner`；按 SKU Tab 数据行去重复「基础费/附加费」标签；**不**整页改 GridTable。样式用普通 `<style>` 前缀 `.out-of-warehouse-fee-page`。见 `OutOfWarehouseFee.vue`。

### 2026-06-17

- **OutOfWarehouseFee 按SKU Tab 单元格降噪**：表头已有列名时数据行去掉重复的「基础费:/附加费:」；`fee-input-line` 横排防「/包裹」换行；`+` 移到 tab 工具栏；保存改 `primary plain`。见 `OutOfWarehouseFee.vue` tab2_0。
- **OutOfWarehouseFee 第二轮视觉**：内层 `el-tabs` 去掉 `border-card` + 去掉 `fee-card` 双层白盒；`fee-tab-toolbar` 红框空 `li` 改为 `fee-tab-bar` / `fee-tip-banner`；表头统一 `ul-th`、斑马纹改 CSS `nth-child`；`tab2_5` 补 `fee-table-block`。样式用普通 `<style>`。
- **GridTable 表头吸顶标准（用户确认）**：服务端分页列表要 **页面自然滚动**（非表内滚动），滚过筛选区后 **vxe 表头吸在顶栏下**。做法：`external-scroll=true` + `initStickyTableHeader` + `.history-order-table-wrap` + body 类 + fixed 分页；**禁止** `external-scroll=false` / `useUniversalSticky`。参考 `TransferGoods.vue`、`MyAllocateTask.vue`；规范见 `wms-frontend-guidelines.mdc` §2。
- **TransferGoods / GridTable 自定义列刷新记忆**：vxe-table v3 列显示偏好存于 localStorage 键 **`VXE_CUSTOM_STORE`**（JSON 内以 table **`id`** 为子键），**不是** `customConfig.storageKey`（v3 无此字段）。`GridTable` 已设 `custom-config.storage: true`，页面须给 `<grid-table id="transfer_goods_columns">`；`checkMethod` 排除勾选/展开/操作等固定列。见 `TransferGoods.vue`。
- **旧页重构红线（最高优先级）**：历史代码不得丢失或改变既有行为，也不得擅自增加需求未授权的逻辑、属性、字段或参数；重构前读 `.svn/pristine` 或 git 旧版，按「表头→表体→展开行→筛选→操作→script」六段建立基线，diff 审查同时检查意外删除与意外新增；完整规则见 `.cursor/rules/wms-refactor-redline.mdc`。
- **TransferGoods 拓展属性横幅**：老代码 L460–462 在**折叠行也显示**「拓展属性：xxx」；不可仅放在展开组件内。做法：主行 `groupId` 列 + `TransferGoodsExpandContent` 双处保留。
- **TransferGoods 展开图标联动**：单行 `showDetailFlag` 须 `syncAllDetailFlag()` 同步表头「全部展开」图标（同老 `tagsOpenFlag`→`openAll`）；空列表时表头展开按钮 `:disabled`。
- **TransferGoods「新增出库单」靠右 + 列设置按钮**：左「更多功能」、右「新增」用 `justify-content:space-between` + `.transfer-goods-add-btn{margin-right:45px}`（同 `MyAllocateTask.vue` float:right）。勿改成左并排。
- **TransferGoods 窄屏/吸顶**：`initStickyTableHeader` 依赖 **页面滚动** + `external-scroll=true`（GridTable 破壁撑高）。误改 `external-scroll=false` 会让 vxe 内部锁高、滚动进表格容器 → 表头吸顶失效、行数少时出现巨大空白。小屏列宽用 12px + 收紧列宽，勿关 external-scroll。
- **TransferGoods 表头字段不可丢（红线）**：重构 vxe-column 时必须逐列对照老 `ul/li` 表头。例：仓库列老代码为「起始仓库→目标仓库 + 备注」两行，非仅两个仓库名；商品总量列为「商品总量 + 种类数(红)」，不可拆成「商品+总量」。见 `.svn/pristine/7b/7ba8329...` 第 118-121 行。
- **DynamicFilterBar 少选项 popover 过宽**：误把 HistoryOrderlist「排序」用的 `width:400px` 套到「有无库存/是否扣款」等 3 项筛选 → 弹层空白过大。正确：`type:'radio'` 走组件内置紧凑弹层；仅多选项（排序）保留 `sort-pill-group--wide`。
- **TransferGoods 表体巨大空白 + 表内滚动条**：服务端分页页误用 `useUniversalSticky` + `allocateTaskList2` 虚拟切片 → wrapper 被注入 padding 占位。做法：`:external-scroll="true"` 绑 `allocateTaskList`，表头吸顶改 `initStickyTableHeader`（同 `MyAllocateTask.vue`）。
- **TransferGoods 最后一行被分页挡住**：fixed 分页 + 静态 `padding-bottom:48px` 不足（含「已选择」时分页栏更高）。做法：`updateFixedPaginationFooter` 按 `footer.offsetHeight + 16` 动态设根容器 `paddingBottom`，并监听 footer `ResizeObserver`。

### 2026-06-16

- **Demo 无顶栏/侧栏**：注册成顶级路由（同 print 页）→ 不经过 `Home.vue`。需要导航壳的业务/demo 页必须挂 `Home.vue` children 并设 `meta.homeid`/`parentpath`；页面高度用 `calc(100vh - 84px)`。
- **Demo 打开仍是首页**：嵌套路由父组件用了 `Index.vue` → 无 `<router-view>`。做法：独立 demo 用顶级路由；业务 demo 挂 `Home.vue` children。见 `src/router/index.js` `/demo_modern_list`。
- **TransferGoods 上千条 Vue warn**：模板用了 `isBanDeduct` 等但未在 `data()` 声明 → keep-alive 下整站白屏。做法：模板新增变量必须同步 `data`/`computed`。
- **现代化 class 不生效**：`wms-modern-page` 等在参考页是 scoped 样式 → 新页/demo 须复制 layout 相关 CSS 到本页 `<style scoped>`。
- **TransferGoods 列表现代化**：旧 `ul/li` 列表改 GridTable 时，展开行内容抽到 `TransferGoods/TransferGoodsExpandContent.vue`；**服务端分页**用 `:external-scroll="true"` + `allocateTaskList` + `initStickyTableHeader`，勿用 `useUniversalSticky` 切片。参考 `MyAllocateTask.vue` + `TransferGoods.vue` 前 506 行。
- **TransferGoods 双滚动条/横向滚动条**：缺 `body.warehouse-update-page-active` + `body.history-order-hide-scrollbar` 全局 CSS → `common-page-container` 与 `main-topmenu-content-wrapper` 双垂直滚动；vxe-table 未禁横向滚动。做法：对齐 HistoryOrderlist 补全局样式 + `:scroll-x="{ enabled: false }"` + `.history-order-table-wrap` 内 `overflow-x: hidden` / `table-layout: fixed`。见 `TransferGoods.vue` mounted body class + style 块。
- **老页面纯布局改动别手动跑 i18n hook**：像 `TransferGoods.vue` 这种历史 `$t()` 已大量混用旧 key/命名空间的页面，若本轮只改列宽、样式、布局，手动执行 `check-vue-i18n.js` 可能批量重写原有 `$t()` 路径并顺带往 `cn.js` 追加无关 key。做法：仅在本轮确实新增/修改了用户可见中文时再跑 hook；纯展示改动只做人工 spot check + lint。见 `src/views/warehouse/TransferGoods.vue`。
- **TransferGoods 分页器消失**：`history-order-table-wrap` 用 `height:calc(100vh-43px)` + 上方筛选区撑高页面，分页 `sticky` 被 `overflow-x:hidden` 打断，落在视口外。做法：分页改 `position:fixed` + `initFixedPaginationFooter` 同步 left/width（同 `MyAllocateTask.vue`），表格区 `height:auto` + `padding-bottom:48px`。
- **OutOfWarehouseFee 第一版视觉回退**：DOM 已包 `fee-table-block`/`fee-tip-banner` 后，阶段 B 的 CSS 仍选 `.el-tab-pane > ul` → 边框/斑马纹样式全失效、页面「又丑又旧」。做法：CSS 必须选 `.fee-table-block`、`.fee-tip-banner`；改 DOM 后跑 div 平衡 + `npm run build`。见 `OutOfWarehouseFee.vue`。
- **node-sass 4 样式编译**：收费子页等样式优先用 **普通 `<style>`**（不用 `lang="scss"`），避免 node-sass 4 解析 `min()`、嵌套 `&`、`#` 选器报错。若必须用 scss，改完须跑 `npm run build` 或看 dev-server 终端。
- **TransferGoods 1427 箱列表默认展开**：1427/1004 去掉箱级 `el-collapse`，vxe 行展开即显示箱头+商品表；结构用 `.expand-box-block`。见 `TransferGoods/TransferGoodsExpandContent.vue`。
- **HistoryOrderlist 筛选按钮位置**：与 `TransferGoods.vue` 相同——`#actions` 槽（搜索/高级查询）由 `DynamicFilterBar` 默认 **follow** 紧跟 `#primary` 最后一项 inline 排列；**不要**加 `flex:0 0 100%` 独占一行。本页仅复用 `.modern-filter-container .el-form-item { margin-bottom:6px; margin-right:6px }`，**勿改公共组件**。
- **用户要求**：每轮对话结束 Agent **自动**把可复用教训写入 rules/skills，无需用户重复提醒。规则见 `.cursor/rules/wms-agent-self-upgrade.mdc`。

### 2026-06-18

- **Admin 订单日志弹窗**：管理员页查跨企业订单日志走 `POST /findOrderLogByAdmin`（`supplierId` + `orderId`），返回 `orderTaskLogs` + `failedLogs` 两个列表；`failedLogs` 时间字段为 `opertime`（小写）。API 加 `api_order.js`；旧 admin 页可直接 `el-dialog` + `el-tabs` 双表，不必强行 CommonDrawerForm。见 `Admin_AllCompany.vue`。
- **StockClassList tab 切换报错 `findOutWarehouseFee1 is not a function`**：外层 Tab 容器 `watch tab_name` 误写 `this.findOutWarehouseFee1()`，方法实际在子组件 `OutOfWarehouseFee.vue`。做法：统一 `const ref = this.$refs.OutOfWarehouseFeeRef; if (ref) { ref.findOutWarehouseFee1(); ... }`。见 `StockClassList.vue`。

### 2026-06-24

- **WaitChecklist 批量操作级联菜单**：原「批量操作」+「批量操作2」平铺 25+ 项 → 合并为一个按钮 + `el-popover` 内嵌 `el-cascader-panel`（`expandTrigger:'hover'` 左右两栏，与 Cascader 一致）；选项在 computed `batchOpCascaderOptions` 按分组构建，每项 `visible` 与原 `v-show`/`v-if` 一致，点击叶子 `@change` → `handleBatchOpChange` 分发原 method；`checkIsBatch` = 原 `checkIsBatch` ∪ `checkIsBatch2`（含 `this.openHwc`）。见 `WaitChecklist.vue`。
- **PDA 加工配货任务**：新页 `pda_produceTask_delivery.vue`，路由 `/produceTask_delivery_p`（挂 `pda_storage` children）。**菜单**：与 `50321`→`/produceTask_p` 同组（`pda_index` id=8 / `pda_storage` tabList3）；用户已有加工管理任务菜单时，前端 `injectProduceTaskDeliveryMenu` 在其后插入 `/produceTask_delivery_p`；**暂仅 supplierId 1004、1963** 可见，无需后台单独配菜单 path。扫配货批次 `pickBatch` → `findProduceTaskDeliveryList` → 先扫仓位再扫 SKU；扫描 SKU 配货 +1 走 `allocateDeliveryBySonStock`（`deliveryQuantity:1`，含 `sonStockId/gridId/gridCode/nameCn/totalQuantity`）；配满后 `sortDeliveryList` 沉底不删行；「结束任务」走 `updateTaskStateFromPickingToGenerating({ pickBatch })`（配货中→生成中），未配完需二次确认；左滑「配货异常」走 `produceTaskDeliveryException`（**暂仅 supplierId 1004** 显示左滑入口）。API 见 `api_produceTask.js`。**UI 参考 `pda_produceTask.vue` 现代卡片**：灰底 `#f0f2f5`、圆角卡片+阴影、步骤 chip、统计双栏、卡片内进度条；`slip-main` 单块 `task-card` 设 `display:block;width:100%`；左滑仍用 `slipbtns`。**流程强制**：必须先扫仓位（`scanMode=grid`）再扫 SKU（`scanMode=sku`）；`fillSku` 不可自动填仓位；未扫仓位点 SKU / 直接扫 SKU 提示 `Extend10.请先扫描仓位再扫描SKU`。
- **左侧菜单不高亮**：`Home.vue` 左侧 `el-menu` 的 `:default-active="$route.meta.parentpath"` 须与后台菜单 `child.path` **大小写完全一致**；**未配置 `parentpath` 则永不亮**。见 `src/router/index.js`。
- **跨模块共享 path 误切顶栏**：后台多个 homeid 可配同一 path（如代理仓 `/allorderlist`），但 `router` 里该路由 `meta.homeid` 仅一个（订单=7）。`Home.vue` `$route` 监听若只读 `meta.homeid` 会切到订单模块。做法：左侧菜单点击时记 `menuNavigationHomeId`，或 `isPathInCurrentLeftMenu` 保留当前 `activeIndex`。见 `Home.vue` `handleLeftMenuSelect`。
- **卖家全部订单搜索后左侧菜单变空白**：`CustomerHome.vue` 的 `/myallorderlist`（`homeid=92`）复用 `AllOrderlist.vue`，搜索单号后按订单状态自动 `push` 到 `/waitchecklist` 等 `homeid=7` 路由，导致页面外壳从 `CustomerHome` 切到 `Home`，卖家菜单没有 7 号模块时侧栏为空。正确：状态自动跳转必须以当前路由容器为边界，`homeid=92` 的全部订单只在当前页展示结果；`homeid=7` 的 `/allorderlist` 保留原状态跳转。见 `src/views/ordermanagement/AllOrderlist.vue`、`src/router/index.js`。

### 2026-06-26

- **重构红线（用户强制）**：未明确要求删除时，**禁止删/合并**业务代码（reportType 块、export 入参、v-if 分支等）。完整规则见 `.cursor/rules/wms-refactor-redline.mdc`；改后逐块对照 `exportXxx()` + grep 分支数。违规先回滚 `.svn/pristine`。
- **OrdermanagementReport 报表类型表单不可合并**：修 `ReportDateTimeRangePicker` 时误把 reportType 13（货值统计：单日期 + `warehouseId` + 分销商品）与 15（进销存：`datetimerange` + `warehouseids` + 全部客户）合成 `13||15` 一块 → 进销存导出读 `times[0]/times[1]` 失败。做法：各 reportType 独立 `v-if` 块，改完对照 `exportXxx()` 入参逐字段核对。见 `OrdermanagementReport.vue`。
- **ReportDateTimeRangePicker 手动时分秒不生效**：`defaultTime` 用 computed 动态取「当前时分秒」，父组件重渲染时 Element UI 会覆盖用户手调时间；普通 `v-model` 归一化只会在 Element UI 日期范围面板点击「确定」后运行，用户点击今天的结束日时，面板会先在内部应用 `defaultTime` 的 `23:59:59`，因此输入框在确认前仍会显示 `23:59`；`normalizeReportTimeRange` 还把历史结束日强制改 23:59:59。该报表选择器统一使用浏览器/电脑本地当前日期和时间，不做员工时区转换。做法：`defaultTimeRange` 固定 `['00:00:00','23:59:59']`；`disabledDate` 用 `getFullYear/getMonth/getDate` 比较浏览器本地日历 YMD 与浏览器本地今天；结束日为浏览器本地今天且晚于浏览器本地当前时刻时截断到本地当前时间，历史日期默认仍为 `23:59:59`，用户手动设置的合法时分秒保持不变；使用 `pickerOptions.onPick` 配合 mounted 后取得的 picker 面板 ref，在 Element 完成 `defaultTime` 赋值后按浏览器本地当前时间立即调整面板 `maxDate`，同时保留最终 `normalizeReportTimeRange` 按浏览器本地当前时间兜底，不弹警告消息；内层 `el-date-picker` 用 computed `innerValue` 双向绑定；所有报表类型（含 type 32）统一复用公共组件。见 `OrdermanagementReport.vue`、`ReportDateTimeRangePicker.vue`、`timeUtils.js`。
- **UpdateWarehouse 仓位条码打印顺序**：`printGridCodePdf` 的 `locationName` 须统一 **列优先**（`locationY → locationX`），与页面 `formatAndSortGridList`（行优先 X→Y）无关；勿再按 `supplierId` 分支。无坐标仓位追加在已排序列表末尾。见 `UpdateWarehouse.vue` `getmabangskulabelurl`。
- **PDA 上架量输入框过窄**：三列中间塞固定宽输入会挤到看不清。做法：上方保持原「图 + 信息 + 仓位」三列；下方整卡宽 `.qty-metrics-bar`（应上架 | 到货量 | 上架量三格灰底条），上架量占第三格 `flex:1.15` 白底输入。勿单独整行 label+input，勿 media 换行改仓位区。见 `pda_shelf.vue`。

### 2026-06-27

- **抽屉内多列表格防横向滚动**：详情抽屉 60% 宽塞 6 列时，用原生 `table` + `table-layout:fixed` + `<colgroup>` 百分比 + 外层 `overflow:hidden` + 单元格 `text-overflow:ellipsis` + `:title` 悬停全文；`v-if` 控制无数据不展示。见 `WaitChecklist.vue` 物流 tab `wait-checklist-declare-section`。

### 2026-06-30

- **CargoTransitOut 主表双横向滚动条**：不要用 `:scroll-x="{ enabled:false }"` + vxe body/header wrapper `overflow-x:hidden` 叠加压滚动，容易形成页面级 + 表格级两条横向滚动。正确：先参考 `/cargo_transit_task`，父页根容器保留 `padding:10px`，子表容器只 `width:100%; position:relative;`，再把主表百分比 `min-width` 总和收敛到 100% 以内。见 `CargoTransitTask.vue`、`CargoTransitOut.vue`、`CargoTransitOutTable.vue`。
- **StockClassList / OutOfWarehouseFee 双垂直滚动条**：父页 `StockClassList.vue` 的 `.el-tabs__content` 设了 `max-height`+`overflow-y:auto`，与 `Home` 的 `.common-page-container` / `.main-topmenu-content-wrapper` 叠加成双滚动。做法：mounted 加 `warehouse-update-page-active`；`.el-tabs__content` 改 `overflow: visible`；`.modern-tabs` 勿 `overflow: hidden`（否则 sticky 表头失效）。表头仍用 `.scroll-float-fixed`。见 `StockClassList.vue`。
- **OutOfWarehouseFee 表头吸顶**：**内层 Tab 内**表格（`.fee-inner-tabs .scroll-float-fixed`）禁用 sticky，防盖住首行；**底部 SKU 自定义收费**（`.fee-sku-class-table`）单独恢复 sticky，`top: var(--scl-sticky-top-outer-tabs)` 贴外层 Tab 下沿。见 `StockClassList.vue`。
- **StorageFee 规则表响应式**：**不要横向滚动条**；CSS Grid 用 `minmax(0, fr)` 弹性列 + 单元格 `min-width:0`；控件 `max-width` 随列收缩；仓库/卖家省略号。见 `StorageFee.vue`。
- **打印页功能临时注释**：用户要求“先把某功能代码都注释”时，需连同入口按钮、`data` 状态、methods 主体一起停用，避免只隐藏按钮却保留导出逻辑成为无入口残留代码。见 `printAllocateTaskNew.vue` 的 PDF 导出。
- **TransferGoods 小屏勾选列被裁切**：`fixed="left"` + `external-scroll` + `Home` 的 `overflow-x:hidden` 会导致勾选列整块消失。做法：去掉勾选/发货列 `fixed`；勾选列锁 `40px` + `col-checkbox`。**已下沉到 `GridTable.vue`**：`table-layout:fixed`、`ensureCheckboxColumnVisible()`、`col--checkbox` 40px；各页勿再 `fixed="left"` 勾选列。见 `GridTable.vue`、`TransferGoods.vue`、`HistoryOrderlist.vue`。
- **PDA 扫描页现代化布局**：扫描类 PDA 页按 `pda_produceTask_delivery.vue` / `pda_pre_sign_basket.vue` / `pda_replenishTask.vue` 标准做灰底 + 10px 间距 + 白色 top-card + 大扫描输入 + 轻卡片列表；不要做大面积蓝色渐变头图。重构时先核对 `v-model/ref/@keyup.enter/@click/v-if/v-show` 是否 1:1 保留，再改样式。见 `pda_scanorder.vue`。

- **PDA NutUI 主题与搜索框内部样式不一致**：NutUI 的 `type=primary/actived` 可能生成独立默认红色边框/图标，`nut-searchbar` 还会自带搜索图标和内部输入壳；仅覆盖外层按钮或 input 会残留红色、双图标和双层边框。正确：主题操作按钮优先使用页面原生 button 明确应用 `#4697f3`，红色只留给删除/库存盘0；自定义扫描壳时传 `hasIcon=false`、`hasSearchButton=false`，并统一清理 `.search-input/.nut-searchbar-input/input` 的边框、背景、高度和 padding。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 操作步骤条必须在长文案和组件全局样式下稳定**：独立卡片会割裂流程；`grid + flex:1` 连接线、普通 span 数字、重复 `STEP 0X` 标签和末步卡片强调会造成节点错位、数字跑出圆圈、线条穿插和视觉噪音。正确：每个步骤单层 `position:relative`，轨道和圆点锁定 `width/min/max-width` 30px，圆点绝对定位且 `overflow:hidden`；数字不作为普通文本，改由 `.step-N::before` 绝对居中渲染，避免 NutUI 全局 span 样式参与排版；连接线必须由同一个 30px 固定轨道 `.guide-track::after` 绘制并以 `left:14px` 对准圆心，不能挂在外层 `.guide-item` 上；静态步骤说明不要再包 `nut-sidenavbar`，其全局导航样式会给首项注入额外缩进；右侧只保留正文。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 顶部操作区窄屏不要隐藏标题或强制按钮换行**：用 `@media` 把扫码行改纵向会让辅助按钮掉行；隐藏左侧标题并给主按钮 `flex:1` 会让按钮吞掉整行。正确：首行标题区 `flex:1; min-width:0`，主按钮固定 96~104px；次行扫码框 `flex:1; min-width:0`，辅助按钮固定 84~92px 且 `white-space:nowrap`，窄屏只缩 gap、图标和字号，不隐藏业务文字。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 表单侧栏长文案按钮会被裁切**：「标签 + 输入 + 右侧操作」若右侧仍用 `span=6` 且按钮 `w-100` + 极小 padding，`待卖家确认`/`获取任务` 会在 340/360px 被挤成半截或贴边难看。正确：扫码行改 flex（按钮 `flex:0 0 auto` + 正常 `12px` padding，输入 `flex:1;min-width:0`，展开链接不收缩）；侧栏放不下的长操作按钮改为条件显示的整行全宽按钮；checkbox 用 `nowrap`。见 `src/views/pda/pda_return_order.vue`。

- **PDA 商品名称不能只靠无标签省略展示**：重排商品卡时若把 `stockName` 改成无标签两行文本，窄屏会被 SKU/卖家 SKU/备注区挤压，用户会认为商品名称被删除。正确：保留 `{{$t('商品名称')}}` 标签和 `item.stockName` 绑定，商品名称使用独立 flex 行，文本区 `min-width:0` + 两行省略，确保字段明确可见。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 操作提示抽屉在小屏接近全屏**：NutUI `nut-popup` 右侧抽屉即使设置 `width:82%`，在 340/360px 设备上仍会占据过大空间，用户难以感知抽屉边界。正确：保留 `position="right"` 和原业务内容，宽度使用 `70vw` 并设置 `max-width:360px`，只做局部 CSS 调整。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 盘点扫描框提示写死仓位文案**：任务盘点页实际首扫 SKU、后续可扫 SKU 计数或仓位结束盘点，但 `nut-searchbar` 同时使用固定 `Shelf.请输入仓位` 和固定 placeholder，导致提示与流程不一致。正确：用 computed 根据 `inventorylist` 与 `scanningNum` 返回「请先获取任务」/「请扫描SKU开始盘点」/「请扫描SKU计数(再次扫描仓位结束盘点)」，同时绑定 `placeText` 和 `placeholder`；仅调整展示提示，保留 `v-model`、回车事件和 `onKey` 分支。见 `src/views/pda/pda_inventory_task.vue`。

- **PDA 任务卡删除按钮位置**：删除入口若放在数量指标栏的「已配货量」单元格内，会挤压指标并与按仓位盘点页的卡片布局不一致。正确：保留原权限条件和 `deleteInventory(item)` 参数，将删除入口放到商品卡右上角独立 `.task-card-actions`，卡片设 `position:relative`，操作区使用绝对定位和浅红边框按钮样式，并给内容区预留右侧空间。见 `src/views/pda/pda_inventory_task.vue`、`src/views/pda/pda_inventory.vue`。

### 2026-07-10

- **PDA Tab 切回后扫码提示不能沿用旧流程状态**：`pda_inventory.vue` 的 `onTabClick('grid')` 只清空列表/计数、未重置 `inputmsg`，导致按仓位流程扫描过仓位后再切 Tab，切回按仓位仍显示“请扫描 SKU”，用户会误以为输入框仍处于按 SKU 流程。原因是两个 Tab 虽然有独立 `v-model`，但按仓位的 `inputmsg` 是可变业务状态。正确：Tab 切换必须同时恢复该 Tab 的首屏占位、当前输入值、计数和上下文状态；切回 grid 至少重置 `inputmsg` 为“请扫描仓位”、`stockSku`/`gridCode`/相关计数按业务初始态核对。见 `src/views/pda/pda_inventory.vue` 的 `onTabClick`。

- **PDA 扫码框下方避免连续色块**：扫描输入下连续叠加橙色提示 Banner、灰色上下文块和悬空危险操作，会让页面显得碎且臃肿。正确：扫描提示改为透明单行辅助文字 + 细分隔线；当前仓位、筛选、结束操作使用白底紧凑状态栏，统计仅有数据时展开。见 `src/views/pda/pda_inventory.vue`。

- **PDA 操作提示按钮位置需对齐成熟扫码页**：若操作提示作为客户行右侧文字链接，会削弱扫码主流程并与任务盘点页不一致。正确：客户选择独占辅助行，操作提示放在扫码输入右侧，使用固定 92px×44px 的按钮；见 `src/views/pda/pda_inventory.vue` 与 `src/views/pda/pda_inventory_task.vue`。

- **PDA 空数据首屏不要用大占位卡**：扫码作业页顶部已经展示动态扫描提示时，空列表再用 190px 虚线白卡、超大图标和重复文案，会让首屏像 Demo 占位并割裂操作流。正确：空状态使用约 100px 的透明轻提示，只保留小图标和一行当前引导；客户选择与操作提示合并为辅助工具栏，扫码框保持唯一视觉焦点。见 `src/views/pda/pda_inventory.vue`。

- **PDA 空状态 `v-else` 被弹窗打断**：列表 `v-if` 后插入 `el-dialog` 再写 `v-else` 会使 Vue2 条件链失效或模板行为不可靠。正确：空状态 `v-else` 必须紧邻列表 `v-if`，弹窗放在完整条件链之后；见 `src/views/pda/pda_inventory.vue`。

- **PDA 页面“换壳式重构”不达标**：只给旧 `nut-row/nut-col` 结构套白卡、改颜色和圆角，会导致首屏仍像旧表单，扫码主操作、流程状态和结束动作层级不清。正确：保留业务绑定但重排为“顶部轻量 Tab → 单一扫码作业控制台 → 流程提示条 → 活动上下文 → 整卡数量指标 → 左滑商品卡”，扫码框全宽突出，结束动作独立布局；以 `src/views/pda/pda_inventory.vue` 对标 `src/views/pda/pda_inventory_task.vue`。

- **virtual-printer.js 开发 Mock 增强**：`getData` 默认配置需含 `100X150`/`A4`/`A5` 等常见纸张；`printFile`/`printFileLocalPDF` 收到 `url` 时用系统命令打开 PDF（macOS `open`）便于开发预览；`writeData`/`getData` 按 `fileName` 持久化到 `.virtual-printer/configs/`（前端 `localStorage` 清空后仍可恢复）。现象：本地页面装有打印代理插件或从局域网设备访问时，直连 `localhost:9998` 会被插件分流或指向访问设备自身。原因：开发调用未经过 WMS devServer。正确：启动 `npm run virtual-printer`（端口 9998），开发环境由 `vue.config.js` 的 `/virtual-print` 同源代理转发，并在 `src/api/mabangprint.js` 中仅开发环境绕过插件；生产环境仍保持原打印插件/本机打印服务行为。见 `virtual-printer.js`、`vue.config.js`、`src/api/index.js`。

### 2026-06-29

- **波次详情单条浏览器打面单勿多调 findOrderAddressPdf**：`findBatchmanagementDetail` 的 `orderList` 已带 `addressPdf`；单条打印应 `normalizePdfUrl(order.addressPdf)` → `window.open` → `updatePrintFlagByTaskId`（同 `WaitShiplist.showDialog`）。`findOrderAddressPdf` 仅用于待发货/波次**批量**打印队列刷新 PDF。见 `BatchmanagementOrder.vue` `printOrderLabel`。
- **HistoryOrderlist 批量操作 cascader 分组**：历史订单页批量菜单按「批量组包 / 批量上传订单信息 / 批量更新订单/物流状态 / 独立项 / 批量其他操作」五级联 `el-cascader`（`batchCascaderOptions` + `handleBatchCascaderChange`），每项 `visible` 与原 `v-show`/`v-if` 一致，勿删未入分组表的项（如批量改派骑手、导入退货单、批量导出满足搜索条件等）。见 `HistoryOrderlist.vue`。

- **老页面 i18n Hook clean 后仍可能漏检模板文案**：`check-vue-i18n.js` 会跳过 `v-if`/`v-show`/`v-for` 整行，模板裸文本也仅依赖有限正则，因此 `StockWarehouseList.vue` 中 `v-if` 同行的“已停用”、普通属性 `label="无包材"`/日期 placeholder、某些裸中文提示可能在 Hook 报告 `additional_context=null` 后仍漏检。正确做法：老页面做最终翻译补齐时，Hook clean 后还要对 template 人工/rg 复核硬编码中文属性与 `>中文<` 裸文本；仅翻用户可见文本，保留比较常量/注释/URL。见 `src/views/stock/StockWarehouseList.vue`、`.cursor/hooks/check-vue-i18n.js`。
- **复杂匹配规则抽屉不适合分类导航双列卡片**：条件数量多且用户按历史顺序逐项阅读时，左侧分类导航 + 双列卡片会切断阅读路径；但把“已选择条件”固定移到顶部也会失去边编辑边核对的效率。正确：保留 `CommonDrawerForm`、基本信息分区和固定底栏；条件按业务章节使用单列 `el-collapse` 顺序展示；桌面端采用约 72% 编辑区 + 28% 右侧 sticky 摘要，摘要可收起且收起后编辑区扩展，小于 900px 时摘要回到顶部；不改任何 `v-model`、供应商分支或保存参数。见 `src/views/logistics/LogisticsruleList.vue`。
- **多模块双列折叠不能拆成两个跨模块长列表**：把多个业务模块直接分配到左右两个长 `el-collapse` 会破坏模块层级，两个独立 `v-model` 还会让同一模块左右各展开一项；父 Grid 整面灰底则会在较短列下方显露。正确：每个业务模块独立保留标题和双列容器，左右列使用同一个 `v-model + accordion`，保证模块内最多展开一个条件、模块之间互不影响；容器 `align-items:start`、列 `align-self:start;height:auto`，只用居中渐变绘制列间分隔，小屏统一回落单列。不得改模块顺序、字段绑定、条件和保存逻辑。见 `src/views/logistics/LogisticsruleList.vue`、`tests/logistics-rule-condition-columns.test.mjs`。

- **TransferGoods 抽屉连续选商品后图片与 SKU 错位**：右侧只传 `stockSku` 并再次 `findIndex` 会取到错误库存行，仓位异步还依赖可漂移的数组下标；左侧 `el-table` 又在无 `row-key` 时用 `splice(0, 0, item)` 头插，可能复用旧行 DOM，最终出现图片与 SKU/名称不属于同一商品。正确做法：选择按钮直接传 `scope.row`，全程使用当前行及 `stockId`，左表设置以明细 id 或 `stockId + box_code` 组成的稳定 `row-key`；用 `try/finally` 统一解除按钮锁，仓位查询失败时停止加入；删除按当前行对象移除。见 `src/views/warehouse/TransferGoods.vue`。

- **TransferGoods 连续选品后图片与 SKU 串行**：左侧 `el-table` 未配置稳定 `row-key`，新商品又用 `splice(0, 0, item)` 头插时，Vue/Element 可能按索引复用行节点；同时历史选择逻辑只传 SKU 并在异步前保存 `stockList` 下标，会增加数据对象漂移风险。正确：右侧直接传当前 `scope.row`，左表按持久化 `id` 或 `stockId + box_code` 设置稳定 `row-key`，仓位结果回写当前行，删除按行对象移除；选择按钮用 `try/finally` 解锁，仓位查询失败时停止添加。见 `src/views/warehouse/TransferGoods.vue`。

### 2026-07-24

- **UI 走查被登录或滑块验证阻断时不能伪造运行时缺陷**：直接把浏览器尝试标成 complete 会让报告误称目标页已验收。正确：保留静态发现，记录尝试过的视口、重定向地址、截图和未测试状态，将状态标为 `runtime-blocked`；不读取凭据、不自动处理 CAPTCHA。见 `scripts/wms-ui-audit/core/audit.mjs`、`reports/wms-ui-audit/runs/<run-id>/runtime-status.json`。
- **审计工具不能把内部流水线暴露给使用者**：要求业务同学记住 `scan/report/run-id` 会造成使用门槛。正确：Codex 入口只需“检查 /路由”，内部自动完成扫描、服务、浏览器和报告；终端备用入口保持为 `npm run audit -- /路由`。见 `.cursor/skills/wms-ui-audit/SKILL.md`、`package.json`。

- **3D 仓库默认镜头被“无区域”拉偏、标题远景不可读**：直接用全部货架总边界 fit camera，会让 `zone = -1` 的离群货架决定画面中心；父组件 `$nextTick` 又可能早于 `requestAnimationFrame` 分批建模完成，导致首次自动排版只拿到前几条货架并再次覆盖正确镜头。正确：默认取景及首次自动排版后的二次取景都优先使用正式区域货架，正式区域为空时才回退全部货架；自动排版必须等全部 `interactableObjects` 构建完成，边界按旋转后的实际长宽计算并对空结果判空。标题仅按货架长度、名称长度和相机距离限幅缩放 Sprite，不缩放货架 Group/hitBox，不修改坐标、尺寸或保存数据。见 `src/views/stock/Warehouse3D/Warehouse3DScene.vue`、`src/views/stock/Warehouse3D/warehouseRack.js`。
- **本地可视化审计入口不能接受任意路径或把静态扫描冒充完整 UI 验收**：页面直接接收文件路径会产生目录穿越风险，只显示“检查完成”又会让用户误以为已做登录后浏览器走查。正确：扫描输入只能来自 `src/router/index.js` 解析出的路由白名单，报告资源限制在 `reports/wms-ui-audit/runs/`；静态扫描状态明确显示“运行时未执行”，浏览器实测仍交给 Codex Skill。见 `scripts/wms-ui-audit/dashboard/server.mjs`、`scripts/wms-ui-audit/dashboard/public/app.js`。
- **本地 AI 审查误称“纯本地只读”**：虽然 Codex CLI 使用 read-only 沙箱不写项目文件，但分析上下文会交给当前 Codex 会话 → 页面必须明确披露发送范围，并用 `--sandbox read-only`、`--ephemeral`、固定参数、禁用 shell、前后哈希校验和异常恢复保证“只分析、不改代码”。见 `scripts/wms-ui-audit/dashboard/ai-analyzer.mjs`、`scripts/wms-ui-audit/dashboard/public/index.html`。

- **本地 AI 工具依赖 Codex 会话，或把 macOS 钥匙串方案直接搬到服务器**：分析 provider 绑定 `codex exec` 会在桌面应用退出后失效；Linux 无钥匙串时若回退明文文件，或把 SDK/SVN 原始错误直接返回页面/机器人，还会泄露凭据 → 将 provider 设为显式的独立中转/Codex 选择；macOS 使用钥匙串，服务器必须由 `WMS_UI_AUDIT_CREDENTIAL_MASTER_KEY` 驱动 AES-256-GCM 凭据文件，无安全存储就拒绝写入；网页、机器人、日志和报告不返回密钥，运行错误只允许白名单安全文案；HTTP 地址持续提示未加密风险。见 `scripts/wms-ui-audit/dashboard/ai-provider-relay.mjs`、`scripts/wms-ui-audit/dashboard/encrypted-file-credential-store.mjs`、`scripts/wms-ui-audit/dashboard/server.mjs`。

- **只指定一个抽屉样板会导致错配**：把 `/mycustomerlist` 当成所有 `CommonDrawerForm` 的唯一视觉模板时，窄单列表单、宽幅配置、高级查询和大量条件抽屉会被套入不合适的列数、宽度或内容结构。原因是公共组件只统一壳、滚动和 footer，不统一业务内容。正确：先以 `src/components/Common/CommonDrawerForm.vue` 为结构权威，再按同业务/同复杂度从 `/warehouselist`、`/updateWarehouse`、`/stockWarehouselist`、`/shippinglist`、`/logisticsrulelist` 选择已验证局部参考，最后才用 `/mycustomerlist` 兜底；同时验证 body 单滚动、footer 安全距离和弹层不裁切。


- **企业微信自然语言被当成自由提示词全部拒绝**：固定 parser 报错后直接返回，会让“我要优化 /upper_shelf 怎么改”和项目知识追问不可用 → 固定命令必须保持优先，但失败后进入安全意图路由；页面任务只使用路由白名单，知识问答只检索管理员批准的 WMS/TMS 同域资料；群聊上下文按群+成员隔离，`详情` 仍按群共享。见 `scripts/wms-ui-audit/wecom/natural-language-router.mjs`、`conversation-store.mjs`、`knowledge/`。

- **弹窗映射副本上传成功后再次打开不回显、表格超出 Dialog**：弹窗若用 `map + ...spread` 从任务箱规生成展示对象，上传成功只 `$set` 当前展示对象，关闭后会从未更新的原始 `boxSpecList` 重建而丢回显；同一固定高度容器内先放箱规面板、再让 vxe-table `height="100%"`，两者高度会直接叠加。正确：上传/删除成功按稳定 `specId` 同步展示对象与任务原始箱规对象；Dialog 内容使用纵向 flex，箱规区 `flex-shrink:0`、表格 wrapper `flex:1; min-height:0`，表格 100% 只占剩余高度。见 `src/views/task/CargoTransitOut.vue`、`src/views/task/outbound/CargoTransitOutTable.vue`。

- **上传 Dialog 不能只禁用底部关闭按钮**：网络较慢时，用户仍可通过右上角、遮罩或 Esc 关闭弹窗，上传回调随后更新已销毁的交互上下文；重复选文件还可能并发提交。正确：请求开始设置显式 loading 锁并禁用 upload/关闭按钮，`before-close` 拦截右上角关闭，同时按锁关闭遮罩和 Esc；成功、业务失败和异常统一在 `finally` 释放锁，程序仅在成功后主动关闭。见 `src/views/task/CargoTransitOut.vue`。

- **PDA 商品页新增虚拟 SKU 不走订单配对链路**：在 `/stockWarehouselist_p` 这类商品查询/维护页新增虚拟 SKU 时，若复用 `waitchecklist` 的配对参数会误带订单上下文并触发包材重算。正确：按商品资料维护链路调用 `API_STOCKWAREHOUSE.addVirtualSku` / `/addVirtualSku`，仅传当前商品的 `stockId`、`stockSku`、`companyId`、输入的 `virtualSku` 与 `keytype=99`；不要传 `matchingPackagingId`。见 `src/views/pda/pda_find_stock.vue`。

- **新增路由不显示在左侧菜单**：`src/router/index.js` 的 children 只负责路由匹配，`Home.vue` 的左侧菜单由 `API_MENU.findPageMenu` 返回值重建；前端新增路由若没有后端菜单记录就不会出现在菜单。正确：正式环境补齐后端菜单与权限；本地联调页可在 `NODE_ENV=development` 下向对应 `homeId` 的菜单组追加临时入口，生产环境不绕过权限。见 `src/views/Home.vue`、`src/router/index.js`。

- **打印插件工作台自动连接、窄屏溢出与 i18n 命名空间**：页面只在按钮事件中连接，或连接栏/多列网格子项缺少 `min-width:0`，会造成首次进入未连接和中等宽度横向滚动；新增文案若混入顶层或旧 `Extend10`，会违反 cn.js 只能在文件末尾最新 Extend 追加的约定。正确：在 `mounted` 调用带连接中/已连接幂等保护的连接方法，并为页面根、网格、卡片和表单行设置宽度收缩约束；页面外层保留明确的上下边距，左右列使用 flex stretch 保持卡片边界对齐；本轮新增文案统一追加到 `Extend11` 并在 Vue 中使用 `Extend11.xxx`，业务已有命名空间不迁移。见 `src/views/order/PrintWebSocketWorkbench.vue`、`src/views/order/WavePrintWebSocket.vue`、`src/i18n/langs/cn.js`。
- **波次业务页连接状态不能只在首次连接时判断**：10098 插件可能在页面打开后启动、重启或短暂断开，仅在按钮事件中连接会导致状态长期过期，业务打印时才暴露错误。正确：页面 `mounted` 立即执行连接与无副作用健康探测，并以串行保护的定时检查自动恢复；`beforeDestroy` 同时清理定时器和 WebSocket，业务页只展示只读状态，不提供手动断开入口。见 `src/views/order/WavePrintWebSocket.vue`。


- **PDA 未纳入 SVN 的 Vue 文件不能直接运行会写回的 i18n Hook**：现象是检查脚本会把同名中文键自动改成其他命名空间，产生本次需求之外的页面 diff；原因是 Hook 对 `?` 文件没有 SVN 基线保护，按候选顺序自动归一化路径。正确：先用 `svn status`/`svn info` 确认版本状态；未纳入版本控制的文件先复制到临时副本执行检查，真实文件只人工核对并保留已确认的 i18n 路径。见 `src/views/pda/pda_order_status_query.vue`、`src/views/pda/orderStatusQueryUtils.js`、`.cursor/hooks/check-vue-i18n.js`。



1. 重构前用 `ui-plan` 明确页面家族、已上线参考、布局边界、业务保护与验收矩阵。
2. UI 完成后用 `ui-review` 对照发布基准和 `CommonDrawerForm` 参考层级验收。
3. 再用 `code-quality` 检查 Vue 2 资源清理、稳定 key、deep watcher、i18n、组件职责和复杂度。
4. 需要一次性交付时用 `full-review` 合并并去重，仍只生成方案，不自动修改或提交 SVN。

- **波次接口包装层字段读取错误会让真实波次显示为空**：接口实际返回顶层 `code/msg/data/datas`，其中 `code` 可能是字符串 `'200'`，订单数组位于 `datas`；若严格比较数字 `200` 或读取 `data`，页面会误判无订单。正确：使用 `String(response.code) === '200'` 并读取 `response.datas`；展示和 `updatePrintFlagByTaskId` 均以 `platformOrderId` 为平台订单标识。见 `src/views/order/WavePrintWebSocket.vue`、`src/api/api_batchmanagement.js`。

- **按箱规创建展开区读取错误**：按箱规任务的箱规数据位于 `boxSpecList`，普通任务仍使用 `boxList`；如果展开区无条件读取单一列表，会导致其中一种任务显示错误。正确：卖家页面按 `skuJson` 的按箱规标识选择 `item.boxSpecList`，其他任务继续读取 `item.boxList`，箱内明细仍读取子项的 `detailList`。见 `src/views/seller/MyCargotransitOut.vue`。

- **空数据扫码工作台不能用业务态最小高度撑满首屏**：现象是无数据时扫码区下方出现大块白板，日志/反馈区被推离首屏，标题与输入控件失去视觉重心；原因是工作区和空态同时设置较大的 `min-height`，把“有数据时的滚动承载”误当成“无数据时的首屏占位”。正确：空态使用按内容计算的紧凑高度，工作区只在真实列表出现时由内容撑开；顶部按“页面身份 → 模式切换 → 主扫码输入 → 辅助连接状态”分层，1366px/1024px 再按行回落，不改变 `ref`、Enter 事件、权限和业务状态机。见 `src/views/order/WavePrintWebSocket.vue`。

- **扫码工作台控件不能无意义铺满**：输入框使用过度 `1fr` 会破坏 WMS 操作密度；只压 inner 高度会造成 Element UI prefix/suffix/clear/select 图标错位。正确：字段按语义设受控宽度，窄屏明确回落换行；outer、inner、prefix、suffix、icon 同步 `height/line-height/box-sizing`，容器使用 `display:flex; align-items:center; top:0`，并在 1440/1366/1024 宽度实测。见 `src/views/order/WavePrintWebSocket.vue`。
- **无成功 ack 的 WebSocket 任务不要复用单响应请求队列**：`getBatchWaybillList` 这类提交成功不返回 ack、后续还可能推送终态事件的方法若复用打印请求的串行 WebSocket，会让 pending 一直等到超时，终态事件还可能被误认成下一条打印响应。正确：仅需触发且不关心结果时使用目标页专属独立短连接，按 `{ methodName, data }` 发送后关闭；公共打印 helper 与历史页面保持基线不变。见 `src/api/wavePrintWebSocket.js`、`src/views/order/WavePrintWebSocket.vue`。
- **迁移打印 helper 不能把缓存包装层机械翻译成插件调用**：历史 `mabang_findprintconfig` 表面对应 `getData`，但实际先读 `localStorage/Cookie`，仅缓存缺失时才访问插件；如果改成每次直接发 WebSocket `getData(orderPrinterConfig)`，会改变页面真实调用行为。正确：先审计 helper 的缓存、fallback 和副作用，再决定 WebSocket 方法映射；目标页需要独立设置时使用路由专属存储键，禁止污染公共 `orderPrinterConfig`。见 `src/api/mabangprint.js`、`src/views/order/services/wavePrintOrderLabel.js`。