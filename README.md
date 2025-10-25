<!doctype html>
<html lang="pl">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>AutoMine — wydobywanie punktów i wymiana na dolary</title>
<style>
  :root{
    --bg:#f4f6f8; --card:#fff; --accent:#2b7a78; --muted:#7b8a90;
    --glass: rgba(255,255,255,0.85);
  }
  body{
    margin:0; font-family:Inter, system-ui, Roboto, "Helvetica Neue", Arial;
    background: linear-gradient(180deg,#e8f0f2 0%, #f8fbfc 100%);
    color:#123; -webkit-font-smoothing:antialiased;
  }
  .wrap{max-width:980px;margin:18px auto;padding:18px;}
  header{display:flex;gap:12px;align-items:center;}
  h1{margin:0;font-size:1.25rem;color:var(--accent);}
  .topbar{margin-left:auto;display:flex;gap:10px;align-items:center;}
  .stat{background:var(--card);padding:8px 12px;border-radius:10px;box-shadow:0 6px 18px rgba(15,20,25,0.06);display:flex;flex-direction:column;align-items:flex-end;}
  .stat small{color:var(--muted);font-size:12px}
  .stat strong{font-size:18px}
  .board{display:grid;grid-template-columns:360px 1fr;gap:16px;margin-top:16px;}
  .card{background:var(--card);padding:14px;border-radius:12px;box-shadow:0 8px 24px rgba(15,20,25,0.06);}
  .game-left{display:flex;flex-direction:column;gap:12px;align-items:center;text-align:center;}
  .avatar{width:160px;height:120px;border-radius:14px;background:linear-gradient(180deg,#fff 0,#f2f6f8 100%);display:flex;align-items:center;justify-content:center;box-shadow:inset 0 -6px 18px rgba(0,0,0,0.03);}
  .avatar svg{width:110px;height:78px}
  .big-number{font-size:28px;margin:8px 0;color:#0b4f4d;font-weight:700;}
  .controls{display:flex;gap:10px;flex-wrap:wrap;justify-content:center;}
  button{background:var(--accent);color:white;border:none;padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600}
  .btn-light{background:#fff;color:var(--accent);border:1px solid #dbe6e6}
  .mini{padding:7px 10px;font-size:14px}
  .upgrades{display:flex;gap:10px;flex-wrap:wrap;justify-content:center;}
  .upgrade{background:linear-gradient(180deg,#fff,#fafafa);padding:10px;border-radius:10px;min-width:160px;box-shadow:0 6px 12px rgba(12,18,20,0.04);}
  .upgrade h4{margin:0 0 6px 0;font-size:14px}
  .row{display:flex;justify-content:space-between;align-items:center;gap:10px}
  .muted{color:var(--muted);font-size:13px}
  .actions{margin-top:12px;display:flex;gap:8px;justify-content:center;}
  .footer-note{font-size:13px;color:var(--muted);margin-top:12px;background:linear-gradient(180deg,rgba(255,255,255,0.6),rgba(255,255,255,0.4));padding:10px;border-radius:8px}
  @media (max-width:880px){ .board{grid-template-columns:1fr; } .avatar{width:140px;height:110px} }
</style>
</head>
<body>
  <div class="wrap">
    <header>
      <svg width="44" height="44" viewBox="0 0 24 24" fill="none" aria-hidden><path d="M3 12h18" stroke="#2b7a78" stroke-width="2" stroke-linecap="round"/></svg>
      <h1>AutoMine — wydobywanie punktów</h1>
      <div class="topbar">
        <div class="stat"><small>$ Dolar</small><strong id="dollars">0.00</strong></div>
        <div class="stat"><small>Punkty</small><strong id="points">0</strong></div>
      </div>
    </header>

    <div class="board">
      <!-- LEFT -->
      <div class="card game-left">
        <div class="avatar" title="Twoja flota">
          <!-- samochód SVG -->
          <svg viewBox="0 0 64 40" xmlns="http://www.w3.org/2000/svg" role="img" aria-hidden>
            <rect x="2" y="18" width="60" height="14" rx="2" fill="#2b7a78"/>
            <path d="M8 18 L14 8 H46 L52 18" fill="#3fa29a"/>
            <circle cx="18" cy="34" r="4" fill="#0b4f4d"/>
            <circle cx="46" cy="34" r="4" fill="#0b4f4d"/>
          </svg>
        </div>

        <div class="big-number" id="storedDisplay">Przechowywane punkty: 0 / 1000</div>

        <div class="controls">
          <button id="mineBtn" class="mini">🚗 Drive / Wydobyj punkty</button>
          <button id="collectBtn" class="btn-light mini">💵 Zamień na $ / Collect</button>
        </div>

        <div class="actions">
          <div class="muted">Kurs: <strong id="rateLabel">10000</strong> punktów = $1</div>
        </div>

        <div class="footer-note">
          Punkty generowane są ręcznie (klik) i pasywnie przez kierowców (fleet). Garaż (capacity) ogranicza ile punktów możesz mieć jednocześnie. Sieć tras (route map) podnosi dzienny limit zdobycia.
        </div>
      </div>

      <!-- RIGHT -->
      <div class="card">
        <div style="display:flex;gap:12px;align-items:center;flex-wrap:wrap">
          <div style="flex:1">
            <h3 style="margin:0 0 6px 0">Statystyki i ulepszenia</h3>
            <div class="muted">Zarządzaj flotą i ulepszaj by zwiększyć pasywne wydobycie.</div>
          </div>
          <div style="text-align:right">
            <div class="muted">Zapis gry: automatyczny</div>
          </div>
        </div>

        <div style="margin-top:12px;display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:10px">
          <div class="upgrade">
            <div class="row"><h4>🚘 Kierowcy (Fleet)</h4><div class="muted" id="fleetCount">0</div></div>
            <div class="muted">Każdy kierowca daje pasywne punkty /s</div>
            <div style="display:flex;gap:8px;margin-top:8px;align-items:center">
              <button class="mini" id="buyDriver">Kup kierowcę</button>
              <div class="muted" id="driverCost">$5.00</div>
            </div>
          </div>

          <div class="upgrade">
            <div class="row"><h4>🔧 Garaż (Capacity)</h4><div class="muted" id="capacityDisplay">1000</div></div>
            <div class="muted">Większa pojemność punktów — rzadziej musisz zbierać</div>
            <div style="display:flex;gap:8px;margin-top:8px;align-items:center">
              <button class="mini" id="buyCapacity">Zwiększ garaż</button>
              <div class="muted" id="capacityCost">$2.50</div>
            </div>
          </div>

          <div class="upgrade">
            <div class="row"><h4>🗺️ Sieć tras (Route Map)</h4><div class="muted" id="routeDisplay">10000</div></div>
            <div class="muted">Zwiększa maksymalny dzienny zysk punktów</div>
            <div style="display:flex;gap:8px;margin-top:8px;align-items:center">
              <button class="mini" id="buyRoute">Rozszerz trasę</button>
              <div class="muted" id="routeCost">$4.00</div>
            </div>
          </div>
        </div>

        <div style="margin-top:14px;">
          <h4 style="margin:0 0 6px 0">Postęp</h4>
          <div class="muted">Dzienne zebrane: <strong id="dailyCollected">0</strong> / <strong id="dailyLimit">10000</strong> punktów</div>
          <div style="height:10px;background:#eef5f5;border-radius:999px;margin-top:8px;overflow:hidden">
            <div id="progressBar" style="height:100%;width:0%;background:linear-gradient(90deg,#2b7a78,#3fa29a)"></div>
          </div>
        </div>

        <div style="margin-top:14px;display:flex;gap:8px;flex-wrap:wrap;align-items:center">
          <button id="resetBtn" class="btn-light mini">Reset gry</button>
          <button id="exportBtn" class="mini btn-light">Eksportuj (kopiuj stan)</button>
        </div>

        <div style="margin-top:12px;" class="muted">
          <strong>Uwaga:</strong> wszystkie wartości są przechowywane lokalnie w Twojej przeglądarce.
        </div>
      </div>
    </div>
  </div>

<script>
/*
  AutoMine - prosty system wydobywania punktów i wymiany na dolary.
  Autor: wygenerowane przez asystenta
*/

// --- Domyślne wartości gry ---
const DEFAULT = {
  points: 0, // przechowywane punkty
  dollars: 0.0,
  fleet: 0, // liczba kierowców
  capacity: 1000, // garaż - max przechowywanych punktów
  route: 10000, // dzienny limit (treasure map analog)
  dailyCollected: 0, // ile już dziś zebrano
  lastTick: Date.now(),
  conversionRate: 10000 // ile punktów = $1
};

// --- Koszty (mogą rosnąć) ---
function driverCost(fleet) { return (5 * Math.pow(1.15, fleet)).toFixed(2); } // $ cost
function capacityCost(levels) { return (2.5 * Math.pow(1.2, Math.log2(levels/1000+1))).toFixed(2); }
function routeCost(route) { return (4 * Math.pow(1.18, (route/10000)-1)).toFixed(2); }

// --- Ładowanie / zapisywanie stanu ---
function saveState(state){
  localStorage.setItem('automine_v1', JSON.stringify(state));
}
function loadState(){
  try{
    const raw = localStorage.getItem('automine_v1');
    if(!raw) return {...DEFAULT};
    const st = JSON.parse(raw);
    return Object.assign({}, DEFAULT, st);
  }catch(e){
    return {...DEFAULT};
  }
}

let state = loadState();

// --- UI elements ---
const pointsEl = document.getElementById('points');
const dollarsEl = document.getElementById('dollars');
const storedEl = document.getElementById('storedDisplay');
const mineBtn = document.getElementById('mineBtn');
const collectBtn = document.getElementById('collectBtn');
const rateLabel = document.getElementById('rateLabel');
const fleetCount = document.getElementById('fleetCount');
const capacityDisplay = document.getElementById('capacityDisplay');
const routeDisplay = document.getElementById('routeDisplay');
const driverCostEl = document.getElementById('driverCost');
const capacityCostEl = document.getElementById('capacityCost');
const routeCostEl = document.getElementById('routeCost');
const buyDriverBtn = document.getElementById('buyDriver');
const buyCapacityBtn = document.getElementById('buyCapacity');
const buyRouteBtn = document.getElementById('buyRoute');
const dailyCollectedEl = document.getElementById('dailyCollected');
const dailyLimitEl = document.getElementById('dailyLimit');
const progressBar = document.getElementById('progressBar');
const resetBtn = document.getElementById('resetBtn');
const exportBtn = document.getElementById('exportBtn');

// --- Utility ---
function clamp(v,min,max){ return Math.max(min,Math.min(max,v)); }
function formatMoney(n){ return Number(n).toLocaleString('en-US',{minimumFractionDigits:2,maximumFractionDigits:2}); }

// --- Game mechanics ---
function updateUI(){
  pointsEl.textContent = Math.floor(state.points).toLocaleString();
  dollarsEl.textContent = '$' + formatMoney(state.dollars);
  storedEl.textContent = `Przechowywane punkty: ${Math.floor(state.points).toLocaleString()} / ${state.capacity.toLocaleString()}`;
  rateLabel.textContent = state.conversionRate.toLocaleString();
  fleetCount.textContent = state.fleet;
  capacityDisplay.textContent = state.capacity.toLocaleString();
  routeDisplay.textContent = state.route.toLocaleString();
  driverCostEl.textContent = '$' + formatMoney(driverCost(state.fleet));
  capacityCostEl.textContent = '$' + formatMoney(capacityCost(state.capacity));
  routeCostEl.textContent = '$' + formatMoney(routeCost(state.route));
  dailyCollectedEl.textContent = Math.floor(state.dailyCollected).toLocaleString();
  dailyLimitEl.textContent = state.route.toLocaleString();
  // progress
  let pct = clamp(state.dailyCollected / state.route * 100, 0, 100);
  progressBar.style.width = pct + '%';
}

// Passive income per second: base 1 point/s per driver (scaled)
function passivePerSecond(){
  // each driver -> 2 points/s initially, and scales slightly
  return state.fleet * 2 * (1 + Math.log10(1 + state.fleet) * 0.1);
}

// Tick: called frequently to add passive points, but respect capacity and daily limit
function tick(){
  const now = Date.now();
  const dt = (now - state.lastTick) / 1000.0;
  state.lastTick = now;
  if(dt <= 0) return;
  // amount to add
  let add = passivePerSecond() * dt;
  // apply daily limit remaining
  const remainToday = Math.max(0, state.route - state.dailyCollected);
  if(remainToday <= 0) add = 0;
  else add = Math.min(add, remainToday);
  // apply capacity limit
  const availableCapacity = Math.max(0, state.capacity - state.points);
  add = Math.min(add, availableCapacity);
  state.points += add;
  state.dailyCollected += add;
  updateUI();
}

// Manual mine (click)
mineBtn.addEventListener('click', ()=>{
  // clicking yields a burst of points, e.g. 50 + small scaling by fleet
  if(state.dailyCollected >= state.route){
    alert('Osiągnięto dzienny limit punktów. Poczekaj do resetu dziennego lub powiększ trasę.');
    return;
  }
  if(state.points >= state.capacity){
    alert('Garaż pełny — zbierz punkty (Collect) lub powiększ garaż.');
    return;
  }
  let base = 50;
  // scale with fleet slightly
  let bonus = state.fleet * 5;
  let gain = Math.min(state.capacity - state.points, Math.min(state.route - state.dailyCollected, base + bonus));
  state.points += gain;
  state.dailyCollected += gain;
  updateUI();
  saveState(state);
});

// Collect -> convert points to dollars
collectBtn.addEventListener('click', ()=>{
  const rate = state.conversionRate;
  if(state.points < rate){
    alert('Za mało punktów, aby zamienić na dolary. Minimalnie ' + rate + ' punktów = $1');
    return;
  }
  // Convert all full chunks
  const chunks = Math.floor(state.points / rate);
  const dollarsGain = chunks * 1.0;
  state.dollars += dollarsGain;
  state.points -= chunks * rate;
  updateUI();
  saveState(state);
  alert('Zamieniono ' + (chunks*rate).toLocaleString() + ' punktów na $' + formatMoney(dollarsGain) + '.');
});

// Kup kierowcę
buyDriverBtn.addEventListener('click', ()=>{
  const cost = Number(driverCost(state.fleet));
  if(state.dollars < cost){
    alert('Za mało dolarów. Spróbuj zebrać więcej punktów i wymienić je na dolary.');
    return;
  }
  state.dollars -= cost;
  state.fleet += 1;
  updateUI();
  saveState(state);
});

// Kup zwiększenie garażu
buyCapacityBtn.addEventListener('click', ()=>{
  const cost = Number(capacityCost(state.capacity));
  if(state.dollars < cost){
    alert('Za mało dolarów.');
    return;
  }
  state.dollars -= cost;
  // Increase capacity by 50% each kupno (rounded)
  state.capacity = Math.round(state.capacity * 1.5);
  updateUI();
  saveState(state);
});

// Kup rozszerzenie trasy (zwiększa dzienny limit)
buyRouteBtn.addEventListener('click', ()=>{
  const cost = Number(routeCost(state.route));
  if(state.dollars < cost){
    alert('Za mało dolarów.');
    return;
  }
  state.dollars -= cost;
  // increase route by 50% each time
  state.route = Math.round(state.route * 1.5);
  updateUI();
  saveState(state);
});

// Reset gry
resetBtn.addEventListener('click', ()=>{
  if(!confirm('Na pewno chcesz zresetować postęp?')) return;
  state = {...DEFAULT};
  saveState(state);
  updateUI();
});

// Eksportuj stan (kopiuj do schowka)
exportBtn.addEventListener('click', async ()=>{
  const text = JSON.stringify(state, null, 2);
  try{
    await navigator.clipboard.writeText(text);
    alert('Stan gry skopiowany do schowka. Możesz go wkleić do pliku jako backup.');
  }catch(e){
    // fallback
    prompt('Oto stan gry (skopiuj ręcznie):', text);
  }
});

// Daily reset logic (resets dailyCollected at midnight local)
function dailyResetIfNeeded(){
  const key = 'automine_last_day';
  const last = localStorage.getItem(key);
  const today = new Date().toDateString();
  if(last !== today){
    state.dailyCollected = 0;
    localStorage.setItem(key, today);
    saveState(state);
  }
}

// Main loop: tick every 1s and save periodically
setInterval(()=>{
  tick();
  // autosave every interval
  saveState(state);
}, 1000);

// initial housekeeping
dailyResetIfNeeded();
updateUI();
saveState(state);

// ensure tick uses correct lastTick after visibility change
document.addEventListener('visibilitychange', ()=>{
  state.lastTick = Date.now();
  saveState(state);
});

// For safety, save before unload
window.addEventListener('beforeunload', ()=> saveState(state));
</script>
</body>
</html>
