# Self-improvement backlog

## 2026-07-21：可见布局异常被错误判定为通过

### 事件

在 `simboss-black-widow` 的 P06 每日用量明细视觉验证中，生产页面必须保留历史业务要求的完整时间格式 `YYYY-MM-DD HH:mm:ss`。Open Design 原型使用较短的 `MM-DD`，同时将日期列固定为 `92px`。实现沿用了原型列宽，导致完整时间在每一行中断为两行，形成明显、重复的纵向拥挤。

测试截图已经完整暴露该问题，但报告以“文本在列内可预测换行、没有 overflow 或 collision”为由将其判为 `CLEAR`。因此这不是浏览器、截图或 Playwright 未覆盖，而是视觉评估者看见异常后作出了错误结论。

### 根因

- 直接根因：执行者没有严格落实 skill 已要求的 wrapping、重复组件对齐和视觉层级检查，把“没有技术性溢出”错误等同于“视觉布局合理”。
- 判断偏差：过度看重 DOM 几何指标、无 overflow、无 collision 和数据完整性，弱化了截图中肉眼可见的排版异常。
- 约束混淆：正确识别了“完整时间不能删减”这一内容约束，却错误推导为“当前换行表现也必须接受”。内容必须保留，不代表布局无需适配内容。
- 流程缺口：最终结论允许评估者直接写 `CLEAR`，但没有强制列出截图中的所有可见异常及其处置依据，主观误判因而没有在结论阶段被拦截。

### 通用教训

1. 始终分离两类约束：
   - 内容/业务约束：字段、格式、状态、计算和文案必须保留。
   - 展示约束：布局必须容纳真实业务内容，不得因原型使用了更短 fixture 而照搬不适用的固定尺寸。
2. DOM 指标只能提供证据，不能否决视觉证据。`scrollWidth <= clientWidth`、没有元素碰撞、内容未丢失，都不能证明排版合理。
3. 浏览器截图中出现明显异常时，不允许仅凭“可预测”“仍可阅读”“没有 overflow”判定通过。重复换行、节奏破坏、密度突变、对齐漂移和层级减弱都应先视为 finding。
4. 原型是视觉参考，不是覆盖历史业务需求的授权。原型 fixture 与真实内容 shape 不一致时，应保留业务内容并调整布局；不得为了贴图擅自缩短、隐藏或改写数据。
5. `CLEAR` 应是排除性结论：只有完成区域级检查且不存在未解释的可见异常时才能成立，而不是因为自动指标没有报错就默认成立。

### 后续优化候选（本次不实施）

- 引入强制的 **Visible Deviation Ledger**：逐区域记录每个肉眼可见偏差，并且只能归类为：
  - `INTENTIONAL`：有设计或业务证据支持；
  - `ACCEPTED`：有用户明确授权；
  - `FINDING`：需要修复；
  - `SKIP`：缺少权限、数据或前置条件，并写明原因。
- 禁止存在未归类偏差时输出 `CLEAR`；`CLEAR` 必须附带“区域已检查、未解释偏差为零、无未声明 skip”的负向证明。
- 在视觉裁决前增加对抗式复核：假设这是他人提交的截图，指出第一眼会圈出的所有异常，再逐项核对是否有证据允许保留。
- 对重复列表、表格、卡片和导航增加专门检查：真实最长内容、换行模式、行高一致性、列宽适配、信息密度及首尾行稳定性。
- 明确规则优先级：业务内容真实性 > 原型 fixture；视觉真源约束布局语言，但不授权改变已有业务字段和格式。

### Johari 复盘

- **Open Area**：截图已显示完整时间重复换行；业务要求明确必须保留完整格式；测试报告曾错误判为通过。
- **Hidden Area**：历史完整时间格式背后的具体产品原因未知，但用户已确认其为业务需求，验证时不得自行推翻。
- **Blind Spot**：评估者把“内容正确”和“展示正确”合并成一个结论，也把自动几何检查当成视觉质量代理。
- **Unknown Area**：何种列宽、响应式策略在全部真实数据和小尺寸下最合适，必须通过真实内容视觉验证确定，不能从当前事件直接假设。

## 2026-07-29：用 `diff.html` 支持逐项视觉验收

### 事件

在 `simboss-palette` 的业务页面布局治理中，一次变更同时涉及 action row、inline filters、
重复 local title、Tabs/content gutter 和卡池 grid。直接给出代码 diff、零散截图或一次性总结，
会把多个独立视觉决策压缩成一个笼统的“是否通过”，用户难以判断每一项的实际效果，也难以指出
某个局部需要调整。

执行过程中改为按审查组生成可浏览的 `diff.html`：每组只覆盖一个明确主题，页面并排展示
Before/After 截图，补充 route、问题描述、实现边界和确定性测量。用户先在浏览器中审阅，再逐组
选择“验收并继续”或“需要调整”。当用户提出购物车位置、badge padding 等增量反馈时，只更新
对应组的实现和证据，不必重新解释整个 Job。

这种模式显著降低了确认成本，也让视觉判断、功能证据和变更边界同时可见。

### 为什么有效

- **把抽象描述变成可比较对象**：用户无需在记忆中重建旧页面，Before/After 在同一视野中直接
  暴露间距、对齐、密度和层级差异。
- **缩小决策粒度**：一个 `diff.html` 对应一个视觉主题或一组强相关 route，避免一个局部异议
  阻塞或否定其他已经成立的改动。
- **保留上下文**：route、viewport、状态、问题、结果和影响边界与截图绑定，截图不再是缺少
  解释的孤立附件。
- **支持增量迭代**：用户反馈可以定位到具体 group；更新截图和 measurement 后可在同一 review
  surface 复验。
- **形成显式 gate**：用户确认发生在实现阶段与 commit/下一阶段之间，适合需要逐项审美判断、
  但又不希望反复阅读代码的工作。

### 推荐结构

`diff.html` 应是现有视觉证据的 review surface，而不是另一个未经验证的报告来源。建议每个
审查组至少包含：

1. **标题与边界**
   - 本组解决什么视觉关系；
   - 明确不改变哪些业务行为。
2. **route / state / viewport**
   - 使用稳定 route；
   - 标记 material state；
   - 写出截图的精确 CSS viewport。
3. **Before / After 并排证据**
   - 引用真实浏览器截图，不使用重新绘制的 mock；
   - 两侧尽量保持相同 viewport、数据和状态；
   - 截图必须先按原始分辨率人工检查。
4. **中性观察与结果**
   - Before 描述可见关系，不先写技术结论；
   - After 描述修复后的关系；
   - 避免只写“更美观”“已优化”。
5. **确定性证据**
   - 例如 gap、x delta、overflow、bounding box、URL 或交互结果；
   - measurement 支撑视觉结论，但不替代截图检查。
6. **影响面说明**
   - 指出 selector、component 或 shared style 的消费者；
   - 说明是否存在跨页面污染风险。
7. **逐项 decision gate**
   - `验收并继续`；
   - `需要调整`；
   - 用户反馈应绑定当前组，避免隐式接受未展示的改动。

目录可以沿用一次 visual run：

```text
.vdr-log/<run-id>/
├── screenshots/
│   ├── <route>-before-<viewport>.png
│   └── <route>-after-<viewport>.png
├── diff-<review-group>.html
└── diff-<review-group>-report.png
```

其中 `diff-<review-group>-report.png` 是对 review surface 自身的可见性检查，可用于确认图片已加载、
双栏布局无 overflow；它不能替代对原始 Before/After 截图的逐张检查。

### 使用条件

优先在以下场景使用：

- 同一 Job 有多个可独立验收的视觉主题；
- 用户需要在 commit 或进入下一 phase 前逐项确认；
- 修改涉及 shared style，需要同时展示 direct finding 与代表性 regression route；
- 视觉差异难以仅靠代码 diff 或文字准确传达；
- 用户可能基于视觉结果继续提出局部调整。

以下场景通常不需要：

- 单一、明确、可用一张 focused screenshot 说明的低风险属性修改；
- 纯功能、API 或不可见内部重构；
- 没有可比 baseline，且任务本质是 exploratory audit。此时应先使用 coverage report，不伪造
  Before/After。

### 约束与反模式

- 不允许为了生成对比页而补拍不一致的数据、状态或 viewport，然后把差异归因于代码。
- 不允许把缩略图看起来正常当作视觉通过；每张计入 coverage 的原图仍须单独检查。
- 不允许只展示成功 route，隐藏 finding、skip 或无法复现的状态。
- 不允许让自定义 `diff.html` 取代标准 `STEP_*` / `VISUAL_*` marker 和最终 run report；
  它是 human review layer。
- 不允许把多个无关问题塞进一张超长对比页；按可独立决策的 review group 拆分。
- 不允许在用户选择“需要调整”后继续下一组，除非该反馈被明确 defer。
- 不允许把 `diff.html` 当作产品代码或提交到业务产物；应存放在已 gitignore 的
  `.vdr-log/` 下。

### 后续优化候选（本次不实施）

- 在主 skill 中增加 **Human Visual Review Gate**：当用户要求逐项确认或一个 run 包含多个
  独立视觉主题时，建议生成按 group 拆分的 `diff.html`。
- 为 Before/After review surface 提供可复用 template，避免每次手写 CSS；模板仍应允许
  route-specific crop、measurement 和影响面说明。
- 在 report aggregation 中建立链接关系：
  `coverage row → original screenshot → diff group → user decision`。
- 为每个 review group 记录状态：`PENDING`、`ACCEPTED`、`ADJUSTMENT_REQUESTED`、
  `DEFERRED`，避免把“看过”误记为“验收”。
- 增加一致性 precheck：图片全部加载、natural size 非零、viewport 标记一致、document
  horizontal overflow 为零。

### Johari 复盘

- **Open Area**：并排 Before/After、明确 measurement 和逐组 decision gate 能降低用户的视觉
  验收成本；本次实践已出现多轮局部反馈并成功收敛。
- **Hidden Area**：用户希望按多细的粒度拆组、是否需要看到代码片段，仍取决于项目和审查习惯，
  不应由 skill 固定成唯一格式。
- **Blind Spot**：精美的对比页可能制造“证据完整”的错觉；若原图未检查、状态不一致或覆盖有
  skip，review surface 仍然不可靠。
- **Unknown Area**：何时自动生成 `diff.html` 最节省成本、模板应内置多少交互，需要在更多项目中
  观察后再整合到主 skill。
