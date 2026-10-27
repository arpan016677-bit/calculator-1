# calculator-1
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Calculator</title>
<style>
  :root {
    --bg:#e9ecef; --body:#f8f9fa; --screen:#1f2933; --text:#f1f5f9; --dim:#8b98a7;
    --key:#fff; --key-t:#1f2933; --op:#dbe4ff; --op-t:#364fc7; --fn:#e9ecef;
    --sci:#eef1f5; --eq:#364fc7; --eq-t:#fff; --shadow:rgba(31,41,51,.18); --err:#ff8787;
  }
  :root[data-theme="dark"] {
    --bg:#0f1419; --body:#1a2129; --screen:#0b0f13; --key:#26303a; --key-t:#f1f5f9;
    --op:#2b3a67; --op-t:#b4c6ff; --fn:#323e4a; --sci:#212b35; --eq:#5c7cfa; --shadow:rgba(0,0,0,.5);
  }
  * { box-sizing:border-box; margin:0; padding:0; -webkit-tap-highlight-color:transparent; }
  html, body { min-height:100%; }
  body {
    background:var(--bg); display:flex; padding:16px; color:var(--key-t);
    font-family:"Segoe UI", system-ui, -apple-system, Roboto, sans-serif;
    transition:background .3s;
  }
  .calc {
    position:relative; overflow:hidden; margin:auto;
    width:100%; max-width:400px; background:var(--body);
    border-radius:28px; padding:16px; box-shadow:0 24px 50px var(--shadow);
    animation:rise .6s cubic-bezier(.2,.9,.3,1.2) both; transition:background .3s;
  }
  @keyframes rise { from{opacity:0; transform:translateY(30px) scale(.94);} to{opacity:1; transform:none;} }

  button {
    position:relative; overflow:hidden; border:none; cursor:pointer;
    font-family:inherit; font-weight:500; color:var(--key-t); background:var(--key);
    transition:transform .08s, box-shadow .08s, filter .15s, background .2s;
    outline:none;
  }
  button:hover { filter:brightness(.95); }
  button:focus-visible { outline:3px solid var(--eq); outline-offset:2px; }

  .top { display:flex; gap:8px; align-items:center; margin-bottom:12px; }
  .top button {
    background:var(--fn); border-radius:10px; padding:7px 12px; font-size:13px;
  }
  .top button.on { background:var(--eq); color:var(--eq-t); }
  .top .sp { flex:1; }
  #mind { font-size:13px; color:var(--op-t); font-weight:700; }

  .screen {
    background:var(--screen); border-radius:18px; padding:14px 16px; margin-bottom:14px;
    text-align:right; min-height:112px; display:flex; flex-direction:column;
    justify-content:flex-end; gap:4px; cursor:copy;
  }
  #sub { color:var(--dim); font-size:15px; min-height:20px; word-break:break-all; }
  #sub.error { color:var(--err); }
  #main {
    color:var(--text); font-size:44px; font-weight:600; line-height:1.15;
    word-break:break-all; max-height:130px; overflow-y:auto; scrollbar-width:none;
  }
  #main.pop { animation:pop .2s ease-out; }
  .screen.shake { animation:shake .35s; }
  @keyframes pop { 0%{transform:scale(.94); opacity:.5;} 100%{transform:scale(1); opacity:1;} }
  @keyframes shake { 0%,100%{transform:translateX(0);} 20%,60%{transform:translateX(-8px);} 40%,80%{transform:translateX(8px);} }

  .mem { display:grid; grid-template-columns:repeat(4,1fr); gap:8px; margin-bottom:10px; }
  .mem button { background:none; box-shadow:none; font-size:13px; padding:6px; border-radius:8px; color:var(--op-t); }

  .sci-wrap { display:grid; grid-template-rows:1fr; transition:grid-template-rows .35s ease, margin .35s; margin-bottom:10px; }
  .sci-wrap.closed { grid-template-rows:0fr; margin-bottom:0; }
  .sci-inner { overflow:hidden; }
  .sci { display:grid; grid-template-columns:repeat(5,1fr); gap:8px; padding:2px 0 4px; }
  .sci button { height:42px; font-size:15px; background:var(--sci); border-radius:12px; box-shadow:0 2px 0 var(--shadow); }

  .keys { display:grid; grid-template-columns:repeat(4,1fr); gap:10px; }
  .keys button { height:58px; font-size:22px; border-radius:16px; box-shadow:0 3px 0 var(--shadow); }
  .keys button.fn { background:var(--fn); }
  .keys button.op { background:var(--op); color:var(--op-t); font-size:26px; }
  .keys button.eq { background:var(--eq); color:var(--eq-t); }
  button:active, button.pressed { transform:translateY(3px); box-shadow:0 0 0 var(--shadow) !important; }

  .ripple { position:absolute; border-radius:50%; background:currentColor; opacity:.25;
    transform:scale(0); pointer-events:none; animation:ripple .5s ease-out forwards; }
  @keyframes ripple { to { transform:scale(4); opacity:0; } }

  /* History drawer */
  #drawer {
    position:absolute; inset:0; background:var(--body); padding:16px; z-index:5;
    display:flex; flex-direction:column; transform:translateX(105%);
    transition:transform .35s cubic-bezier(.3,.8,.3,1);
  }
  #drawer.open { transform:none; }
  #drawer header { display:flex; align-items:center; gap:8px; margin-bottom:12px; }
  #drawer h2 { flex:1; font-size:18px; }
  #drawer header button { background:var(--fn); border-radius:10px; padding:7px 12px; font-size:13px; }
  #hlist { list-style:none; overflow-y:auto; flex:1; display:flex; flex-direction:column; gap:8px; }
  #hlist button { width:100%; text-align:right; padding:10px 14px; border-radius:14px; background:var(--key); box-shadow:0 2px 0 var(--shadow); }
  #hlist small { display:block; color:var(--dim); font-size:13px; word-break:break-all; }
  #hlist strong { font-size:20px; word-break:break-all; }
  #hlist .empty { color:var(--dim); text-align:center; margin-top:40px; }
  #hlist li { animation:pop .25s ease-out; }

  #toast {
    position:fixed; left:50%; bottom:24px; transform:translate(-50%,20px); opacity:0;
    background:var(--screen); color:var(--text); padding:8px 16px; border-radius:20px;
    font-size:14px; transition:.25s; pointer-events:none; z-index:10;
  }
  #toast.show { opacity:1; transform:translate(-50%,0); }

  @media (prefers-reduced-motion:reduce) { * { animation:none !important; transition:none !important; } }
</style>
</head>
<body>

<main class="calc" aria-label="Calculator">
  <div class="top">
    <button id="bSci" class="on" title="Scientific functions">Sci</button>
    <button id="bAng" title="Angle mode">DEG</button>
    <button id="bHist" title="History">History</button>
    <span id="mind" hidden>M</span>
    <span class="sp"></span>
    <button id="bTheme" title="Switch theme" aria-label="Switch theme">☾</button>
  </div>

  <div class="screen" id="screen" title="Click to copy" aria-live="polite">
    <div id="sub"></div>
    <div id="main">0</div>
  </div>

  <div class="mem" id="mem">
    <button data-a="mc">MC</button><button data-a="mr">MR</button>
    <button data-a="mp">M+</button><button data-a="mm">M−</button>
  </div>

  <div class="sci-wrap" id="sciWrap"><div class="sci-inner"><div class="sci" id="sci"></div></div></div>
  <div class="keys" id="keys"></div>

  <aside id="drawer" aria-label="History">
    <header>
      <h2>History</h2>
      <button id="hClear">Clear</button>
      <button id="hClose">Close</button>
    </header>
    <ul id="hlist"></ul>
  </aside>
</main>
<div id="toast"></div>

<script>
(() => {
  const $ = s => document.querySelector(s);
  const mainEl = $('#main'), subEl = $('#sub'), screen = $('#screen');
  let expr = '', fresh = false, deg = true, mem = 0, ans = '0', hist = [];

  try { hist = JSON.parse(localStorage.getItem('calc-history') || '[]'); } catch (e) {}
  const saveHist = () => { try { localStorage.setItem('calc-history', JSON.stringify(hist)); } catch (e) {} };

  /* ---------- Build keys ---------- */
  const sciKeys = [
    ['sin','sin('],['cos','cos('],['tan','tan('],['ln','ln('],['log','log('],
    ['(','('],[')',')'],['π','π'],['e','e'],['x!','!'],
    ['x²','^2'],['xʸ','^'],['√','√('],['1/x',null,'inv'],['Ans','Ans']
  ];
  const mainKeys = [
    ['AC',null,'clear','fn'],['⌫',null,'del','fn'],['%','%',null,'fn'],['÷','÷',null,'op'],
    ['7','7'],['8','8'],['9','9'],['×','×',null,'op'],
    ['4','4'],['5','5'],['6','6'],['−','−',null,'op'],
    ['1','1'],['2','2'],['3','3'],['+','+',null,'op'],
    ['±',null,'sign','fn'],['0','0'],['.','.'],['=',null,'eq','eq']
  ];
  const build = (list) => list.map(([l, v, a, c]) =>
    `<button class="${c || ''}" ${v ? `data-v="${v}"` : ''} ${a ? `data-a="${a}"` : ''}>${l}</button>`).join('');
  $('#sci').innerHTML = build(sciKeys);
  $('#keys').innerHTML = build(mainKeys);

  /* ---------- Expression engine (safe recursive-descent parser) ---------- */
  const FN = ['sin','cos','tan','ln','log','√'];
  const isNum = t => /^[\d.]/.test(t);

  function tokenize(s) {
    const out = []; let i = 0;
    const re = /\d+\.?\d*(?:e[+-]?\d+)?|\.\d+|sin|cos|tan|ln|log|Ans|[π√e+\-−×÷^()!%]/y;
    while (i < s.length) {
      re.lastIndex = i;
      const m = re.exec(s);
      if (!m) throw new Error('Unexpected character');
      let t = m[0]; i = re.lastIndex;
      if (t === '-') t = '−';
      const p = out[out.length - 1];
      const value = isNum(t) || ['π','e','Ans'].includes(t);
      const opener = t === '(' || FN.includes(t);
      if (p !== undefined && (value || opener) && (isNum(p) || [')','!','%','π','e','Ans'].includes(p))) out.push('×');
      out.push(t);
    }
    return out;
  }

  const closeParens = s => {
    const d = (s.match(/\(/g) || []).length - (s.match(/\)/g) || []).length;
    if (d < 0) throw new Error('Extra closing bracket');
    return s + ')'.repeat(d);
  };

  const trig = f => x => f(deg ? x * Math.PI / 180 : x);
  const F = {
    sin: trig(Math.sin), cos: trig(Math.cos), tan: trig(Math.tan),
    ln: x => { if (x <= 0) throw new Error('ln needs a positive number'); return Math.log(x); },
    log: x => { if (x <= 0) throw new Error('log needs a positive number'); return Math.log10(x); },
    '√': x => { if (x < 0) throw new Error('Cannot take root of a negative'); return Math.sqrt(x); }
  };
  const fact = n => {
    if (n < 0 || !Number.isInteger(n)) throw new Error('Factorial needs a whole number');
    if (n > 170) throw new Error('Number too large');
    let r = 1; for (let i = 2; i <= n; i++) r *= i; return r;
  };

  function evaluate(src) {
    const t = tokenize(closeParens(src));
    let i = 0;
    const peek = () => t[i];
    const next = () => { if (i >= t.length) throw new Error('Incomplete expression'); return t[i++]; };

    const expr_ = () => {
      let v = term();
      while (peek() === '+' || peek() === '−') { const o = next(); const r = term(); v = o === '+' ? v + r : v - r; }
      return v;
    };
    const term = () => {
      let v = unary();
      while (peek() === '×' || peek() === '÷') {
        const o = next(); const r = unary();
        if (o === '÷' && r === 0) throw new Error('Cannot divide by zero');
        v = o === '×' ? v * r : v / r;
      }
      return v;
    };
    const unary = () => {
      if (peek() === '−') { next(); return -unary(); }
      if (peek() === '+') { next(); return unary(); }
      return power();
    };
    const power = () => {
      const b = postfix();
      if (peek() === '^') { next(); return Math.pow(b, unary()); }
      return b;
    };
    const postfix = () => {
      let v = primary();
      while (peek() === '!' || peek() === '%') v = next() === '!' ? fact(v) : v / 100;
      return v;
    };
    const primary = () => {
      const x = next();
      if (isNum(x)) return parseFloat(x);
      if (x === 'π') return Math.PI;
      if (x === 'e') return Math.E;
      if (x === 'Ans') return parseFloat(ans.replace('−', '-'));
      if (x === '(') {
        const v = expr_();
        if (next() !== ')') throw new Error('Missing closing bracket');
        return v;
      }
      if (FN.includes(x)) return F[x](primary());
      throw new Error('Invalid expression');
    };

    const result = expr_();
    if (i < t.length) throw new Error('Invalid expression');
    if (!isFinite(result)) throw new Error('Result is not a real number');
    return result;
  }

  const fmt = n => {
    let v = parseFloat(n.toPrecision(12));
    if (Math.abs(v) < 1e-12) v = 0;
    return String(v).replace(/^-/, '−');
  };

  /* ---------- Rendering ---------- */
  function render(msg, isErr) {
    mainEl.textContent = expr || '0';
    const n = mainEl.textContent.length;
    mainEl.style.fontSize = n > 24 ? '20px' : n > 16 ? '26px' : n > 10 ? '34px' : '44px';
    subEl.classList.toggle('error', !!isErr);
    if (msg !== undefined) { subEl.textContent = msg; return; }
    subEl.textContent = '';
    if (expr && !fresh) {
      try {
        const r = fmt(evaluate(expr));
        if (r !== expr) subEl.textContent = '= ' + r;
      } catch (e) {}
    }
  }
  const pop = () => { mainEl.classList.remove('pop'); void mainEl.offsetWidth; mainEl.classList.add('pop'); };
  const shake = () => { screen.classList.remove('shake'); void screen.offsetWidth; screen.classList.add('shake'); };

  /* ---------- Input logic ---------- */
  const OPS = '+−×÷^';
  function insert(v) {
    const startsNew = /^[\d.πe(]|^(sin|cos|tan|ln|log|√|Ans)/.test(v);
    if (fresh) { if (startsNew) expr = ''; fresh = false; }
    if (expr.length >= 120) return;
    const last = expr.slice(-1);
    const opens = (expr.match(/\(/g) || []).length, closes = (expr.match(/\)/g) || []).length;

    if ('+×÷^'.includes(v)) {
      if (!expr || last === '(') return;
      expr = OPS.includes(last) ? expr.slice(0, -1) + v : expr + v;
    } else if (v === '−') {
      expr = (last === '−' || last === '+') ? expr.slice(0, -1) + v : expr + v;
    } else if (v === '!' || v === '%') {
      if (!expr || OPS.includes(last) || last === '(') return;
      expr += v;
    } else if (v === ')') {
      if (opens <= closes || OPS.includes(last) || last === '(') return;
      expr += v;
    } else if (v === '.') {
      const m = /[\d.]+$/.exec(expr);
      if (m && m[0].includes('.')) return;
      expr += m ? '.' : '0.';
    } else if (/^\d$/.test(v)) {
      if (/(^|[^\d.])0$/.test(expr)) expr = expr.slice(0, -1);
      expr += v;
    } else {
      expr += v;
    }
    render(); pop();
  }

  function equals() {
    if (!expr) return;
    try {
      const r = fmt(evaluate(expr));
      const e = closeParens(expr);
      if (e !== r) {
        hist.unshift({ e, r }); hist = hist.slice(0, 50); saveHist(); renderHist();
      }
      ans = r; expr = r; fresh = true;
      render(e + ' ='); pop();
    } catch (err) {
      render(err.message, true); shake();
    }
  }

  const actions = {
    clear() { expr = ''; fresh = false; render(); pop(); },
    del() { fresh = false; expr = expr.replace(/(sin\(|cos\(|tan\(|ln\(|log\(|√\(|Ans|.)$/, ''); render(); },
    sign() {
      if (!expr) return; fresh = false;
      expr = /^−[\d.]+$/.test(expr) ? expr.slice(1) : /^[\d.]+$/.test(expr) ? '−' + expr : '−(' + expr + ')';
      render(); pop();
    },
    inv() { fresh = false; expr = expr ? '1÷(' + expr + ')' : '1÷('; render(); pop(); },
    eq: equals,
    mc() { mem = 0; updMem(); toast('Memory cleared'); },
    mr() { insert(fmt(mem)); },
    mp() { memAdd(1); },
    mm() { memAdd(-1); }
  };
  function memAdd(sign) {
    try { mem += sign * evaluate(expr || '0'); updMem(); toast('Memory: ' + fmt(mem)); }
    catch (e) { render(e.message, true); shake(); }
  }
  const updMem = () => { $('#mind').hidden = mem === 0; };

  /* ---------- History ---------- */
  function renderHist() {
    const ul = $('#hlist');
    ul.innerHTML = hist.length ? '' : '<li class="empty">No calculations yet</li>';
    hist.forEach(h => {
      const li = document.createElement('li');
      const b = document.createElement('button');
      b.innerHTML = '<small></small><strong></strong>';
      b.children[0].textContent = h.e + ' =';
      b.children[1].textContent = h.r;
      b.onclick = () => { expr = h.r; fresh = true; render(h.e + ' ='); pop(); $('#drawer').classList.remove('open'); };
      li.appendChild(b); ul.appendChild(li);
    });
  }

  /* ---------- UI wiring ---------- */
  function toast(msg) {
    const t = $('#toast'); t.textContent = msg; t.classList.add('show');
    clearTimeout(toast.id); toast.id = setTimeout(() => t.classList.remove('show'), 1400);
  }

  document.querySelector('.calc').addEventListener('click', e => {
    const btn = e.target.closest('button[data-v], button[data-a]');
    if (!btn) return;
    const r = document.createElement('span'), rect = btn.getBoundingClientRect(), size = Math.max(rect.width, rect.height);
    r.className = 'ripple'; r.style.width = r.style.height = size / 3 + 'px';
    r.style.left = ((e.clientX ? e.clientX - rect.left : rect.width / 2) - size / 6) + 'px';
    r.style.top = ((e.clientY ? e.clientY - rect.top : rect.height / 2) - size / 6) + 'px';
    btn.appendChild(r); setTimeout(() => r.remove(), 500);
    if (btn.dataset.v) insert(btn.dataset.v); else actions[btn.dataset.a]();
  });

  $('#bSci').onclick = e => { $('#sciWrap').classList.toggle('closed'); e.currentTarget.classList.toggle('on'); };
  $('#bAng').onclick = e => { deg = !deg; e.currentTarget.textContent = deg ? 'DEG' : 'RAD'; render(); };
  $('#bHist').onclick = () => $('#drawer').classList.add('open');
  $('#hClose').onclick = () => $('#drawer').classList.remove('open');
  $('#hClear').onclick = () => { hist = []; saveHist(); renderHist(); };
  screen.onclick = async () => {
    try { await navigator.clipboard.writeText(expr || '0'); toast('Copied'); } catch (e) { toast('Copy not available'); }
  };

  const applyTheme = t => { document.documentElement.dataset.theme = t; $('#bTheme').textContent = t === 'dark' ? '☀' : '☾'; };
  let theme = null;
  try { theme = localStorage.getItem('calc-theme'); } catch (e) {}
  applyTheme(theme || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'));
  $('#bTheme').onclick = () => {
    const t = document.documentElement.dataset.theme === 'dark' ? 'light' : 'dark';
    applyTheme(t); try { localStorage.setItem('calc-theme', t); } catch (e) {}
  };

  /* ---------- Keyboard ---------- */
  const keyMap = { '-': '−', '*': '×', '/': '÷', ',': '.', 'x': '×', 'p': 'π' };
  window.addEventListener('keydown', e => {
    if (e.ctrlKey || e.metaKey || e.altKey) return;
    let sel = null, k = keyMap[e.key] || e.key;
    if (/^[\d.+−×÷^()!%π]$/.test(k)) sel = `[data-v="${k}"]`;
    else if (e.key === 'Enter' || e.key === '=') sel = '[data-a="eq"]';
    else if (e.key === 'Backspace') sel = '[data-a="del"]';
    else if (e.key === 'Escape') {
      if ($('#drawer').classList.contains('open')) { $('#drawer').classList.remove('open'); return; }
      sel = '[data-a="clear"]';
    }
    if (!sel) return;
    e.preventDefault();
    const b = document.querySelector(sel); if (!b) return;
    b.classList.add('pressed'); setTimeout(() => b.classList.remove('pressed'), 120);
    b.click();
  });

  renderHist(); updMem(); render();
})();
</script>
</body>
</html>
