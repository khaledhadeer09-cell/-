<!doctype html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>آلة حاسبة بسيطة</title>
<style>
  :root{
    --bg:#0b1220; --panel:#0f1724; --key:#0b1220; --accent:#7ee3b6; --muted:#98aec2; --text:#e6eef8;
    --glass: rgba(255,255,255,0.03);
  }
  *{box-sizing:border-box;font-family:Inter, 'Cairo', system-ui, sans-serif}
  html,body{height:100%;margin:0;background:linear-gradient(180deg,#041020,#071428);color:var(--text);display:flex;align-items:center;justify-content:center;padding:18px}
  .calc {
    width:100%; max-width:420px; border-radius:18px; overflow:hidden; box-shadow:0 14px 40px rgba(2,6,23,0.7);
    background:linear-gradient(180deg,var(--panel),#061222);
  }
  .top {display:flex;align-items:center;justify-content:space-between;padding:12px 14px;background:linear-gradient(90deg,rgba(255,255,255,0.02),transparent)}
  .brand{font-weight:800;font-size:16px}
  .small-btn{background:transparent;border:0;color:var(--muted);padding:6px 8px;border-radius:8px;cursor:pointer;font-weight:700}
  .display {
    padding:16px 14px 10px; text-align:left; background:linear-gradient(180deg,rgba(255,255,255,0.01),transparent);
  }
  .expr {font-size:14px;color:var(--muted);min-height:20px;word-break:break-all}
  .result {font-size:34px;font-weight:800;margin-top:6px;word-break:break-all}
  .keys {padding:12px;display:grid;grid-template-columns:repeat(4,1fr);gap:10px;background:linear-gradient(180deg,transparent, rgba(0,0,0,0.06));}
  button.key{height:56px;border-radius:10px;border:0;background:var(--key);color:var(--text);font-size:18px;font-weight:700;cursor:pointer;box-shadow:0 6px 18px rgba(2,6,23,0.6)}
  button.key.op{background:linear-gradient(180deg,#13303a,#0f2228);color:var(--accent)}
  button.key.accent{background:linear-gradient(180deg,#2a5b50,#1b4c3f)}
  .key:active{transform:translateY(2px);box-shadow:none}
  .wide{grid-column:span 2}
  .history {max-height:160px;overflow:auto;padding:8px 12px;border-top:1px solid rgba(255,255,255,0.02);font-size:13px;color:var(--muted);background:linear-gradient(180deg,rgba(255,255,255,0.01),transparent)}
  .history .item{padding:8px;border-radius:8px;margin-bottom:8px;background:var(--glass);display:flex;justify-content:space-between;gap:8px;align-items:center}
  .history .item small{color:var(--muted);font-size:12px}
  /* responsive */
  @media (max-width:380px){ .result{font-size:28px} .keys button{height:52px;font-size:16px} }
</style>
</head>
<body>
  <div class="calc" role="application" aria-label="آلة حاسبة">
    <div class="top">
      <div class="brand">آلة حاسبة — سما</div>
      <div>
        <button class="small-btn" id="btnCopy" title="نسخ النتيجة">نسخ</button>
        <button class="small-btn" id="btnClearAll" title="مسح الكل">مسح</button>
      </div>
    </div>

    <div class="display" id="display">
      <div class="expr" id="expr">&nbsp;</div>
      <div class="result" id="result">0</div>
    </div>

    <div class="keys" id="keys">
      <button class="key op" data-val="(">(</button>
      <button class="key op" data-val=")">)</button>
      <button class="key" id="btnBack" title="حذف">⌫</button>
      <button class="key op" data-val="/">÷</button>

      <button class="key" data-val="7">7</button>
      <button class="key" data-val="8">8</button>
      <button class="key" data-val="9">9</button>
      <button class="key op" data-val="*">×</button>

      <button class="key" data-val="4">4</button>
      <button class="key" data-val="5">5</button>
      <button class="key" data-val="6">6</button>
      <button class="key op" data-val="-">−</button>

      <button class="key" data-val="1">1</button>
      <button class="key" data-val="2">2</button>
      <button class="key" data-val="3">3</button>
      <button class="key op" data-val="+">+</button>

      <button class="key" data-val="0">0</button>
      <button class="key" data-val=".">.</button>
      <button class="key op" id="btnPercent" data-val="%">%</button>
      <button class="key accent wide" id="btnEquals">=</button>
    </div>

    <div class="history" id="history" aria-live="polite">
      <!-- history items -->
    </div>
  </div>

<script>
(() => {
  const exprEl = document.getElementById('expr');
  const resEl = document.getElementById('result');
  const keys = document.getElementById('keys');
  const histEl = document.getElementById('history');
  const btnClearAll = document.getElementById('btnClearAll');
  const btnBack = document.getElementById('btnBack');
  const btnEquals = document.getElementById('btnEquals');
  const btnCopy = document.getElementById('btnCopy');

  let expr = ''; // expression stored as JS-friendly (with % handled)
  let history = JSON.parse(localStorage.getItem('calc_history_v1')||'[]');

  function renderExpr(){
    exprEl.textContent = expr || '\u00A0'; // non-breaking space when empty
  }
  function renderResult(v){
    resEl.textContent = v;
  }
  function pushHistory(item){
    history.unshift(item);
    if(history.length>20) history.pop();
    localStorage.setItem('calc_history_v1', JSON.stringify(history));
    renderHistory();
  }
  function renderHistory(){
    histEl.innerHTML = '';
    if(history.length===0){ histEl.innerHTML = '<div style="opacity:.6">لا توجد عمليات حتى الآن</div>'; return; }
    history.forEach(h=>{
      const d = document.createElement('div'); d.className='item';
      const left = document.createElement('div'); left.innerHTML = '<strong>'+escapeHtml(h.exprDisplay)+'</strong><div><small>'+escapeHtml(h.when)+'</small></div>';
      const right = document.createElement('div'); right.innerHTML = '<div style="font-weight:800">'+escapeHtml(h.result)+'</div><button class="small-btn" style="margin-left:8px" data-load="'+escapeHtml(h.expr)+'">تحميل</button>';
      d.appendChild(left); d.appendChild(right);
      histEl.appendChild(d);
      const loadBtn = right.querySelector('button');
      loadBtn.addEventListener('click', ()=>{ expr = h.expr; renderExpr(); renderResult(h.result); });
    });
  }

  function escapeHtml(t){ return (t+'').replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c])); }

  // safe evaluate: allow digits, operators, parentheses, dot, percent, spaces
  function safeEval(raw){
    if(!raw) return '0';
    // convert display operators to JS operators
    let s = raw.replace(/×/g,'*').replace(/÷/g,'/').replace(/−/g,'-');
    // handle percent: convert 50% -> (50/100)
    s = s.replace(/(\d+(\.\d+)?)%/g, '($1/100)');
    // allowed chars only
    if(!/^[0-9+\-*/().\s%]+$/.test(s)) throw new Error('حرف غير مسموح');
    // final eval using Function
    try{
      // limit long evaluation time by simple length check
      if(s.length>200) throw new Error('طويل جداً');
      // eslint-disable-next-line no-new-func
      const fn = new Function('return ('+s+');');
      const val = fn();
      if(!isFinite(val)) throw new Error('غير معرف');
      return Math.round((val + Number.EPSILON) * 100000000)/100000000; // limit precision
    } catch(e){
      throw e;
    }
  }

  // handlers
  keys.addEventListener('click', e=>{
    const btn = e.target.closest('button');
    if(!btn) return;
    const v = btn.dataset.val;
    if(btn.id === 'btnEquals'){ calculate(); return; }
    if(btn.id === 'btnBack'){ expr = expr.slice(0,-1); renderExpr(); return; }
    if(btn.id === 'btnPercent'){ expr += '%'; renderExpr(); return; }
    if(v !== undefined){ expr += v; renderExpr(); }
  });

  btnClearAll.addEventListener('click', ()=>{ expr=''; renderExpr(); renderResult('0'); });
  btnBack.addEventListener('click', ()=>{ expr = expr.slice(0,-1); renderExpr(); });

  function calculate(){
    try{
      const display = expr.replace(/\*/g,'×').replace(/\//g,'÷').replace(/\-/g,'−');
      const value = safeEval(expr);
      renderResult(value);
      pushHistory({expr, exprDisplay:display, result:value+'', when: new Date().toLocaleString()});
    }catch(err){
      renderResult('خطأ');
      console.warn(err);
    }
  }

  // copy result
  btnCopy.addEventListener('click', ()=>{
    const txt = resEl.textContent || '';
    navigator.clipboard?.writeText(txt).then(()=> flash('نسخ إلى الحافظة'), ()=> flash('فشل النسخ'));
  });

  // keyboard support
  window.addEventListener('keydown', (e)=>{
    if(e.key === 'Enter'){ e.preventDefault(); calculate(); return; }
    if(e.key === 'Backspace'){ expr = expr.slice(0,-1); renderExpr(); return; }
    if(e.key === 'Escape'){ expr=''; renderExpr(); renderResult('0'); return; }
    const map = {'/':'/','*':'*','-':'-','+':'+','.':'.','%':'%','(':'(',' )':')'};
    if(map[e.key]){ expr += map[e.key]; renderExpr(); return; }
    if(/\d/.test(e.key)){ expr += e.key; renderExpr(); return; }
  });

  // small toast
  function flash(msg){
    const t = document.createElement('div');
    t.textContent = msg; t.style.position='fixed'; t.style.left='50%'; t.style.transform='translateX(-50%)';
    t.style.bottom='24px'; t.style.background='rgba(0,0,0,0.7)'; t.style.color='white'; t.style.padding='8px 12px'; t.style.borderRadius='10px';
    document.body.appendChild(t); setTimeout(()=> t.remove(),1400);
  }

  // init
  renderExpr();
  renderResult('0');
  renderHistory();

  // expose for debug (optional)
  window.calc = {
    get expr(){ return expr; },
    set expr(v){ expr = v; renderExpr(); }
  };
})();
</script>
</body>
</html>
