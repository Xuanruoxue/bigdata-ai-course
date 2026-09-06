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

### 5. 测验引擎（沿用现成 JS，见 references/quiz-engine.md）

- 题目 8–12 道，按难度分三层：入门（基础，须全对）→ 进阶（原理）→ 挑战（细节/辨析）。
- 三层分组标题 + 颜色区分；选完**即时判分并显示一句解析**。
- 顶层显示进度条与“已答/答对”，实时刷新“每层 已答/答对”分层统计。
- 交卷显示：分层成绩单（每层小进度条）+ 分层判定建议（哪层未达标就指向对应章节复习）+ 错题号列表。
- 达标线模板：入门全对 ＋ 进阶 ≥3 ＋ 挑战 ≥2（按题目总数等比调整）。
- 题目选项 ≤4 个；解析 ≤2 行；题干与解析语言同正文（中文）。

### 6. 校验（交付前必做）

1. **脚本语法**：用 Node 对 HTML 内所有 `<script>` 做纯语法检查（不执行 DOM）：
   ```bash
   node -e "const fs=require('fs');const s=fs.readFileSync('<file>','utf8');const m=[...s.matchAll(/<script>([\s\S]*?)<\/script>/g)];m.forEach((x,i)=>{try{new Function(x[1]);console.log('block',i,'OK')}catch(e){console.log('block',i,'ERR',e.message)}})"
   ```
   有 ERR 必须修复后重验。
2. **字号下限**：grep 确认不存在 `font-size: 0–11px`（含 10.5/11.5 等小于 12 的值）：
   ```bash
   grep -nE "font-size:\s*(0|([1-9]|1[01]))(\.\d+)?px" <file>   # 期望无输出
   ```
3. 检查所有脚本 `getElementById` 的 id 与 HTML 中一致；页面可在浏览器直接打开。

### 7. 交付

- 用 present_files 打开生成的 HTML，让用户直接预览。
- 一句话说明文件路径（`learning-materials/<concept>.html`）与“学完→自测”的路径。

## 资源

### references/design-tokens.md

Anthropic/Claude 设计 token 数值表（颜色/字体刻度/留白/圆角/组件），实现时逐项照抄，不要改写数值。

### references/quiz-engine.md

三层测验的完整 HTML 结构与可复制 JS 引擎（渲染、判分、分层统计、结果判定），含达标线与文案示例。

### assets/

（当前为空；如沉淀出通用模板可放 `assets/template.html`。）
