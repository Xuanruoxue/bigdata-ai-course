# 测验引擎（三层难度 · 即时判分）

把以下 HTML 结构与 JS 原样嵌入生成的页面，仅需替换 `Q` 数组与文案。样式类名与 design-tokens 配套。

## 页面 HTML（放在测验 <section> 内）

```html
<p class="sub">（难度说明与达标线文案，如：入门 4/4 ＋ 进阶 ≥3 ＋ 挑战 ≥2）</p>
<div class="qbar"><i id="qbarFill"></i></div>
<div class="qstat"><span id="qDone">已答 0 / N</span><span id="qScore">答对 0 题</span></div>
<div class="glayer" id="lpill"></div>
<div id="quizRoot"></div>
<div class="result" id="resultBox">
  <p class="sc" id="resScore"></p>
  <div class="gd" id="resGrade"></div>
  <div class="ms" id="resMsg"></div>
  <div class="rtiers" id="resTiers"></div>
  <div class="wl" id="resWrong"></div>
  <div style="margin-top:var(--s-md)"><button class="btn" onclick="restartQuiz()">↻ 再来一次</button></div>
</div>
```

## JS 引擎（直接复制；数字 12/4/3/2 按实际题数与达标线调整）

```js
/* 三层难度配色：一律用语义色 success / warning / error */
const LAYERS=[
  {name:'入门',color:'#5db872'},
  {name:'进阶',color:'#d4a017'},
  {name:'挑战',color:'#c64545'}
];
const Q=[
  /* ---- 第一层 · 入门（基础概念，应全对）---- */
  {L:0,q:'题干？',o:['选项 A','选项 B','选项 C','选项 D'],a:0,e:'一句解析。'},
  /* ---- 第二层 · 进阶（原理与实战）---- */
  {L:1,q:'题干？',o:['选项 A','选项 B','选项 C','选项 D'],a:1,e:'一句解析。'},
  /* ---- 第三层 · 挑战（细节与辨析）---- */
  {L:2,q:'题干？',o:['选项 A','选项 B','选项 C','选项 D'],a:2,e:'一句解析。'}
];
const LCNT=[4,4,4];          /* 每层题数，与 Q 中 L 计数一致 */
const TOTAL=12;              /* 总题数 */
const heads=['基础 · 先懂“是什么”，需全对','原理与实战 · 怎么运行、怎么写','细节辨析 · 易混点与规范'];

let answered=0,correct=0,layerA=[0,0,0],layerC=[0,0,0];
const letters=['A','B','C','D'];

function paint(){
  document.getElementById('qbarFill').style.width=(answered/TOTAL*100)+'%';
  document.getElementById('qDone').textContent='已答 '+answered+' / '+TOTAL;
  document.getElementById('qScore').textContent='答对 '+correct+' 题';
  document.getElementById('lpill').innerHTML=LAYERS.map((l,i)=>
    '<span style="color:'+l.color+'">● '+l.name+' '+layerA[i]+'/'+LCNT[i]+'</span>').join('　');
}
function render(){
  const root=document.getElementById('quizRoot');
  root.innerHTML='';let g=0;
  LAYERS.forEach((ly,l)=>{
    const items=Q.filter(q=>q.L===l); if(!items.length)return;
    const h=document.createElement('div');h.className='ghead';
    h.innerHTML='<span class="dot" style="background:'+ly.color+'"></span>'+ly.name+
      '<span class="cnt" style="background:'+ly.color+'">'+LCNT[l]+' 题</span><span class="tip">'+heads[l]+'</span>';
    root.appendChild(h);
    items.forEach(it=>{
      g++;
      const box=document.createElement('div');box.className='qbox';
      box.innerHTML='<div class="qt">'+g+'. '+it.q+'</div><div class="qm">'+ly.name+' · 单选 · '+g+'/'+TOTAL+'</div>'+
        it.o.map((op,i)=>'<div class="opt" data-i="'+i+'"><span class="key">'+letters[i]+'</span><span>'+op+'</span></div>').join('')+
        '<div class="expl"></div>';
      root.appendChild(box);
      box.querySelectorAll('.opt').forEach(el=>el.onclick=()=>{
        if(box.dataset.done)return;
        box.dataset.done='1';
        const pick=+el.dataset.i;answered++;layerA[l]++;
        const right=pick===it.a;
        if(right){correct++;layerC[l]++;el.classList.add('r');}
        else{el.classList.add('w');box.querySelectorAll('.opt')[it.a].classList.add('r');}
        box.querySelectorAll('.opt').forEach(x=>{if(x!==el&&!(right&&x===el))x.style.opacity=.45;});
        const ex=box.querySelector('.expl');
        ex.classList.add('show',right?'ok':'no');
        ex.innerHTML=(right?'✓ 对':'✗ 答案是 '+letters[it.a])+'：'+it.e;
        paint();if(answered===TOTAL)showResult();
      });
    });
  });
}
function showResult(){
  const box=document.getElementById('resultBox');box.classList.add('show');
  document.getElementById('resScore').textContent=correct+' / '+TOTAL;
  document.getElementById('resTiers').innerHTML=LAYERS.map((l,i)=>
    '<div class="rt"><span class="nm" style="color:'+l.color+'">'+l.name+'</span>'+
    '<span class="bar"><i style="width:'+(layerC[i]/LCNT[i]*100)+'%;background:'+l.color+'"></i></span>'+
    '<span class="nu">'+layerC[i]+' / '+LCNT[i]+'</span></div>').join('');
  const eOk=layerC[0]===LCNT[0],mOk=layerC[1]>=3,hOk=layerC[2]>=2;
  let gd,ms;
  if(correct===TOTAL){gd='满分';ms='三层全通，可以直接实战了。';}
  else if(eOk&&mOk&&hOk){gd='达标通关';ms='基础、原理、细节都过关。';}
  else if(!eOk){gd='先补基础';ms='回到「定义 / 结构」两节再看一遍。';}
  else if(!mOk){gd='进阶层没过';ms='重点复习「核心原理」一节。';}
  else{gd='差在细节';ms='重点看「易混淆对比」与字段/细节规范。';}
  document.getElementById('resGrade').textContent=gd;
  document.getElementById('resMsg').textContent=ms;
  const wrong=[];
  document.querySelectorAll('.qbox').forEach((b,i)=>{if(b.querySelector('.opt.w'))wrong.push(i+1);});
  document.getElementById('resWrong').textContent=wrong.length?'待复习：第 '+wrong.join('、')+' 题':'';
  box.scrollIntoView({behavior:'smooth',block:'center'});
}
function restartQuiz(){
  answered=0;correct=0;layerA.fill(0);layerC.fill(0);
  document.getElementById('resultBox').classList.remove('show');
  paint();render();
  document.getElementById('quiz').scrollIntoView({behavior:'smooth'});
}
render();paint();
```

## 配套 CSS（配合 design-tokens.md 的 :root）

- `.qbar`：6px 高、底色 `--hair`、进度条填充 `--primary`。
- `.qstat/.glayer`：12–13px、`--muted`。
- `.ghead`：`--ink` 16px/500；`.dot` 9px 圆点（层级色）；`.cnt` 圆角 pill（层级色底、白字）。
- `.qbox`：`background:var(--soft)` + `1px var(--hair-soft)` 描边、radius lg12、内边距 lg24。
- `.opt`：`background:var(--canvas)`、radius md8；`.r` 用 success 边+浅绿底、`.w` 用 error 边+浅红底；`.sel` 用 `--strong` 底 + `--ink` 边（键盘态可选）。
- `.expl.ok/.no`：success/error 低透明度底色。
- `.result`：`--card` 底、radius xl16、内边距 xxl48。

## 注意

- 题干/选项/解析全中文，与正文语言一致。
- 字号下限 12px：`.qm/.cnt/.nm` 等一律 ≥12px。
- 解析 ≤2 行；每题选项 ≤4。
