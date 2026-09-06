# Anthropic / Claude 设计 token（数值照抄，勿改）

> 来源：社区整理的 Claude DESIGN.md（claude.com 观测汇总，shadcn / ExplainX / aiskill.market，2026-09-03 检索）。
> 使用原则：奶油底 + 暖墨 + 单一珊瑚点缀。不要用纯 #fff / #000 铺大面积；不要引入第三个强调色。

## 颜色（25 色）

```
--canvas:#faf9f5; --soft:#f5f0e8; --card:#efe9de; --strong:#e8e0d2;
--dark:#181715; --dark-elev:#252320; --dark-soft:#1f1e1b;
--ink:#141413; --body:#3d3d3a; --body-strong:#252523; --muted:#6c6a64; --muted-soft:#8e8b82;
--on-primary:#ffffff; --on-dark:#faf9f5; --on-dark-soft:#a09d96;
--hair:#e6dfd8; --hair-soft:#ebe6df;
--primary:#cc785c; --primary-active:#a9583e; --primary-disabled:#e6dfd8;
--teal:#5db8a6; --amber:#e8a55a;
--success:#5db872; --warning:#d4a017; --error:#c64545;
```

珊瑚 `--primary` 只用于：CTA 按钮、整块引用卡、极少量眉标/章节号。难度色用语义 token（success/warning/error），不要占用珊瑚。

## 字体刻度（15 级，px / 行高 / 字距 / 字重）

| 样式 | 字体 | size | lh | ls | w |
|---|---|---|---|---|---|
| display-xl | serif | 64 | 1.05 | -1.5px | 400 |
| display-lg | serif | 48 | 1.1 | -1px | 400 |
| display-md | serif | 36 | 1.15 | -0.5px | 400 |
| display-sm | serif | 28 | 1.2 | -0.3px | 400 |
| title-lg | sans | 22 | 1.3 | 0 | 500 |
| title-md | sans | 18 | 1.4 | 0 | 500 |
| title-sm | sans | 16 | 1.4 | 0 | 500 |
| body-md | sans | 16 | 1.55 | 0 | 400 |
| body-sm | sans | 14 | 1.55 | 0 | 400 |
| caption | sans | 13 | 1.4 | 0 | 500 |
| caption-uppercase | sans | 12 | 1.4 | +1.5px | 500 |
| code | mono | 14 | 1.6 | 0 | 400 |
| button | sans | 14 | 1 | 0 | 500 |
| nav-link | sans | 14 | 1.4 | 0 | 500 |

响应式阶梯：按 token 步进降级（display-xl→lg→md），勿用任意 clamp。

## 留白（4px 基）

```
xxs4 · xs8 · sm12 · md16 · lg24 · xl32 · xxl48 · section96
```
- 章节（band）之间留白统一 **96px**。
- 特性卡内边距 **32px**；代码窗/小卡 **24px**；整块珊瑚引用卡 **48px**。
- 主内容容器 max-width 1200px 居中；正文可收窄提高可读性。

## 圆角

```
xs4 · sm6 · md8 · lg12 · xl16 · pill9999
```
按钮/输入 md8；内容卡/代码窗 lg12；hero/整块引用/成绩卡 xl16；标签 pill。

## 组件要点

- 主按钮：`background:var(--primary); color:var(--on-primary); radius:8px; padding:12px 20px; height:40px; font:button`；hover `--primary-active`。
- 描边：一律 1px `--hair`/`--hair-soft`，禁止花哨阴影（色块+描边做层级）。
- 三种表面节奏：奶油画布 → 奶油卡 `--card` → 深色产品面板 `--dark`（放代码/目录/示例）。
- 字体替代（官方建议）：Copernicus ≈ Cormorant Garamond 500；StyreneB ≈ Inter。
  引入方式（可离线回退）：
  ```html
  <link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,500;0,600;1,500&family=Inter:wght@400;500&display=swap" rel="stylesheet">
  ```
- **字号下限：全页最小 12px**（含标签/徽章/角标）。

## 可直接粘贴的 :root 参考

```css
:root{
  --canvas:#faf9f5;--soft:#f5f0e8;--card:#efe9de;--strong:#e8e0d2;
  --dark:#181715;--dark-elev:#252320;--dark-soft:#1f1e1b;
  --ink:#141413;--body:#3d3d3a;--body-strong:#252523;--muted:#6c6a64;--muted-soft:#8e8b82;
  --on-primary:#ffffff;--on-dark:#faf9f5;--on-dark-soft:#a09d96;
  --hair:#e6dfd8;--hair-soft:#ebe6df;
  --primary:#cc785c;--primary-active:#a9583e;--primary-disabled:#e6dfd8;
  --teal:#5db8a6;--amber:#e8a55a;--success:#5db872;--warning:#d4a017;--error:#c64545;
  --r-xs:4px;--r-sm:6px;--r-md:8px;--r-lg:12px;--r-xl:16px;--r-pill:9999px;
  --s-xxs:4px;--s-xs:8px;--s-sm:12px;--s-md:16px;--s-lg:24px;--s-xl:32px;--s-xxl:48px;--s-section:96px;
  --font-display:"Cormorant Garamond",Georgia,"Noto Serif SC","Songti SC",SimSun,serif;
  --font-sans:"Inter","Styrene B",-apple-system,BlinkMacSystemFont,"Segoe UI","PingFang SC","Hiragino Sans GB","Microsoft YaHei",sans-serif;
  --font-mono:"JetBrains Mono","Berkeley Mono",ui-monospace,Consolas,"Cascadia Code",monospace;
}
```
