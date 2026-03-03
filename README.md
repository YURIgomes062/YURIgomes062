## Hi there 👋

<!--
**YURIgomes062/YURIgomes062** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.


-  I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>PAC-COMMIT</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=VT323:wght@400&display=swap');

  :root {
    --green: #00ff41;
    --dark-green: #003b00;
    --bg: #0a0a0a;
    --amber: #ffb700;
    --red: #ff2d2d;
    --cyan: #00fff5;
    --dim: #005500;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg);
    color: var(--green);
    font-family: 'Share Tech Mono', monospace;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    overflow: hidden;
  }

  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0,255,65,0.03) 2px,
      rgba(0,255,65,0.03) 4px
    );
    pointer-events: none;
    z-index: 100;
  }

  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background: radial-gradient(ellipse at center, transparent 60%, rgba(0,0,0,0.8) 100%);
    pointer-events: none;
    z-index: 99;
  }

  #header {
    font-family: 'VT323', monospace;
    font-size: 3rem;
    color: var(--amber);
    text-shadow: 0 0 20px var(--amber), 0 0 40px rgba(255,183,0,0.4);
    letter-spacing: 8px;
    margin-bottom: 6px;
    animation: flicker 4s infinite;
  }

  #subheader {
    font-size: 0.7rem;
    color: var(--dim);
    letter-spacing: 4px;
    margin-bottom: 24px;
  }

  #game-wrapper {
    position: relative;
    border: 1px solid var(--dark-green);
    box-shadow: 0 0 30px rgba(0,255,65,0.15), inset 0 0 30px rgba(0,0,0,0.5);
    background: #050505;
  }

  canvas {
    display: block;
    image-rendering: pixelated;
  }

  #hud {
    display: flex;
    justify-content: space-between;
    padding: 8px 16px;
    border-top: 1px solid var(--dark-green);
    font-size: 0.75rem;
    color: var(--dim);
  }

  #hud span { color: var(--green); }

  #score-label { color: var(--dim); }
  #score-val { color: var(--amber); font-family: 'VT323', monospace; font-size: 1.1rem; }

  #commit-log {
    margin-top: 16px;
    width: 640px;
    max-height: 80px;
    overflow: hidden;
    border: 1px solid var(--dark-green);
    padding: 6px 12px;
    font-size: 0.65rem;
    color: var(--dim);
    position: relative;
  }

  #commit-log::before {
    content: '$ git log --oneline';
    display: block;
    color: var(--green);
    margin-bottom: 4px;
  }

  .log-entry {
    animation: slideIn 0.3s ease;
    color: var(--green);
  }

  .log-entry.eaten { color: #ff2d2d; text-decoration: line-through; opacity: 0.4; }

  #controls {
    margin-top: 12px;
    font-size: 0.6rem;
    color: var(--dim);
    letter-spacing: 2px;
  }

  @keyframes flicker {
    0%, 95%, 100% { opacity: 1; }
    96% { opacity: 0.8; }
    97% { opacity: 1; }
    98% { opacity: 0.7; }
    99% { opacity: 1; }
  }

  @keyframes slideIn {
    from { transform: translateX(-10px); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
  }
</style>
</head>
<body>

<div id="header">PAC-COMMIT</div>
<div id="subheader">git eat --all --amend</div>

<div id="game-wrapper">
  <canvas id="c" width="640" height="160"></canvas>
  <div id="hud">
    <span id="score-label">COMMITS EATEN: <span id="score-val">0</span></span>
    <span id="branch-label">BRANCH: <span>main</span></span>
    <span id="lives-label">LIVES: <span id="lives-val">███</span></span>
  </div>
</div>

<div id="commit-log">
  <div id="log-lines"></div>
</div>

<div id="controls">[ ARROW KEYS / WASD ] MOVE &nbsp;|&nbsp; [R] RESTART</div>

<script>
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;

const CELL = 20;
const COLS = W / CELL;
const ROWS = H / CELL;

const ADJECTIVES = ['fix','feat','refactor','chore','docs','style','test','perf','ci','build','revert'];
const NOUNS = ['login','auth','api','routes','models','utils','config','tests','db','cache','hooks','types','styles','build','deps'];
const HASH = () => Math.random().toString(16).slice(2,9);

function makeCommit() {
  const adj = ADJECTIVES[Math.random()*ADJECTIVES.length|0];
  const noun = NOUNS[Math.random()*NOUNS.length|0];
  return `${HASH()} ${adj}(${noun}): ${['update','fix','add','remove','refactor'][Math.random()*5|0]} ${noun}`;
}

// Game state
let pacman, commits, particles, score, lives, dead, deadTimer, gameOver, logEntries;

function init() {
  pacman = {
    x: 1.5 * CELL,
    y: H / 2,
    vx: CELL * 0.12,
    vy: 0,
    targetVx: CELL * 0.12,
    targetVy: 0,
    mouth: 0,
    mouthDir: 1,
    mouthMax: 0.35,
  };
  commits = [];
  particles = [];
  score = 0;
  lives = 3;
  dead = false;
  deadTimer = 0;
  gameOver = false;
  logEntries = [];

  // Spawn initial commits
  for (let i = 0; i < 8; i++) spawnCommit(true);

  document.getElementById('score-val').textContent = '0';
  document.getElementById('lives-val').textContent = '███'.slice(0, lives);
  document.getElementById('log-lines').innerHTML = '';
}

function spawnCommit(initial) {
  const lane = Math.floor(Math.random() * (ROWS - 2)) + 1;
  const x = initial
    ? (3 + Math.random() * (COLS - 4)) * CELL
    : (COLS + 1) * CELL;
  const speed = -(0.8 + Math.random() * 1.2);
  commits.push({
    x, y: lane * CELL + CELL / 2,
    vx: speed,
    text: makeCommit(),
    glow: Math.random(),
    type: Math.random() < 0.15 ? 'merge' : Math.random() < 0.1 ? 'revert' : 'commit',
    pulse: Math.random() * Math.PI * 2,
  });
}

function spawnParticles(x, y, color) {
  for (let i = 0; i < 12; i++) {
    const angle = (Math.PI * 2 / 12) * i + Math.random() * 0.3;
    const speed = 1.5 + Math.random() * 3;
    particles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      life: 1,
      decay: 0.04 + Math.random() * 0.04,
      color,
      size: 2 + Math.random() * 3,
      char: ['0','1','#','@','$','*'][Math.random()*6|0],
    });
  }
}

const keys = {};
document.addEventListener('keydown', e => {
  keys[e.key] = true;
  if (['ArrowLeft','ArrowRight','ArrowUp','ArrowDown'].includes(e.key)) e.preventDefault();
  if (e.key === 'r' || e.key === 'R') init();
});
document.addEventListener('keyup', e => { keys[e.key] = false; });

function handleInput() {
  const spd = CELL * 0.13;
  if (keys['ArrowRight'] || keys['d']) { pacman.targetVx = spd; pacman.targetVy = 0; }
  else if (keys['ArrowLeft'] || keys['a']) { pacman.targetVx = -spd; pacman.targetVy = 0; }
  else if (keys['ArrowDown'] || keys['s']) { pacman.targetVx = 0; pacman.targetVy = spd; }
  else if (keys['ArrowUp'] || keys['w']) { pacman.targetVx = 0; pacman.targetVy = -spd; }
}

function drawGrid() {
  ctx.strokeStyle = 'rgba(0,60,0,0.2)';
  ctx.lineWidth = 0.5;
  for (let x = 0; x <= W; x += CELL) {
    ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, H); ctx.stroke();
  }
  for (let y = 0; y <= H; y += CELL) {
    ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
  }
}

function drawPacman() {
  const p = pacman;
  const r = CELL * 0.42;

  // Direction angle
  let facing = 0;
  if (p.vx < 0) facing = Math.PI;
  else if (p.vy > 0) facing = Math.PI / 2;
  else if (p.vy < 0) facing = -Math.PI / 2;

  const mouth = p.mouth;

  if (dead) {
    // Death animation - spinning collapse
    const progress = deadTimer / 60;
    ctx.save();
    ctx.translate(p.x, p.y);
    ctx.rotate(progress * Math.PI * 4);
    ctx.scale(1 - progress * 0.8, 1 - progress * 0.8);
    ctx.shadowBlur = 20;
    ctx.shadowColor = '#ff2d2d';
    ctx.fillStyle = '#ff2d2d';
    ctx.beginPath();
    ctx.arc(0, 0, r, 0, Math.PI * 2);
    ctx.fill();
    ctx.restore();
    return;
  }

  // Glow
  ctx.save();
  ctx.shadowBlur = 18;
  ctx.shadowColor = '#ffb700';

  // Body
  ctx.fillStyle = '#ffb700';
  ctx.beginPath();
  ctx.moveTo(p.x, p.y);
  ctx.arc(p.x, p.y, r, facing + mouth, facing + Math.PI * 2 - mouth);
  ctx.closePath();
  ctx.fill();

  // Eye
  const eyeAngle = facing - Math.PI / 4;
  const ex = p.x + Math.cos(eyeAngle) * r * 0.55;
  const ey = p.y + Math.sin(eyeAngle) * r * 0.55;
  ctx.fillStyle = '#0a0a0a';
  ctx.shadowBlur = 0;
  ctx.beginPath();
  ctx.arc(ex, ey, 2.5, 0, Math.PI * 2);
  ctx.fill();

  ctx.restore();
}

function getCommitColor(type) {
  if (type === 'merge') return '#00fff5';
  if (type === 'revert') return '#ff2d2d';
  return '#00ff41';
}

function drawCommits(t) {
  commits.forEach(c => {
    const col = getCommitColor(c.type);
    const pulse = 0.7 + 0.3 * Math.sin(c.pulse + t * 0.05);

    ctx.save();
    ctx.globalAlpha = pulse;
    ctx.shadowBlur = 12;
    ctx.shadowColor = col;

    // Hash dot
    ctx.fillStyle = col;
    ctx.beginPath();
    ctx.arc(c.x, c.y, 5, 0, Math.PI * 2);
    ctx.fill();

    // Text label
    ctx.font = '9px "Share Tech Mono", monospace';
    ctx.fillStyle = col;
    ctx.globalAlpha = 0.8 * pulse;
    ctx.shadowBlur = 6;
    const label = c.text.slice(0, 30);
    ctx.fillText(label, c.x + 8, c.y + 3);

    // Branch line
    ctx.strokeStyle = col;
    ctx.globalAlpha = 0.2;
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(c.x + 8, c.y);
    ctx.lineTo(c.x + 8 + ctx.measureText(label).width, c.y);
    ctx.stroke();

    ctx.restore();
  });
}

function drawParticles() {
  particles.forEach(p => {
    ctx.save();
    ctx.globalAlpha = p.life;
    ctx.fillStyle = p.color;
    ctx.shadowBlur = 8;
    ctx.shadowColor = p.color;
    ctx.font = `${p.size * 2}px "Share Tech Mono"`;
    ctx.fillText(p.char, p.x, p.y);
    ctx.restore();
  });
}

function addLogEntry(text, eaten) {
  const el = document.createElement('div');
  el.className = 'log-entry' + (eaten ? ' eaten' : '');
  el.textContent = text.slice(0, 50);
  const log = document.getElementById('log-lines');
  log.insertBefore(el, log.firstChild);
  if (log.children.length > 5) log.removeChild(log.lastChild);
}

let t = 0;
function update() {
  if (gameOver) return;
  t++;

  handleInput();

  if (!dead) {
    // Smooth direction change
    pacman.vx += (pacman.targetVx - pacman.vx) * 0.2;
    pacman.vy += (pacman.targetVy - pacman.vy) * 0.2;

    pacman.x += pacman.vx;
    pacman.y += pacman.vy;

    // Wrap around
    if (pacman.x > W + CELL) pacman.x = -CELL;
    if (pacman.x < -CELL) pacman.x = W + CELL;
    if (pacman.y > H + CELL) pacman.y = -CELL;
    if (pacman.y < -CELL) pacman.y = H + CELL;

    // Mouth animation
    pacman.mouth += 0.08 * pacman.mouthDir;
    if (pacman.mouth >= pacman.mouthMax) pacman.mouthDir = -1;
    if (pacman.mouth <= 0.02) pacman.mouthDir = 1;

    // Check eat commits
    commits = commits.filter(c => {
      const dx = pacman.x - c.x;
      const dy = pacman.y - c.y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      if (dist < CELL * 0.7) {
        score++;
        document.getElementById('score-val').textContent = score;
        const col = getCommitColor(c.type);
        spawnParticles(c.x, c.y, col);
        addLogEntry(c.text, true);

        // Revert commits hurt!
        if (c.type === 'revert') {
          lives--;
          updateLives();
          if (lives <= 0) { gameOver = true; showGameOver(); }
        }
        return false;
      }
      return true;
    });
  } else {
    deadTimer++;
    if (deadTimer > 70) {
      dead = false;
      deadTimer = 0;
      pacman.x = 1.5 * CELL;
      pacman.y = H / 2;
      pacman.vx = CELL * 0.12;
      pacman.vy = 0;
      pacman.targetVx = CELL * 0.12;
      pacman.targetVy = 0;
    }
  }

  // Move commits
  commits.forEach(c => {
    c.x += c.vx;
    c.pulse += 0.05;
  });

  // Remove off-screen
  commits = commits.filter(c => c.x > -CELL * 10);

  // Spawn new
  if (t % 90 === 0) spawnCommit(false);
  if (commits.length < 4) spawnCommit(false);

  // Particles
  particles.forEach(p => {
    p.x += p.vx;
    p.y += p.vy;
    p.vy += 0.1;
    p.life -= p.decay;
  });
  particles = particles.filter(p => p.life > 0);
}

function updateLives() {
  const bars = ['', '█', '██', '███'];
  document.getElementById('lives-val').textContent = bars[Math.max(0, lives)];
}

function showGameOver() {
  setTimeout(() => {
    ctx.save();
    ctx.fillStyle = 'rgba(0,0,0,0.7)';
    ctx.fillRect(0, 0, W, H);
    ctx.font = '3rem "VT323", monospace';
    ctx.fillStyle = '#ff2d2d';
    ctx.shadowBlur = 30;
    ctx.shadowColor = '#ff2d2d';
    ctx.textAlign = 'center';
    ctx.fillText('MERGE CONFLICT', W/2, H/2 - 20);
    ctx.font = '1rem "Share Tech Mono", monospace';
    ctx.fillStyle = '#00ff41';
    ctx.shadowColor = '#00ff41';
    ctx.shadowBlur = 10;
    ctx.fillText(`${score} commits eaten  |  [R] to retry`, W/2, H/2 + 20);
    ctx.restore();
  }, 800);
}

function draw() {
  ctx.clearRect(0, 0, W, H);
  ctx.fillStyle = '#050505';
  ctx.fillRect(0, 0, W, H);

  drawGrid();
  drawCommits(t);
  drawParticles();
  drawPacman();

  // Scanline moving
  ctx.save();
  ctx.globalAlpha = 0.04;
  ctx.fillStyle = '#00ff41';
  const scanY = (t * 2) % (H + 4);
  ctx.fillRect(0, scanY - 2, W, 4);
  ctx.restore();
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

init();
loop();
</script>
</body>
</html>
