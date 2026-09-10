# svg-diagrams.md — 自包含 SVG 图解规范与校验

图解是学习页里最容易被跳过、也最容易被做坏的一环。本文给出**能直接复制**的画法约定与交付前必跑的三项校验。

---

## 一、为什么用自包含 SVG

| 方案 | 问题 |
|---|---|
| 位图（PNG/JPG） | 放大糊；改字要重画；GitHub 上要额外传文件 |
| 引用外部 CSS / 字体的 SVG | 脱离本页就掉样式——md、GitHub 预览里全变黑框 |
| **自包含 SVG（本技能采用）** | 矢量不失真、色值硬编码、能嵌 HTML 也能嵌 md、GitHub 网页端直接出缩略图 |

代价是**不能用 CSS 变量**，所有颜色必须写成字面量。这不是缺陷，是换取可移植性的必要代价。

---

## 二、画布与坐标约定

- 统一 `viewBox="0 0 680 H"`：**宽固定 680**，高 `H` 按内容自定（常见 240–440）。固定宽度让多张图在页面上宽度一致、不跳版。
- 根节点必带无障碍属性：

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 680 358" width="100%"
     role="img" aria-labelledby="anatT anatD">
  <title id="anatT">Agent 的五个部件</title>
  <desc id="anatD">Agent 运行时由大脑 LLM、规划、记忆、工具四个部件组成，外面套一层护栏。</desc>
  <defs>
    <marker id="ag" viewBox="0 0 10 10" refX="8" refY="5"
            markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#8e8b82" stroke-width="1.5"
            stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>
  <g font-family="Inter, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif">
    …
  </g>
</svg>
```

要点：

1. **字体写一整条回退链**，放在最外层 `<g>` 上一次即可（会被继承）。
2. **箭头用 `<marker>`**，在 `<defs>` 里定义一次、`marker-end="url(#ag)"` 复用；`orient="auto-start-reverse"` 省得为反向箭头再定义一个。
3. **id 加图名前缀**（`anatT` / `ag`），避免同页多图时 id 冲突。
4. 圆角矩形 `rx="10"`（块）/ `rx="20"`（外框）；描边统一 `stroke-width="0.5"` 的 hairline 风格。

### 设计 token 取色（硬编码值）

| 用途 | 色值 |
|---|---|
| 正文/标题 | `#141413` `#3d3d3a` |
| 次要说明 | `#6c6a64` `#a09d96` |
| 珊瑚（主体/关键路径） | 底 `#f9e7df`、描边 `#cc785c`、字 `#8f4430` |
| 中性卡片 | `#efe9de` / 描边 `#d8cdba` |
| 浅底 | `#f5f0e8` / 描边 `#e6dfd8` |
| 深色条（结论/边界） | `#181715` 底 + `#faf9f5` 字 |
| 三档难度 | 绿 `#5db872`、黄 `#d4a017`、红 `#c64545` |
| 箭头/连接线 | `#8e8b82` |

---

## 三、中文文本宽度估算（画图时防溢出）

SVG 没有自动换行，**文字超框只能靠算**。经验系数（相对 `font-size`）：

| 字符 | 系数 |
|---|---|
| CJK 汉字、全角标点 | 1.00 |
| ASCII 字母、数字、半角标点 | 0.56 |
| 空格、`·`、`—`、`→`、`←`、`|` | 0.45 |

判断方式：`文字宽度 = Σ(系数 × font-size)`，再按 `text-anchor` 折算左右边界。

> **实测经验**：12px 下一行中文超过 **约 46 个字**（680 宽画布、左右各留 80 边距）就该换行了。宁可拆两行 `<text>`，也不要压框。

### ⚠️ 最容易踩的坑：`text-anchor` 会继承

`text-anchor` 定义在 `<g>` 上时，**子 `<text>` 会继承**。写检测脚本时若只看 `<text>` 自身的属性、默认成 `start`，就会把居中的文字按左对齐算宽，**误报一堆“压框”**。

```python
def walk(e, inh_anchor='start'):
    anchor = e.get('text-anchor', inh_anchor)   # ← 关键：向下传递
    if e.tag == NS + 'text':
        ...
    for c in e:
        walk(c, anchor)
```

第一次写这个脚本时，正是漏了继承，7 张图里误报 3 处（`context-window.svg` 的“工具与返回 / 本次输入 / 现场指令”三条居中标签）。修正后全部通过。

---

## 四、交付前必跑：三项校验

```python
# -*- coding: utf-8 -*-
"""svg_check.py — 自包含 SVG 图解三项校验
用法: python svg_check.py <svg 文件或目录> [...]
① XML 合法性  ② 图形/文本不越 viewBox  ③ 中文文本按估算宽度不压框
"""
import sys, os, glob
import xml.etree.ElementTree as ET

NS = '{http://www.w3.org/2000/svg}'
TOL = 0.5          # 容差：hairline 描边允许压线

def flen(s, fs):
    """按字符类型估算文本宽度"""
    w = 0.0
    for ch in s:
        if ord(ch) > 0x2E7F:              # CJK / 全角
            w += 1.00 * fs
        elif ch in ' ·—–→←|':
            w += 0.45 * fs
        else:
            w += 0.56 * fs
    return w

def num(v, d=0.0):
    try:
        return float(v)
    except (TypeError, ValueError):
        return d

def check(path):
    errs = []
    try:
        root = ET.parse(path).getroot()
    except ET.ParseError as e:
        return ['XML 解析失败: %s' % e]

    vb = root.get('viewBox')
    if not vb:
        return ['缺少 viewBox']
    W, H = [float(x) for x in vb.split()][2:4]

    def walk(e, inh_anchor='start'):
        anchor = e.get('text-anchor', inh_anchor)     # 继承！别漏
        tag = e.tag

        if tag == NS + 'text':
            fs = num(e.get('font-size'), 13)
            x, y = num(e.get('x')), num(e.get('y'))
            s = ''.join(e.itertext()).strip()
            if s:
                tw = flen(s, fs)
                if anchor == 'middle':
                    x0, x1 = x - tw / 2, x + tw / 2
                elif anchor == 'end':
                    x0, x1 = x - tw, x
                else:
                    x0, x1 = x, x + tw
                if x0 < -TOL or x1 > W + TOL:
                    errs.append('文本横向压框 [%.0f,%.0f] > 画布宽 %d: %r'
                                % (x0, x1, W, s[:24]))
                if y - fs < -TOL or y > H + TOL:
                    errs.append('文本纵向越界 y=%.0f: %r' % (y, s[:24]))

        elif tag == NS + 'rect':
            x, y = num(e.get('x')), num(e.get('y'))
            w, h = num(e.get('width')), num(e.get('height'))
            if x < -TOL or y < -TOL or x + w > W + TOL or y + h > H + TOL:
                errs.append('rect 越界 (%.0f,%.0f,%.0f,%.0f)' % (x, y, w, h))

        elif tag == NS + 'circle':
            cx, cy, r = num(e.get('cx')), num(e.get('cy')), num(e.get('r'))
            if cx - r < -TOL or cy - r < -TOL or cx + r > W + TOL or cy + r > H + TOL:
                errs.append('circle 越界 (%s,%s,r=%s)' % (e.get('cx'), e.get('cy'), e.get('r')))

        for c in e:
            walk(c, anchor)

    walk(root)
    return errs

targets = []
for a in sys.argv[1:] or ['learning-materials/assets']:
    targets += sorted(glob.glob(os.path.join(a, '*.svg'))) if os.path.isdir(a) else [a]

total = 0
for p in targets:
    errs = check(p)
    total += len(errs)
    print(('  OK  ' if not errs else ' FAIL ') + os.path.basename(p))
    for m in errs:
        print('        - ' + m)
print('\n共 %d 张图，问题 %d 处' % (len(targets), total))
sys.exit(1 if total else 0)
```

**判读**：输出 `共 7 张图，问题 0 处` 才算过。`<path>` 的曲线不参与越界判断（端点难解析），所以**画箭头/曲线时靠留白自觉**，别贴着边画。

**校验之外还要核对引用**：HTML 里的 `src="assets/*.svg"` 与 md 里的 `![](assets/*.svg)` 路径必须逐一存在，且目录里不能有画了却没被任何页面引用的“孤儿图”。

```python
import io, re, os
root = 'learning-materials'
used = set()
for f in ['agent.html', 'llm-context.html', 'skill.html', 'concept-relationship.html']:
    s = io.open(os.path.join(root, f), encoding='utf-8').read()
    used |= set(re.findall(r'assets/[\w.-]+\.svg', s))
for f in os.listdir(os.path.join(root, 'assets')):
    if f.endswith('.svg'):
        p = 'assets/' + f
        print(('  引用 OK ' if p in used else '  未引用!') + p)
```

---

## 五、嵌入写法

### HTML（插在对应 `<section>` 内，不要堆页尾）

```html
<figure class="figimg">
  <img src="assets/agent-anatomy.svg" alt="Agent 的五个部件" loading="lazy">
  <figcaption>图 1 · 五部件：大脑负责想，规划负责拆，记忆负责记，工具负责做，护栏兜底</figcaption>
</figure>
```

```css
.figimg{margin:var(--s-md) 0 0;padding:var(--s-md);background:var(--canvas);border:1px solid var(--hair);border-radius:var(--r-lg)}
.figimg img{display:block;width:100%;max-width:820px;margin:0 auto;height:auto}
.figimg figcaption{font-size:12px;line-height:1.5;color:var(--muted-soft);text-align:center;margin-top:var(--s-sm)}
```

### Markdown

```markdown
![Agent 的五个部件](assets/agent-anatomy.svg)

*图 1 · 五部件：大脑负责想，规划负责拆，记忆负责记，工具负责做，护栏兜底*
```

并在 `## 目录` 下补「图解索引」，方便按图找章节。

### 批量嵌入（多页多图时）

按 `<section id="…">` 定位、`</section>` 收尾插入，**插入前断言锚点唯一且未插过**（幂等），避免重跑脚本产生重复图：

```python
import io, re
s = io.open(path, encoding='utf-8').read()
m = re.search(r'<section id="%s">' % sec, s)
assert m, '找不到章节 ' + sec
end = s.index('</section>', m.end())
assert 'assets/%s' % svg not in s, '已插过，跳过'
s = s[:end] + block + '\n' + s[end:]
io.open(path, 'w', encoding='utf-8').write(s)
```
