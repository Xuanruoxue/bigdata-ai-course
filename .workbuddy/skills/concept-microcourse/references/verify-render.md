# verify-render.md — 无头校验与 md 题库导出

两个可复制脚本：**A) 真渲染校验**（确认题目/颜色真的出来了）、**B) 从 HTML 导出 md 题库**（保证讲义与网页同源）。

---

## 前置：jsdom（一次性）

jsdom 装在托管 Node 工作区，不要装到全局、不要装进用户项目：

```bash
mkdir -p "C:/Users/80696/.workbuddy/binaries/node/workspace"
cd "C:/Users/80696/.workbuddy/binaries/node/workspace"
"C:/Users/80696/.workbuddy/binaries/node/versions/22.22.2/node.exe" \
  "C:/Users/80696/.workbuddy/binaries/node/versions/22.22.2/node_modules/npm/bin/npm-cli.js" \
  install jsdom --no-audit --no-fund
```

运行任何脚本时都要带上 `NODE_PATH`：

```bash
cd "C:/Users/80696/.workbuddy/binaries/node/workspace"
NODE_PATH="C:/Users/80696/.workbuddy/binaries/node/workspace/node_modules" \
  "C:/Users/80696/.workbuddy/binaries/node/versions/22.22.2/node.exe" check_render.js
```

---

## A) check_render.js — 真渲染校验

为什么必须做：源码层面（语法、样式类、id）全部检查通过，页面仍可能空白。只有让脚本真的在 DOM 上跑一遍才能确认。

```javascript
const fs = require('fs');
const path = require('path');
const { JSDOM } = require('jsdom');

const dir = 'C:/Users/80696/Desktop/bigdata-ai-course/learning-materials';
const files = ['agent.html', 'llm-context.html', 'skill.html', 'concept-relationship.html'];

for (const f of files) {
  const html = fs.readFileSync(path.join(dir, f), 'utf8');
  const dom = new JSDOM(html, { runScripts: 'dangerously', pretendToBeVisual: true });
  const d = dom.window.document;

  const boxes = d.querySelectorAll('.qbox');
  const heads = [...d.querySelectorAll('.ghead')].map(h => h.textContent.replace(/\s+/g, ' ').trim());
  const lpill = d.getElementById('lpill');
  const colors = [...((lpill || {}).innerHTML || '').matchAll(/color:([^"';]+)/g)].map(m => m[1]);
  const dotColors = [...d.querySelectorAll('.ghead .dot')].map(e => e.getAttribute('style'));

  const firstOpt = d.querySelector('.qbox .opt');
  let clickMsg = '(未点击)';
  if (firstOpt) {
    firstOpt.dispatchEvent(new dom.window.MouseEvent('click', { bubbles: true }));
    const expl = d.querySelector('.qbox .expl');
    clickMsg = (expl ? expl.textContent.slice(0, 60) : '(无解析)') + ' | 类=' + (expl ? expl.className : '-');
  }

  console.log('=== ' + f + ' ===');
  console.log('  题目数(.qbox):', boxes.length);
  console.log('  分层标题:', heads.join(' | '));
  console.log('  分层颜色:', colors.join(', ') || '(空)');
  console.log('  色点内联样式:', dotColors.join(' , '));
  console.log('  点击首题首选项 ->', clickMsg);
  console.log();
}
```

**判读标准**

| 项 | 期望 |
|---|---|
| `.qbox` 数量 | 等于该页题目数（现为 12） |
| `.ghead` | 3 条，分别带「入门 / 进阶 / 挑战」与 `N 题` |
| 分层颜色 | 三个色值齐全，顺序为 `#5db872`(绿) → `#d4a017`(黄) → `#c64545`(红) |
| `.ghead .dot` | 每条都有 `background:#...` 内联样式 |
| 点击首选项 | `.expl` 文本以「✓ 答对了。」或「✗ 答错了，正确答案是 X。」开头，且 class 含 `show` 与 `ok`/`no` |

任一项为空 → 别急着交，先查：`<script>` 是否在 `</body>` 之前、`getElementById` 的 id 是否拼错、`.qbox/.opt/.expl` 等类在 `<style>` 里是否存在。

---

## B) md_quiz.js — 从 HTML 导出 md 题库

直接执行页面里的 `Q` 数组，避免手抄导致的图文不一致。

```javascript
const fs = require('fs');
const { JSDOM } = require('jsdom');

const html = fs.readFileSync('C:/Users/80696/Desktop/bigdata-ai-course/learning-materials/agent.html', 'utf8');
const dom = new JSDOM(html, { runScripts: 'dangerously' });
const w = dom.window;
const Q = w.eval('Q');
const LAYERS = w.eval('LAYERS');
const LCNT = w.eval('LCNT');
const letters = ['A', 'B', 'C', 'D'];

const tierMark = ['🟢', '🟡', '🔴'];
const tierWord = ['入门', '进阶', '挑战'];
const tierNeed = ['基础 · 需全对', '原理与实战 · 达标 ≥3', '细节辨析 · 达标 ≥2'];

let md = '## 六、自测(三层 · 12 题)\n\n';
md += '> **难度分三层**:🟢 入门(基础)　→　🟡 进阶(原理与实战)　→　🔴 挑战(细节辨析)。\n';
md += '> **达标线**:入门 4/4 + 进阶 ≥3 + 挑战 ≥2。\n';
md += '> **用法**:先自己选,选完再对照文末「答案与解析」。\n\n';

let g = 0;
for (let l = 0; l < LAYERS.length; l++) {
  const items = Q.filter(q => q.L === l);
  if (!items.length) continue;
  md += '### ' + tierMark[l] + ' ' + tierWord[l] + '(' + LCNT[l] + ' 题 · ' + tierNeed[l] + ')\n\n';
  for (const it of items) {
    g++;
    md += '**' + g + '. ' + it.q + '**\n\n';
    it.o.forEach((op, i) => { md += '- ' + letters[i] + '. ' + op + '\n'; });
    md += '\n';
  }
  md += '---\n\n';
}

md += '### 答案与解析\n\n';
g = 0;
for (let l = 0; l < LAYERS.length; l++) {
  const items = Q.filter(q => q.L === l);
  if (!items.length) continue;
  md += '#### ' + tierMark[l] + ' ' + tierWord[l] + '\n\n';
  for (const it of items) {
    g++;
    md += '**' + g + '. 答案:' + letters[it.a] + '**\n\n' + it.e + '\n\n';
  }
}
md += '---\n\n';

fs.writeFileSync('C:/Users/80696/AppData/Local/Temp/quiz_section.md', md, 'utf8');
console.log('生成成功,字符数:', md.length, '| 题目数:', g);
```

**落盘到讲义**（用 Python 精确插入，避免手改出错）：

```python
import io
p = 'learning-materials/agent.md'
s = io.open(p, encoding='utf-8').read()
q = io.open('C:/Users/80696/AppData/Local/Temp/quiz_section.md', encoding='utf-8').read()
anchor = '## 附:快速记忆卡'
assert s.count(anchor) == 1 and '## 六、自测' not in s   # 幂等：防止重复插入
s = s.replace(anchor, q.rstrip() + '\n\n' + anchor)
io.open(p, 'w', encoding='utf-8').write(s)
```

**收尾校验**（题目 / 选项 / 答案 / 分布）：

```python
import io, re
s = io.open('learning-materials/agent.md', encoding='utf-8').read()
print('题目:', len(re.findall(r'^\*\*\d+\. .*？\*\*$', s, re.M)))
print('选项:', len(re.findall(r'^- [ABCD]\. ', s, re.M)))          # 应为题目数 × 4
print('答案:', len(re.findall(r'^\*\*\d+\. 答案:[ABCD]\*\*$', s, re.M)))
print('分布:', {k: len(re.findall('答案:' + k + r'\*\*', s)) for k in 'ABCD'})  # 不应集中在 A
```
