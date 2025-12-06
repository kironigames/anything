<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Search & Powers — Your mini web app</title>
<style>
  :root{
    --bg:#0f1724; --card:#0b1220; --accent:#7c3aed; --muted:#9aa4b2;
    --glass: rgba(255,255,255,0.03);
    --radius:14px;
    font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  *{box-sizing:border-box}
  body{
    margin:0; min-height:100vh; background:
    radial-gradient(1000px 400px at 10% 10%, rgba(124,58,237,0.12), transparent 6%),
    linear-gradient(180deg, #071022 0%, #061420 50%),
    var(--bg);
    color:#e6eef6;
    padding:28px;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
  }
  .wrap{max-width:980px; margin:0 auto; display:grid; gap:20px;
    grid-template-columns: 1fr 420px;
  }
  header{grid-column:1/-1; display:flex; align-items:center; gap:16px}
  h1{margin:0; font-size:20px; letter-spacing:0.4px}
  .subtitle{color:var(--muted); font-size:13px}
  .card{
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border:1px solid rgba(255,255,255,0.04);
    padding:16px; border-radius:var(--radius); box-shadow: 0 6px 30px rgba(2,6,23,0.6);
  }

  /* Search area */
  .search-area{display:flex; gap:12px; align-items:center}
  .search-box{flex:1; display:flex; gap:8px}
  input[type="search"]{
    flex:1; padding:12px 14px; border-radius:10px; border:none; outline:none;
    background:var(--glass); color:inherit; font-size:14px;
  }
  select{padding:10px 12px; border-radius:10px; border:none; background:var(--glass); color:inherit}
  .btn{
    background:linear-gradient(90deg,var(--accent), #5b21b6); border:none; color:white;
    padding:10px 14px; border-radius:10px; cursor:pointer; font-weight:600;
    box-shadow: 0 6px 18px rgba(92,33,182,0.18);
  }

  /* Powers column */
  .powers-col{display:flex; flex-direction:column; gap:12px;}
  .power-list{display:flex; flex-direction:column; gap:8px; margin-top:6px;}
  .power-item{
    display:flex; justify-content:space-between; gap:8px; align-items:center;
    padding:10px; border-radius:10px; background:linear-gradient(180deg, rgba(255,255,255,0.015), transparent);
    border:1px solid rgba(255,255,255,0.02);
  }
  .power-name{font-weight:700}
  .small{font-size:12px; color:var(--muted)}

  form.create-power{display:grid; gap:8px; grid-template-columns:1fr;}
  input[type="text"], textarea, input[type="range"]{
    width:100%; padding:8px 10px; border-radius:8px; border:none; background:var(--glass); color:inherit;
  }
  textarea{min-height:64px; resize:vertical}

  .actions{display:flex; gap:8px}
  .ghost{background:transparent; border:1px solid rgba(255,255,255,0.06); padding:8px 10px; border-radius:8px; color:var(--muted); cursor:pointer}

  /* granted badge */
  .badge{
    display:inline-flex; gap:8px; align-items:center; padding:8px 12px; border-radius:999px;
    background: linear-gradient(90deg, rgba(124,58,237,0.12), rgba(59,130,246,0.06));
    color:#eaf2ff; font-weight:700; font-size:13px;
  }

  /* Confetti / animation overlay */
  .overlay{
    position:fixed; inset:0; pointer-events:none; display:flex; align-items:center; justify-content:center;
  }
  .modal{
    position:fixed; left:50%; top:50%; transform:translate(-50%,-50%); z-index:50;
    background:#071127; padding:18px; border-radius:14px; border:1px solid rgba(255,255,255,0.04);
    box-shadow: 0 20px 60px rgba(2,6,23,0.8);
  }

  footer{grid-column:1/-1; color:var(--muted); font-size:13px; text-align:center; margin-top:6px;}

  /* responsive */
  @media (max-width:920px){
    .wrap{grid-template-columns:1fr; padding-bottom:80px}
    .powers-col{order:2}
    header{flex-direction:column; align-items:flex-start}
  }

  /* simple visual for power strength */
  .strength{
    height:8px; width:120px; background:rgba(255,255,255,0.04); border-radius:8px; overflow:hidden;
  }
  .strength > i{display:block; height:100%; background:linear-gradient(90deg,#34d399,#60a5fa); width:30%}
  /* small helper */
  .note{color:var(--muted); font-size:13px}
</style>
</head>
<body>
  <div class="wrap">
    <header>
      <div>
        <h1>Search & Powers</h1>
        <div class="subtitle">Search the web + create and "possess" fictional powers (saved locally)</div>
      </div>
      <div style="margin-left:auto; display:flex; gap:10px; align-items:center">
        <div class="badge" id="current-power-badge" title="Your currently granted power (if any)">
          No power granted
        </div>
      </div>
    </header>

    <!-- LEFT: Search + generator -->
    <section class="card">
      <h2 style="margin:0 0 10px 0">Search the web</h2>
      <div class="search-area">
        <div class="search-box" style="flex:1">
          <input id="q" type="search" placeholder="Search the web — type and press Search" autocomplete="off" />
          <select id="engine" title="Choose search engine">
            <option value="https://www.google.com/search?q=">Google</option>
            <option value="https://www.bing.com/search?q=">Bing</option>
            <option value="https://duckduckgo.com/?q=" selected>DuckDuckGo</option>
            <option value="https://www.startpage.com/sp/search?q=">Startpage</option>
          </select>
        </div>
        <button class="btn" id="searchBtn">Search</button>
      </div>
      <div style="margin-top:12px" class="note">Tip: press Enter to search. Results open in a new tab. This page doesn't index the web itself.</div>

      <hr style="margin:16px 0; border:none; height:1px; background:rgba(255,255,255,0.02)">

      <h3 style="margin:0 0 10px 0">Quick tools</h3>
      <div style="display:flex; gap:8px; flex-wrap:wrap">
        <button class="ghost" id="searchImages">Search images</button>
        <button class="ghost" id="searchNews">Search news</button>
        <button class="ghost" id="searchSite">Search this site (example)</button>
      </div>

      <hr style="margin:16px 0; border:none; height:1px; background:rgba(255,255,255,0.02)">

      <h3 style="margin:0 0 10px 0">Random power generator</h3>
      <div style="display:flex; gap:8px; align-items:center">
        <button class="btn" id="genPower">Generate random power</button>
        <div class="note" id="randomPowerPreview" style="margin-left:auto">No power yet</div>
      </div>
      <div style="margin-top:10px; color:var(--muted); font-size:13px">
        Generated powers are fictional and for fun.
      </div>
    </section>

    <!-- RIGHT: Powers management -->
    <aside class="card powers-col">
      <div>
        <h2 style="margin:0 0 8px 0">Powers — create & possess</h2>
        <div class="small">Design custom powers, save them, and "grant" yourself a power.</div>
      </div>

      <form id="createPower" class="create-power" onsubmit="return false">
        <input id="pname" type="text" placeholder="Power name (e.g. Telekinesis)" required />
        <textarea id="pdesc" placeholder="Short description / effect (e.g. Move objects with your mind)"></textarea>
        <label class="small">Strength: <span id="strengthVal">50</span></label>
        <input id="pstr" type="range" min="1" max="100" value="50" />
        <div class="actions">
          <button id="savePower" class="btn">Save Power</button>
          <button id="grantRandom" class="ghost">Grant a Random Power</button>
        </div>
      </form>

      <div style="margin-top:6px">
        <h4 style="margin:0 0 8px 0">Saved powers</h4>
        <div id="powers" class="power-list">
          <!-- power items go here -->
        </div>
      </div>

      <div style="margin-top:8px; display:flex; gap:8px; justify-content:space-between; align-items:center">
        <div class="small">Your powers are saved in this browser only.</div>
        <div style="display:flex; gap:8px">
          <button class="ghost" id="exportJSON">Export JSON</button>
          <button class="ghost" id="clearAll">Clear All</button>
        </div>
      </div>
    </aside>

    <footer>
      Made for you — save this file and open it locally. No account required. Have fun and stay fictional!
    </footer>
  </div>

  <!-- Modal placeholder -->
  <div id="modalRoot"></div>
  <div id="confettiRoot" class="overlay" aria-hidden="true"></div>

<script>
/*
  Search & Powers app
  Save as index.html and open.
  - Search: builds a query URL for the chosen engine and opens a new tab.
  - Powers: create, list, grant, save to localStorage.
*/

const $ = id => document.getElementById(id);

// --- SEARCH HANDLERS ---
const qInput = $('q'), engineSel = $('engine'), searchBtn = $('searchBtn');

function searchQuery(options = {}) {
  const q = (options.q ?? qInput.value ?? '').trim();
  if (!q) {
    alert('Type something to search.');
    return;
  }
  const engineBase = engineSel.value;
  // support image/news shortcuts
  const mode = options.mode || '';
  let url = engineBase + encodeURIComponent(q);
  if (mode === 'images') {
    // common image parameter for engines (best-effort)
    if (engineBase.includes('duckduckgo')) url = 'https://duckduckgo.com/?q=' + encodeURIComponent(q) + '&iax=images&ia=images';
    if (engineBase.includes('google')) url = 'https://www.google.com/search?tbm=isch&q=' + encodeURIComponent(q);
    if (engineBase.includes('bing')) url = 'https://www.bing.com/images/search?q=' + encodeURIComponent(q);
  } else if (mode === 'news') {
    if (engineBase.includes('google')) url = 'https://www.google.com/search?tbm=nws&q=' + encodeURIComponent(q);
    else if (engineBase.includes('bing')) url = 'https://www.bing.com/news/search?q=' + encodeURIComponent(q);
    else url = engineBase + encodeURIComponent(q) + '&t=news';
  } else if (mode === 'site') {
    // example: search this site (this file). Using site:example for Google-friendly result.
    const site = location.hostname || 'example.com';
    url = engineBase + encodeURIComponent(\`site:\${site} \${q}\`);
  }
  window.open(url, '_blank');
}
searchBtn.addEventListener('click', () => searchQuery());
qInput.addEventListener('keydown', e => { if (e.key === 'Enter') searchQuery(); });

$('searchImages').addEventListener('click', () => searchQuery({mode:'images'}));
$('searchNews').addEventListener('click', () => searchQuery({mode:'news'}));
$('searchSite').addEventListener('click', () => {
  const q = prompt('Search this site for (example):', qInput.value || '');
  if (q !== null) {
    qInput.value = q;
    searchQuery({mode:'site'});
  }
});

// --- POWERS STORAGE & UI ---
const STORAGE_KEY = 'my_powers_v1';
let powers = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');

function savePowers(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(powers));
}

function renderPowers(){
  const container = $('powers');
  container.innerHTML = '';
  if (!powers.length){
    container.innerHTML = '<div class="small">No saved powers yet — create one or generate a random power.</div>';
    return;
  }
  powers.forEach((p, idx) => {
    const el = document.createElement('div');
    el.className = 'power-item';
    el.innerHTML = \`
      <div>
        <div class="power-name">\${escapeHtml(p.name)}</div>
        <div class="small">\${escapeHtml(p.desc || '')}</div>
      </div>
      <div style="display:flex; gap:8px; align-items:center">
        <div class="small" title="Strength">
          <div class="strength" style="width:100px"><i style="width:\${Math.max(6,p.strength)}%"></i></div>
        </div>
        <button class="ghost" data-idx="\${idx}" onclick="grantPower(\${idx})">Grant</button>
        <button class="ghost" data-idx="\${idx}" onclick="deletePower(\${idx})">Delete</button>
      </div>
    \`;
    container.appendChild(el);
  });
}

// utility: escape for safety
function escapeHtml(s){ return String(s || '').replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c])); }

// Save from form
$('pstr').addEventListener('input', e => $('strengthVal').textContent = e.target.value);
$('savePower').addEventListener('click', () => {
  const name = $('pname').value.trim();
  const desc = $('pdesc').value.trim();
  const strength = parseInt($('pstr').value || 50, 10);
  if (!name) { alert('Give the power a name.'); return; }
  const p = { id: Date.now(), name, desc, strength };
  powers.unshift(p);
  savePowers();
  renderPowers();
  $('pname').value=''; $('pdesc').value=''; $('pstr').value=50; $('strengthVal').textContent='50';
});

// delete
function deletePower(i){
  if (!confirm('Delete this saved power?')) return;
  powers.splice(i,1);
  savePowers(); renderPowers();
}

// clear all
$('clearAll').addEventListener('click', () => {
  if (!confirm('Clear all saved powers? This cannot be undone.')) return;
  powers = []; savePowers(); renderPowers();
});

// export
$('exportJSON').addEventListener('click', () => {
  const blob = new Blob([JSON.stringify(powers, null, 2)], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = 'powers.json'; a.click();
  URL.revokeObjectURL(url);
});

// generate random power
const prefixes = ['Trans','Electro','Chrono','Aero','Pyro','Aqua','Shadow','Light','Geo','Psy'];
const nouns = ['kinesis','shift','blast','cloak','field','sense','link','echo','wing','morph'];
const effects = [
  'Move objects with your mind.',
  'Slow down or speed up time nearby.',
  'Breathe underwater and control currents.',
  'Create a temporary protective shield.',
  'Teleport short distances.',
  'Phase through thin walls for a moment.',
  'Read surface thoughts for a few seconds.',
  'Summon a small familiar for aid.',
  'Turn invisible while still audible.',
  'Project calming or chaotic emotions.'
];
function randomFrom(a){ return a[Math.floor(Math.random()*a.length)]; }
const randomPowerPreview = $('randomPowerPreview');
$('genPower').addEventListener('click', () => {
  const name = randomFrom(prefixes) + randomFrom(nouns);
  const desc = randomFrom(effects);
  const strength = Math.floor(30 + Math.random()*70);
  randomPowerPreview.textContent = \`\${name} — \${desc} (strength \${strength})\`;
  // quick option: add to saved list
  if (confirm('Save this random power to your list?')) {
    powers.unshift({id:Date.now(), name, desc, strength});
    savePowers(); renderPowers();
  }
});

// grant a random saved power
$('grantRandom').addEventListener('click', () => {
  if (!powers.length){
    alert('No saved powers to grant — generate or save one first.');
    return;
  }
  const p = powers[Math.floor(Math.random()*powers.length)];
  grantPowerByObj(p);
});

// grant handler accessible from inline onclick
window.grantPower = function(i){
  const p = powers[i];
  if (!p) return;
  grantPowerByObj(p);
};

function grantPowerByObj(p){
  // visual modal + confetti
  showModal(\`You are granted: <strong>\${escapeHtml(p.name)}</strong>\`, \`\${escapeHtml(p.desc || '')}<br><br><em>Strength: \${p.strength}</em>\`);
  // store current in localStorage separately
  localStorage.setItem('current_power', JSON.stringify(p));
  updateBadge();
}

// update badge with current power
function updateBadge(){
  const badge = $('current-power-badge');
  const cur = JSON.parse(localStorage.getItem('current_power') || 'null');
  if (cur) badge.textContent = cur.name;
  else badge.textContent = 'No power granted';
}

// delete-power function (must be global for onclick to see it)
window.deletePower = deletePower;

// modal & confetti simple implementation
function showModal(titleHtml, bodyHtml){
  const root = $('modalRoot');
  root.innerHTML = '';
  const modal = document.createElement('div');
  modal.className = 'modal';
  modal.innerHTML = \`<div style="font-weight:800; margin-bottom:8px; font-size:16px">\${titleHtml}</div><div style="color:var(--muted); margin-bottom:12px">\${bodyHtml}</div><div style="display:flex; gap:8px; justify-content:flex-end"><button id="closeModal" class="btn">Nice!</button></div>\`;
  root.appendChild(modal);
  // confetti
  launchConfetti();
  document.getElementById('closeModal').focus();
  document.getElementById('closeModal').addEventListener('click', () => {
    root.innerHTML = '';
  });
}

// tiny confetti: spawn emojis that float
function launchConfetti(){
  const layer = $('confettiRoot');
  if(!layer) return;
  layer.innerHTML = '';
  layer.style.pointerEvents = 'none';
  for(let i=0;i<18;i++){
    const e = document.createElement('div');
    e.textContent = ['✨','⚡','🌟','🔮','🔥','🌀'][Math.floor(Math.random()*6)];
    Object.assign(e.style, {
      position:'absolute', left: (20+Math.random()*60)+'%',
      top: (40+Math.random()*10)+'%',
      fontSize: (12+Math.random()*28)+'px',
      opacity:1,
      transform: 'translateY(0) rotate('+ (Math.random()*360) +'deg)',
      transition: 'transform 1200ms cubic-bezier(.2,.8,.2,1), opacity 1200ms ease-out'
    });
    layer.appendChild(e);
    // animate
    requestAnimationFrame(()=> {
      const dx = (Math.random()*180-90);
      const dy = - (120 + Math.random()*260);
      e.style.transform = \`translate(\${dx}px,\${dy}px) rotate(\${Math.random()*720}deg) scale(1.02)\`;
      e.style.opacity = 0;
    });
  }
  // clear later
  setTimeout(()=> layer.innerHTML = '', 1500);
}

// load initial
renderPowers();
updateBadge();

// restore user-pasted JSON (optional)
(function tryLoadIfHashJson(){
  // if user drops JSON into location.hash like #load={"name":"X"} it's disabled for safety
})();

</script>
</body>
</html>
