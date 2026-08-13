<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡ Neon Orb Rush — Cyan Glow Game</title>
<style>
  :root{
    --cyan: #00fff2;
    --cyan-dim: #00b3ad;
    --bg: #050b0c;
    --panel: #0a1416;
  }
  *{ box-sizing:border-box; }
  html,body{
    margin:0; padding:0; height:100%;
    background: radial-gradient(circle at 50% 20%, #0a1a1c 0%, #030607 70%);
    font-family: 'Courier New', monospace;
    color: var(--cyan);
    overflow:hidden;
  }
  #hud{
    position:absolute; top:0; left:0; right:0;
    display:flex; justify-content:space-between; align-items:center;
    padding:14px 24px;
    z-index:10;
    text-shadow: 0 0 8px var(--cyan), 0 0 18px var(--cyan);
    letter-spacing:2px;
  }
  #hud .label{ font-size:12px; color:#7ffff7; opacity:.8; }
  #hud .value{ font-size:26px; font-weight:bold; }
  #timer.warn{ color:#ff3b6a; text-shadow:0 0 10px #ff3b6a; }

  #arena{ position:relative; width:100%; height:100vh; }

  .orb{
    position:absolute;
    width:46px; height:46px;
    border-radius:50%;
    background: radial-gradient(circle at 35% 30%, #ffffff 0%, var(--cyan) 35%, var(--cyan-dim) 70%, transparent 100%);
    box-shadow: 0 0 12px var(--cyan), 0 0 30px var(--cyan), 0 0 60px var(--cyan-dim);
    cursor:pointer;
    animation: pulse 1s ease-in-out infinite, fadeIn .15s ease-out;
    transform-origin:center;
  }
  .orb.bad{
    background: radial-gradient(circle at 35% 30%, #fff 0%, #ff3b6a 35%, #7a0021 70%, transparent 100%);
    box-shadow: 0 0 12px #ff3b6a, 0 0 30px #ff3b6a;
  }
  @keyframes pulse{
    0%,100%{ transform:scale(1); }
    50%{ transform:scale(1.12); }
  }
  @keyframes fadeIn{ from{ opacity:0; transform:scale(.3);} to{ opacity:1; transform:scale(1);} }
  @keyframes pop{ to{ transform:scale(1.8); opacity:0; } }

  #overlay{
    position:absolute; inset:0;
    background:rgba(3,8,9,.92);
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    text-align:center; z-index:20; gap:18px;
  }
  #overlay h1{
    font-size:38px; margin:0;
    text-shadow: 0 0 10px var(--cyan), 0 0 30px var(--cyan), 0 0 60px var(--cyan);
    letter-spacing:4px;
  }
  #overlay p{ color:#8fffee; max-width:420px; line-height:1.6; margin:0 4px; }
  button{
    background:transparent;
    color:var(--cyan);
    border:2px solid var(--cyan);
    padding:12px 32px;
    font-family:inherit;
    font-size:16px;
    letter-spacing:3px;
    cursor:pointer;
    border-radius:6px;
    box-shadow: 0 0 12px var(--cyan-dim);
    transition: all .2s ease;
  }
  button:hover{
    background:var(--cyan);
    color:#00201f;
    box-shadow: 0 0 25px var(--cyan);
  }
  .hidden{ display:none !important; }
  #grid{
    position:absolute; inset:0; z-index:0; opacity:.15;
    background-image:
      linear-gradient(var(--cyan) 1px, transparent 1px),
      linear-gradient(90deg, var(--cyan) 1px, transparent 1px);
    background-size: 40px 40px;
  }
  footer{
    position:absolute; bottom:8px; width:100%; text-align:center;
    font-size:11px; color:#3d8a86; letter-spacing:1px; z-index:10;
  }
</style>
</head>
<body>

<div id="grid"></div>

<div id="hud">
  <div><div class="label">SCORE</div><div class="value" id="score">0</div></div>
  <div><div class="label">WAKTU</div><div class="value" id="timer">30</div></div>
  <div><div class="label">HIGH SCORE</div><div class="value" id="high">0</div></div>
</div>

<div id="arena"></div>

<div id="overlay">
  <h1 id="overlayTitle">⚡ NEON ORB RUSH ⚡</h1>
  <p>Klik bola <b>cyan</b> secepat mungkin untuk dapat poin. Awas bola <b style="color:#ff3b6a">merah</b> — klik itu malah mengurangi skor! Kamu punya 30 detik.</p>
  <button id="startBtn">▶ MULAI</button>
</div>

<footer>Made with cyan glow · dibuat untuk profil GitHub</footer>

<script>
  const arena = document.getElementById('arena');
  const scoreEl = document.getElementById('score');
  const timerEl = document.getElementById('timer');
  const highEl = document.getElementById('high');
  const overlay = document.getElementById('overlay');
  const overlayTitle = document.getElementById('overlayTitle');
  const startBtn = document.getElementById('startBtn');

  let score = 0, timeLeft = 30, spawnLoop, tickLoop, playing = false;
  let high = Number(localStorage.getItem('neonOrbHigh') || 0);
  highEl.textContent = high;

  function spawnOrb(){
    if(!playing) return;
    const orb = document.createElement('div');
    const isBad = Math.random() < 0.22;
    orb.className = 'orb' + (isBad ? ' bad' : '');
    const size = 46;
    const maxX = arena.clientWidth - size;
    const maxY = arena.clientHeight - size - 60;
    orb.style.left = (Math.random() * maxX) + 'px';
    orb.style.top = (60 + Math.random() * maxY) + 'px';

    const life = setTimeout(() => orb.remove(), 1300);

    orb.addEventListener('click', () => {
      clearTimeout(life);
      score += isBad ? -5 : 10;
      if(score < 0) score = 0;
      scoreEl.textContent = score;
      orb.style.animation = 'pop .25s forwards';
      setTimeout(() => orb.remove(), 200);
    });

    arena.appendChild(orb);
  }

  function startGame(){
    score = 0; timeLeft = 30; playing = true;
    scoreEl.textContent = score;
    timerEl.textContent = timeLeft;
    timerEl.classList.remove('warn');
    overlay.classList.add('hidden');
    arena.innerHTML = '';

    spawnLoop = setInterval(spawnOrb, 550);
    tickLoop = setInterval(() => {
      timeLeft--;
      timerEl.textContent = timeLeft;
      if(timeLeft <= 10) timerEl.classList.add('warn');
      if(timeLeft <= 0) endGame();
    }, 1000);
  }

  function endGame(){
    playing = false;
    clearInterval(spawnLoop);
    clearInterval(tickLoop);
    arena.innerHTML = '';
    if(score > high){
      high = score;
      localStorage.setItem('neonOrbHigh', high);
      highEl.textContent = high;
    }
    overlayTitle.textContent = '⚡ SKOR: ' + score + ' ⚡';
    startBtn.textContent = '↻ MAIN LAGI';
    overlay.classList.remove('hidden');
  }

  startBtn.addEventListener('click', startGame);
</script>
</body>
</html>
