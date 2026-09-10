---
name: concept-microcourse
description: 把任意概念做成「3 分钟能学完」的单文件 HTML 微课程：含权威来源考据、生活化类比、
  可视化图解与入门/进阶/挑战三层难度的即时判分自测。当用户想快速学习或搞懂某个概念（AI、技术、
  理论等），要求通俗易懂、带可视化并有习题的中文教学 HTML 时使用；视觉按 Anthropic/Claude 设计
  体系排版，文案极简，字号不小于 12px。This skill builds a single-file HTML micro-course that
  teaches any concept quickly with sourced facts, analogies, diagrams, and a tiered quiz styled
  after the Anthropic/Claude design system.
agent_created: true
---

# Concept Microcourse（概念速学 HTML 微课程）

## 目标

把任何一个概念，做成可直接打开的单文件 HTML 教学页：**来源准确、一看就懂、带图解、做完能自测**。单次运行产出 1 个 HTML，交付前先做脚本语法校验。

## 输入与输出

- **必填输入**：概念名（中文或英文均可，如“RAG / 向量数据库 / 大模型的上下文”）。
- **可选输入**：目标读者、篇幅、指定应用领域；缺省按“通用速学页”设计。
- **输出**：`learning-materials/<concept>.html`（单文件，风格统一）。若要总览多个概念，另输出 `learning-materials/concept-relationship.html`。
- **附带输出**：①若同概念已存在 Markdown 讲义（`learning-materials/<concept>.md`），必须同步补入**同一套自测题**，见 §8；②图解 SVG 写入 `learning-materials/assets/`，见 §5。不允许出现“HTML 有题、md 无题”的割裂状态——用户会两处都翻。

## 触发场景（示例）

- “快速学习一下 X 概念” / “3 分钟搞懂 X”
- “做一个介绍 X 的网页 / HTML 教程，要通俗、有图、有习题”
- “把刚才那种 skill 学习页改造成讲 Y 概念的”

## 工作流程

### 1. 澄清需求（仅当存在歧义）

概念含义多义时先确认范围，例如 “X 是指哪个领域/哪家产品的 X？目标读者是谁？”。其余情况直接开始，不要打断用户。

### 2. 源头检索（先查证，后动笔）

1. 用中英文各搜 1–2 轮，定位**一手或权威来源**（官方文档/规范站点/权威机构）。
2. 收集：官方定义原文（含出处）、关键数字/时间线/字段名、常见误区、易混淆相关概念。
3. 内容有任何拿不准的点：要么查证，要么在页面上明确标注“待确认”，**严禁编造**。
4. 概念若处于快速演进期（AI 类尤甚），务必先确认当前日期再检索，只采用当时可见的最新事实。
5. 记录“核对日期 = 今天”，写入页脚；页脚列出所用来源链接。

### 3. 编排内容（骨架见下，可按概念增删）

- **Step 0 一句话定义**：用公式/拆解式可视化点破本质（如 “A ＋ B ＝ 能力”）。
- **Step 1 打个比方**：2 个生活化类比卡片（新手入职手册、菜谱等），首选官方若用过的类比。
- **Step 2 解剖结构/要素**：目录树、分层卡片或字段表——讲“它由什么组成”。
- **Step 3 核心原理**：把最重要的机制画成步骤/层级图解（本技能示范的是“渐进式披露”式三级结构）。
- **Step 4 易混淆对比**：与相近概念的 3 行以内对照表。
- **Step 5 最小示例**：一段能看懂的极简代码/配置/实例。
- **Step 6 五行速记**：考前金句 3–5 条（可选）。
- **Step 7 分层自测**：见“测验引擎”。

**文案规范（沿用已确认的用户偏好）**
- 简体中文；**短句、低文字密度**；避免长篇段落与凑字数的铺垫。
- 术语首次出现中英对照；结论优先，理由从简。
- 全页**字号最小 12px**，禁止任何小于 12px 的文字（标签、徽章、角标同理）。

### 4. 页面实现（视觉强约束）

1. 单文件、尽量离线可用：样式与脚本全部内联；如需外链字体（如 Google Fonts），必须给出无需外网的本地回退，断网仍可读。
2. 视觉一律套用 **Anthropic/Claude 设计 token**，逐项照抄数值（色值/字号/行高/字距/留白/圆角），不要自由发挥——完整清单见 `references/design-tokens.md`。
3. 珊瑚主色只用于：CTA 按钮、整块引用卡、极少量眉标；其余一律走 25 色 token 内的中性/语义色。
4. 交互组件保持克制：无阴影依赖的“色块 + hairline 描边”层次即可。
5. 文件名用英文连字符式：`<concept>.html`，统一写入 `learning-materials/` 目录（与既有 agent.html / llm-context.html / skill.html 同目录，便于维护与索引）。

### 5. 图解配图（自包含 SVG）

抽象机制（循环、分层、对比、容量、关系）**能画就画**：一张图胜过一段文字。图解写成**自包含 SVG**，落到 `learning-materials/assets/`，再嵌进正文**对应章节内**——不要贴位图，不要全堆在页尾。

1. **命名**：`<概念>-<用途>.svg`，如 `agent-loop.svg`、`context-window.svg`、`workflow-vs-agent.svg`。
2. **自包含**：`viewBox="0 0 680 H"`（宽固定 680，高按内容自定）；色值**硬编码**取自设计 token，不依赖外部 CSS / 字体 / 图片——这样 GitHub 网页端能直接渲染缩略图，md 里也能引用。
3. **可访问**：根节点带 `role="img"` + `aria-labelledby`，内部 `<title>` / `<desc>` 说明图意。
4. **配色**：正文用到的语义色要与图解一致（珊瑚 `#cc785c` = 关键路径/主体，`#5db872` 绿、`#d4a017` 黄、`#c64545` 红 = 三档难度或状态，`#181715` 深色条 = 结论/边界）。
5. **先读正文再画图**：图解里的术语、层级、配色必须和正文完全对得上，否则图文互相打架。

**嵌入写法**

- **HTML**：在对应 `<section>` 内插 `figure.figimg`，并把这段样式随图一并加进页内 `<style>`：
  ```html
  <figure class="figimg">
    <img src="assets/agent-loop.svg" alt="…" loading="lazy">
    <figcaption>图 2 · 思考、行动、观察，循环回思考——这就是 Agent 与脚本的分水岭</figcaption>
  </figure>
  ```
  ```css
  .figimg{margin:var(--s-md) 0 0;padding:var(--s-md);background:var(--canvas);border:1px solid var(--hair);border-radius:var(--r-lg)}
  .figimg img{display:block;width:100%;max-width:820px;margin:0 auto;height:auto}
  .figimg figcaption{font-size:12px;line-height:1.5;color:var(--muted-soft);text-align:center;margin-top:var(--s-sm)}
  ```
- **Markdown**：`![图注](assets/x.svg)`；同时在 `## 目录` 下加一个「图解索引」小节，列出全部图与所在位置。
- **批量嵌入**：多页插图用一次性 Python 脚本，按 `re.search('<section id="…">')` 定位、`s.index('</section>', m.end())` 收尾插入，并**断言插入点唯一**防重复。

**画完必做三项校验**（XML 合法性 / 元素不越 viewBox / 中文文本不压框），脚本与坑见 `references/svg-diagrams.md` 与 §7。

### 6. 测验引擎（沿用现成 JS，见 references/quiz-engine.md）

- 题目 8–12 道，按难度分三层：入门（基础，须全对）→ 进阶（原理）→ 挑战（细节/辨析）。
- 三层分组标题 + 颜色区分；选完**即时判分并显示一句解析**。
- 顶层显示进度条与“已答/答对”，实时刷新“每层 已答/答对”分层统计。
- 交卷显示：分层成绩单（每层小进度条）+ 分层判定建议（哪层未达标就指向对应章节复习）+ 错题号列表。
- 达标线模板：入门全对 ＋ 进阶 ≥3 ＋ 挑战 ≥2（按题目总数等比调整）。
- 题目选项 ≤4 个；解析 ≤2 行；题干与解析语言同正文（中文）。
- **正确答案必须打散**：A/B/C/D 各占约 1/4，禁止集中在 A（用户会一路选 A 蒙分，测验就失效了）。生成后核对一次分布。
- 解析要先点出正确答案为什么对，再顺带说明**常见误选错在哪**；只写“A 是对的”等于没解析。

### 7. 校验（交付前必做）

1. **脚本语法**：用 Node 对 HTML 内所有 `<script>` 做纯语法检查（不执行 DOM）：
   ```bash
   node -e "const fs=require('fs');const s=fs.readFileSync('<file>','utf8');const m=[...s.matchAll(/<script>([\s\S]*?)<\/script>/g)];m.forEach((x,i)=>{try{new Function(x[1]);console.log('block',i,'OK')}catch(e){console.log('block',i,'ERR',e.message)}})"
   ```
   有 ERR 必须修复后重验。
2. **字号下限**：grep 确认不存在 `font-size: 0–11px`（含 10.5/11.5 等小于 12 的值）：
   ```bash
   grep -nE "font-size:\s*(0|([1-9]|1[01]))(\.\d+)?px" <file>   # 期望无输出
   ```
3. **无头真渲染**（关键一步，别只看源码）：用 jsdom 真正执行页面脚本，断言「题目渲染出来了、分层颜色在、点一下能判分」。源码看着没问题、DOM 里却是空白的情况很常见（脚本位置、id 拼错、样式类缺失）。脚本与判读方法见 `references/verify-render.md`；期望输出：每页 `.qbox` = 题目数、`.ghead .dot` 三个分层色值、点击首选项后 `.expl` 出现 `show` 类与解析文本。
4. **标签配平**：`div/section/ol/li/p` 开闭数量对齐（先把 `<script>`/`<style>` 内容屏蔽再数，否则会误报）。历史坑：`<div class="k">` 漏闭合会让后续章节 DOM 层级整体错位。
5. 检查所有脚本 `getElementById` 的 id 与 HTML 中一致；页面可在浏览器直接打开。
6. **图解三项校验**（新增/改动 SVG 时）：跑 `references/svg-diagrams.md` 里的校验脚本，断言 ①XML 合法 ②图形与文本都不越 `viewBox` ③中文文本按估算宽度不压框；再核对 HTML 的 `src="assets/*.svg"` 与 md 的 `![](assets/*.svg)` 路径**全部可解析**，且 `assets/` 里没有“画了却没引用”的孤儿图。
7. **产物一致性**：HTML 与 md 的题目数、选项数、答案分布、图解数量四处对齐后再交付。

### 8. Markdown 讲义同步（有 `<concept>.md` 时必做）

HTML 有样式能上色，`.md` 没有——但用户会两个都打开，所以 md 必须自带同一套题。

1. **题目来源只认 HTML**：直接执行 HTML 里的 `Q` 数组导出，**不要手抄**（手抄必然和网页版本漂移）。导出脚本见 `references/verify-render.md`。
2. **难度用色标代替颜色**：Markdown 不支持自定义颜色，用 🟢 入门 / 🟡 进阶 / 🔴 挑战 三个圆点做分层色标；若用户明确要真彩色，直接指向对应 HTML 页。
3. **结构固定为**：`## <序号>、自测(三层 · 12 题)` → 三层题面（题干加粗 + `- A./B./C./D.` 选项）→ `### 答案与解析`（`**N. 答案:X**` ＋ 解析正文）。
4. **顺带修目录**：往 `## 目录` 里补该节，并检查文末“参考资料/文献资料”等旧措辞与正文一致；**若已配图，同步在目录下补「图解索引」**（图名 → 所在小节）。
5. 改完校验：题目数 = 选项数/4 = 答案数，且答案分布不集中在 A。

### 9. 交付

- 用 present_files 打开生成的 HTML，让用户直接预览；同概念的 md 讲义一并列出。
- 一句话说明文件路径（`learning-materials/<concept>.html`）与“学完→自测”的路径。

## 资源

### references/design-tokens.md

Anthropic/Claude 设计 token 数值表（颜色/字体刻度/留白/圆角/组件），实现时逐项照抄，不要改写数值。

### references/quiz-engine.md

三层测验的完整 HTML 结构与可复制 JS 引擎（渲染、判分、分层统计、结果判定），含达标线与文案示例。

### references/verify-render.md

两个可复制脚本：**jsdom 真渲染校验**（断言题目数、三个分层色值、点击后解析是否弹出）与**从 HTML 的 `Q` 数组导出 Markdown 题库**（含插入讲义与收尾校验代码）。jsdom 的安装位置与 `NODE_PATH` 用法也在这里。

### references/svg-diagrams.md

自包含 SVG 图解的完整规范：画布与坐标约定、设计 token 取色对照、中文文本宽度估算表（画图时用来防溢出）、`text-anchor` 从父级 `<g>` 继承的坑，以及**三项校验的完整 Python 脚本**（XML 合法 / 不越 viewBox / 中文不压框）。

### assets/

配图产出目录。**当前 7 张样例已成图**，位于仓库的 `learning-materials/assets/`（技能本体不再重复存放，避免两份漂移）：

| 文件 | 图解模式 | 可复用于 |
|---|---|---|
| `agent-anatomy.svg` | 容器内多部件 | 任何“X 由哪几部分组成” |
| `agent-loop.svg` | 流程 + 回环箭头 | 任何循环机制（ReAct、反馈、迭代） |
| `workflow-vs-agent.svg` | 左右对比 + 结论条 | 任何 A/B 路线之争 |
| `context-window.svg` | 横向容量条 + 上限量线 | 任何“有限容量里装了什么” |
| `context-ushape.svg` | 曲线图（U 型 / 衰减） | 任何性能—位置/长度关系 |
| `skill-progressive.svg` | 三层嵌套 + 引线说明 | 任何分层加载 / 渐进披露 |
| `concept-map.svg` | 三框 + 读写箭头 | 任何多概念关系总览 |

套新概念时：**复制 → 改文字 → 换色（仍取 token）→ 跑 §7 的三项校验**。SVG 写法与校验规则见 `references/svg-diagrams.md`。
