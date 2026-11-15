<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Space Game Explorer</title>
<style>
:root { box-sizing: border-box; padding-top: env(safe-area-inset-top, 0px); padding-bottom: env(safe-area-inset-bottom, 0px); color-scheme: dark; }
*, *::before, *::after { box-sizing: border-box; }
html, body { margin: 0; height: 100%; background: #070a1f; color: #eef4ff; font-family: system-ui, -apple-system, sans-serif; overflow: hidden; touch-action: none; user-select: none; -webkit-user-select: none; }
canvas { position: fixed; inset: 0; width: 100%; height: 100%; display: block; }
.panel { background: rgba(18, 14, 48, 0.8); border: 1px solid rgba(255, 95, 162, 0.4); border-radius: 14px; padding: 10px 14px; backdrop-filter: blur(4px); }
#hud { position: fixed; top: env(safe-area-inset-top, 0px); left: 0; right: 0; padding: 12px 14px; display: flex; justify-content: space-between; align-items: flex-start; pointer-events: none; font-size: 14px; }
#fuelBar { width: 150px; height: 8px; background: #2a2060; border-radius: 4px; overflow: hidden; margin-top: 6px; }
#fuel { height: 100%; width: 100%; background: linear-gradient(90deg, #ff5fa2, #ffd86b); transition: width 0.1s; }
#info { position: fixed; left: 50%; bottom: 170px; transform: translateX(-50%); width: min(92vw, 380px); display: none; pointer-events: none; }
#info h2 { margin: 0 0 6px; font-size: 18px; color: #ffd86b; }
#info p { margin: 4px 0; font-size: 14px; line-height: 1.45; }
#toast { position: fixed; top: 110px; left: 50%; transform: translateX(-50%); font-size: 15px; font-weight: 600; color: #8ff0ff; opacity: 0; transition: opacity 0.4s; pointer-events: none; white-space: nowrap; }
#controls { position: fixed; bottom: calc(env(safe-area-inset-bottom, 0px) + 12px); left: 0; right: 0; padding: 0 16px; display: flex; justify-content: space-between; align-items: flex-end; pointer-events: none; }
.pad { display: grid; grid-template-columns: repeat(2, 64px); gap: 8px; pointer-events: auto; }
#thrustBtn { width: 100%; height: 64px; grid-column: 1 / -1; }
button { background: rgba(255, 95, 162, 0.15); border: 1px solid rgba(255, 95, 162, 0.55); color: #fff; border-radius: 14px; font-size: 16px; height: 48px; padding: 0 14px; cursor: pointer; touch-action: none; font-family: inherit; }
#leftBtn, #rightBtn, #thrustBtn { font-size: 22px; height: 64px; }
button:active, button.on, button.sel { background: rgba(255, 95, 162, 0.55); transform: scale(0.96); }
button { transition: transform 0.12s ease, background 0.2s ease, box-shadow 0.2s ease; }
button:hover { box-shadow: 0 0 12px rgba(255, 95, 162, 0.45); }
#hud button { pointer-events: auto; height: 36px; padding: 0 12px; font-size: 14px; margin-left: 6px; }
#mini { position: fixed; top: 150px; right: 14px; width: 110px; height: 110px; border-radius: 50%; border: 1px solid rgba(255, 95, 162, 0.5); background: rgba(18, 14, 48, 0.6); pointer-events: none; }
button:disabled { opacity: 0.6; cursor: default; }
.modal { position: fixed; inset: 0; display: flex; align-items: center; justify-content: center; background: rgba(7, 10, 31, 0.88); z-index: 8; padding: 20px; }
.modal .panel { max-width: 420px; width: 100%; text-align: center; padding: 26px 22px; }
.modal h1 { margin: 0 0 10px; font-size: 28px; color: #ffd86b; }
.modal h2 { margin: 0 0 8px; font-size: 20px; color: #ffd86b; }
.modal p { font-size: 15px; line-height: 1.5; margin: 8px 0; }
.modal ul { text-align: left; font-size: 14px; line-height: 1.6; padding-left: 20px; }
.optList { display: flex; flex-direction: column; gap: 8px; margin-top: 12px; }
.optList button { width: 100%; height: auto; min-height: 46px; text-align: left; }
#startBtn, #qNext { margin-top: 14px; width: 100%; height: 52px; font-size: 18px; font-weight: 600; background: rgba(255, 216, 107, 0.25); border-color: #ffd86b; }
#diffRow { display: flex; gap: 8px; margin-top: 12px; }
#diffRow button { flex: 1; }
@media (max-width: 600px) {
  #hud { padding: 8px 10px; font-size: 12px; }
  #fuelBar { width: 110px; }
  #hud .panel { padding: 6px 10px; }
  #hud button { height: 32px; font-size: 13px; padding: 0 9px; }
  #mini { width: 84px; height: 84px; top: 130px; right: 10px; }
  #info { bottom: 150px; width: min(94vw, 360px); }
  #info p { font-size: 13px; }
  .pad { grid-template-columns: repeat(2, 56px); gap: 6px; }
  #leftBtn, #rightBtn, #thrustBtn { height: 56px; font-size: 20px; }
  .modal .panel { max-height: 90vh; overflow-y: auto; padding: 18px 16px; }
  .modal h1 { font-size: 22px; }
}
.hidden { display: none !important; }
</style>
</head>
<body>
<canvas id="c"></canvas>

<div id="hud">
  <div class="panel">
    <div><strong>Discovered:</strong> <span id="count">0</span> / <span id="total">0</span></div>
    <div id="fuelBar"><div id="fuel"></div></div>
  </div>
  <div class="panel" style="text-align:right;font-size:13px;">Stars collected<br><strong id="stars">0</strong><div style="margin-top:8px;display:flex;justify-content:flex-end"><button id="logBtn">Log</button><button id="soundBtn" aria-label="Sound">&#128266;</button>
<button id="pauseBtn" aria-label="Pause">&#10074;&#10074;</button></div></div>
</div>

<div id="toast"></div>

<div id="info" class="panel">
  <h2 id="infoName"></h2>
  <p id="infoFact"></p>
</div>

<div id="controls">
  <div class="pad">
    <button id="leftBtn" aria-label="Turn left">&#8630;</button>
    <button id="rightBtn" aria-label="Turn right">&#8631;</button>
    <button id="thrustBtn" aria-label="Thrust">&#9650; Thrust</button>
  </div>
</div>

<div id="quiz" class="modal hidden">
  <div class="panel">
    <h2 id="qName"></h2>
    <p id="qText"></p>
    <div id="qOpts" class="optList"></div>
    <p id="qRes"></p>
    <button id="qNext" class="hidden">Continue</button>
  </div>
</div>

<canvas id="mini" width="110" height="110"></canvas>

<div id="logModal" class="modal hidden">
  <div class="panel" style="max-width:460px;max-height:80vh;overflow:auto;text-align:left">
    <h2>Discovery Log</h2>
    <div id="logList"></div>
    <button id="logClose" style="margin-top:12px;width:100%">Close</button>
  </div>
</div>

<div id="tutModal" class="modal hidden" style='z-index:11'>
  <div class="panel">
    <h2>How to Play</h2>
    <p id="tutText"></p>
    <div style="display:flex;gap:8px;margin-top:14px">
      <button id="tutPrev" style="flex:1">Back</button>
      <button id="tutNext" style="flex:1">Next</button>
    </div>
  </div>
</div>

<div id="overlay" class="modal">
  <div class="panel">
    <h1>Space Game Explorer</h1>
    <p>Pilot your rocket through the solar system and discover what makes each world special.</p>
    <ul>
      <li>Fly close to a world to discover it, then answer a quick quiz.</li>
      <li>Thrust uses fuel. Collect glowing stars to refuel.</li>
      <li>Find every object to complete your mission!</li>
    </ul>
    <p style="font-size:13px;opacity:.8">Controls: Arrow keys or A/D to turn, Up, W, or Space to thrust. On a phone, use the on-screen buttons.</p>
    <p style="margin-bottom:0"><strong>Difficulty</strong></p>
    <div id="diffRow">
      <button class="dbtn" data-d="easy">Easy</button>
      <button class="dbtn sel" data-d="normal">Normal</button>
      <button class="dbtn" data-d="hard">Hard</button>
    </div>
    <button id="tutBtn" style="margin-top:14px;width:100%;height:46px">How to Play</button>
    <button id="startBtn">Launch Rocket</button>
  </div>
</div>

<script>
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
const $ = id => document.getElementById(id);

const WORLD = 6000, CX = 3000, CY = 3000;
const DIFF = {
  easy:   { cost: 0.07, star: 22, reach: 220 },
  normal: { cost: 0.12, star: 15, reach: 160 },
  hard:   { cost: 0.18, star: 10, reach: 100 }
};
let diff = DIFF.normal;

// Bodies. Planets first so moons can follow their parents.
// q = [question, options, index of correct answer]
const bodies = [
  { name: "The Sun", type: "star", d: 0, a: 0, r: 150, c: "#ffb347", glow: true,
    fact: "The Sun is a star about 109 times wider than Earth. Its light takes about 8 minutes to reach us.",
    q: ["How long does sunlight take to reach Earth?", ["About 8 minutes", "About 8 seconds", "About 8 hours"], 0] },
  { name: "Mercury", d: 450, a: 0.3, r: 26, c: "#c9b8a6",
    fact: "Mercury is the smallest planet. A year there lasts only 88 Earth days.",
    q: ["Which is the smallest planet?", ["Mars", "Mercury", "Venus"], 1] },
  { name: "Venus", d: 700, a: 1.6, r: 40, c: "#ffd27a",
    fact: "Venus is the hottest planet because of its thick carbon dioxide atmosphere. It also spins backwards.",
    q: ["Why is Venus so hot?", ["It is closest to the Sun", "Its thick atmosphere traps heat", "It has many volcanoes"], 1] },
  { name: "Earth", d: 980, a: 2.9, r: 42, c: "#4fb3ff",
    fact: "Earth is the only known planet with liquid water on its surface and life.",
    q: ["What makes Earth special?", ["It has liquid water and life", "It has rings", "It is the largest planet"], 0] },
  { name: "Mars", d: 1250, a: 4.2, r: 32, c: "#ff7a4d",
    fact: "Mars is home to Olympus Mons, the tallest known volcano in the solar system.",
    q: ["What is the Red Planet's tallest volcano called?", ["Olympus Mons", "Mount Everest", "Io Peak"], 0] },
  { name: "Jupiter", d: 1650, a: 5.4, r: 88, c: "#ffb97a", bands: true,
    fact: "Jupiter is the largest planet. Its Great Red Spot is a storm bigger than Earth.",
    q: ["Which planet is the largest?", ["Saturn", "Neptune", "Jupiter"], 2] },
  { name: "Saturn", d: 2100, a: 0.9, r: 76, c: "#f5e0a0", ring: true,
    fact: "Saturn's rings are made mostly of ice and rock. Saturn is so light it would float in water.",
    q: ["What are Saturn's rings made of, mostly?", ["Gas", "Ice and rock", "Metal"], 1] },
  { name: "Uranus", d: 2500, a: 2.2, r: 58, c: "#9ff5ff",
    fact: "Uranus rotates on its side, tilted about 98 degrees compared to most planets.",
    q: ["Why does Uranus look like it rolls around the Sun?", ["It is tilted on its side", "It is very fast", "It has no gravity"], 0] },
  { name: "Neptune", d: 2850, a: 4.0, r: 56, c: "#5a7bff",
    fact: "Neptune has the strongest winds in the solar system, reaching over 1,200 miles per hour.",
    q: ["Which planet has the strongest winds?", ["Earth", "Neptune", "Mercury"], 1] },
  { name: "Moon", parent: "Earth", d: 120, sp: 0.01, a: 0, r: 13, c: "#d8d8e0",
    fact: "The Moon is about a quarter the width of Earth. Its gravity causes ocean tides.",
    q: ["What causes ocean tides on Earth?", ["The Sun's heat", "The Moon's gravity", "Wind"], 1] },
  { name: "Io", parent: "Jupiter", d: 150, sp: 0.02, a: 1, r: 12, c: "#ffe066",
    fact: "Io is the most volcanically active body in the solar system.",
    q: ["Which moon is the most volcanic?", ["Io", "Titan", "The Moon"], 0] },
  { name: "Titan", parent: "Saturn", d: 170, sp: 0.008, a: 2.5, r: 16, c: "#e7a85a",
    fact: "Titan is Saturn's largest moon and has a thick atmosphere, unlike most moons.",
    q: ["Which moon has a thick atmosphere?", ["Titan", "Io", "Phobos"], 0] },
  { name: "Halley's Comet", type: "comet", ax: 1800, ay: 1000, a: 0.5, sp: 0.0012, r: 9, c: "#e8f8ff",
    fact: "Halley's Comet returns about every 76 years. It was last seen in 1986 and returns around 2061.",
    q: ["About how often does Halley's Comet return?", ["Every 76 years", "Every year", "Every 500 years"], 0] }
];
const byName = {};
bodies.forEach(b => byName[b.name] = b);
const TOTAL = bodies.length;

function place(b) {
  if (b.sp) b.a += b.sp;
  if (b.type === "comet") {
    b.x = CX + Math.cos(b.a) * b.ax;
    b.y = CY + Math.sin(b.a) * b.ay;
  } else if (b.parent) {
    const p = byName[b.parent];
    b.x = p.x + Math.cos(b.a) * b.d;
    b.y = p.y + Math.sin(b.a) * b.d;
  } else {
    b.x = CX + Math.cos(b.a) * b.d;
    b.y = CY + Math.sin(b.a) * b.d;
  }
}
bodies.forEach(place);

const stars = [];
for (let i = 0; i < 90; i++) {
  stars.push({ x: 200 + Math.random() * (WORLD - 400), y: 200 + Math.random() * (WORLD - 400), picked: false, tw: Math.random() * 6 });
}
const bgStars = Array.from({ length: 180 }, () => ({ x: Math.random(), y: Math.random(), s: Math.random() * 1.6 + 0.3, t: Math.random() * 6 }));

const rocket = { x: CX + 230, y: CY, vx: 0, vy: 0, a: -Math.PI / 2, av: 0 };
let thrustPower = 0, userPaused = false, logOpen = false;
const cam = { x: rocket.x, y: rocket.y };
const keys = {};
const touch = { left: false, right: false, thrust: false };
const discovered = new Set();
let fuel = 100, starCount = 0, current = null, running = false, paused = false, flameOn = false;
let w = 0, h = 0, dpr = 1, frame = 0, toastTimer = null;

function resize() {
  dpr = window.devicePixelRatio || 1;
  w = window.innerWidth; h = window.innerHeight;
  canvas.width = w * dpr; canvas.height = h * dpr;
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
}
window.addEventListener('resize', resize);
resize();
$('total').textContent = TOTAL;

function toast(msg) {
  const t = $('toast');
  t.textContent = msg; t.style.opacity = 1;
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => t.style.opacity = 0, 1800);
}

function bindHold(el, key) {
  const on = e => { e.preventDefault(); touch[key] = true; el.classList.add('on'); };
  const off = e => { e.preventDefault(); touch[key] = false; el.classList.remove('on'); };
  el.addEventListener('pointerdown', on);
  el.addEventListener('pointerup', off);
  el.addEventListener('pointercancel', off);
  el.addEventListener('pointerleave', off);
}
bindHold($('leftBtn'), 'left');
bindHold($('rightBtn'), 'right');
bindHold($('thrustBtn'), 'thrust');

window.addEventListener('keydown', e => {
  if (['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(e.key)) e.preventDefault();
  keys[e.key.toLowerCase()] = true;
  if (e.key.toLowerCase() === 'p') togglePause();
});
window.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });

document.querySelectorAll('.dbtn').forEach(btn => btn.addEventListener('click', () => {
  document.querySelectorAll('.dbtn').forEach(b => b.classList.remove('sel'));
  btn.classList.add('sel');
  diff = DIFF[btn.dataset.d];
}));

$('startBtn').addEventListener('click', () => {
  $('overlay').classList.add('hidden');
  running = true;
  audioCtx = audioCtx || new (window.AudioContext || window.webkitAudioContext)();
  toast('Mission started! Good luck, explorer.');
  requestAnimationFrame(loop);
});

const extraQ = {
  'The Sun': [['What is the Sun?', ['A planet', 'A star', 'A moon'], 1]],
  'Mercury': [['Which planet is closest to the Sun?', ['Venus', 'Mercury', 'Earth'], 1]],
  'Venus': [['Which planet is the hottest?', ['Mercury', 'Venus', 'Mars'], 1]],
  'Earth': [['How many moons does Earth have?', ['One', 'Two', 'Four'], 0]],
  'Mars': [['What color is Mars often called?', ['Blue', 'Red', 'Green'], 1]],
  'Jupiter': [['Jupiter is mostly made of what?', ['Rock', 'Hydrogen and helium', 'Ice'], 1]],
  'Saturn': [['Which planet is famous for its rings?', ['Saturn', 'Mars', 'Neptune'], 0]],
  'Uranus': [['What color is Uranus?', ['Pale blue-green', 'Bright red', 'Black'], 0]],
  'Neptune': [['Compared to Earth, how far is Neptune from the Sun?', ['Much farther', 'Closer', 'About the same'], 0]],
  'Moon': [['How long does the Moon take to orbit Earth?', ['About 27 days', 'About 1 year', 'About 1 day'], 0]],
  'Io': [['What is special about Io?', ['It is covered in ice', 'It has active volcanoes', 'It is made of water'], 1]],
  'Titan': [['Titan is a moon of which planet?', ['Jupiter', 'Saturn', 'Mars'], 1]],
  "Halley's Comet": [['The Halley comet is made mostly of what?', ['Ice and dust', 'Iron', 'Gold'], 0]]
};

function showQuiz(b) {
  if (!b.q) return;
  paused = true;
  const pool = [b.q, ...(extraQ[b.name] || [])];
  const [question, options, correct] = pool[Math.floor(Math.random() * pool.length)];
  $('qName').textContent = 'Quiz: ' + b.name;
  $('qText').textContent = question;
  $('qRes').textContent = '';
  $('qNext').classList.add('hidden');
  const list = $('qOpts');
  list.innerHTML = '';
  options.forEach((opt, i) => {
    const btn = document.createElement('button');
    btn.textContent = opt;
    btn.addEventListener('click', () => {
      list.querySelectorAll('button').forEach(x => x.disabled = true);
      if (i === correct) {
        fuel = Math.min(100, fuel + 25);
        $('qRes').textContent = 'Correct! +25 fuel';
        playCorrect();
        btn.style.background = 'rgba(120, 255, 170, 0.5)';
      } else {
        $('qRes').textContent = 'Good try! The answer was: ' + options[correct];
        playWrong();
        btn.style.background = 'rgba(255, 90, 90, 0.4)';
      }
      $('qNext').classList.remove('hidden');
    });
    list.appendChild(btn);
  });
  $('quiz').classList.remove('hidden');
}

$('qNext').addEventListener('click', () => {
  $('quiz').classList.add('hidden');
  paused = false;
});

function isLeft()   { return keys['arrowleft'] || keys['a'] || touch.left; }
function isRight()  { return keys['arrowright'] || keys['d'] || touch.right; }
function isThrust() { return keys['arrowup'] || keys['w'] || keys[' '] || touch.thrust; }

function update() {
  bodies.forEach(place);

  const turn = (isRight() ? 1 : 0) - (isLeft() ? 1 : 0);
  rocket.av += (turn * 0.06 - rocket.av) * 0.15;
  rocket.a += rocket.av;

  const target = isThrust() && fuel > 0 ? 1 : 0;
  thrustPower += (target - thrustPower) * 0.12;
  flameOn = thrustPower > 0.05;
  if (flameOn) {
    rocket.vx += Math.cos(rocket.a) * 0.12 * thrustPower;
    rocket.vy += Math.sin(rocket.a) * 0.12 * thrustPower;
    fuel = Math.max(0, fuel - diff.cost * thrustPower);
  }
  rocket.vx *= 0.993; rocket.vy *= 0.993;
  rocket.x += rocket.vx; rocket.y += rocket.vy;

  if (rocket.x < 20 || rocket.x > WORLD - 20) { rocket.vx *= -0.5; rocket.x = Math.min(Math.max(rocket.x, 20), WORLD - 20); }
  if (rocket.y < 20 || rocket.y > WORLD - 20) { rocket.vy *= -0.5; rocket.y = Math.min(Math.max(rocket.y, 20), WORLD - 20); }

  for (const s of stars) {
    if (!s.picked && Math.hypot(s.x - rocket.x, s.y - rocket.y) < 30) {
      s.picked = true; starCount++;
      fuel = Math.min(100, fuel + diff.star);
      toast('+ Fuel! Star collected');
      playBlip();
    }
  }

  let near = null, nearDist = Infinity;
  for (const b of bodies) {
    const d = Math.hypot(b.x - rocket.x, b.y - rocket.y);
    if (d < b.r + diff.reach && d < nearDist) { near = b; nearDist = d; }
  }
  if (near && !discovered.has(near.name)) {
    discovered.add(near.name);
    $('count').textContent = discovered.size;
    toast('Discovered ' + near.name + '!');
    playDiscover();
    if (discovered.size === TOTAL) toast('Mission complete! You found every object!');
    showQuiz(near);
  }
  if (near !== current) {
    current = near;
    if (current) {
      $('infoName').textContent = current.name;
      $('infoFact').textContent = current.fact;
      $('info').style.display = 'block';
    } else {
      $('info').style.display = 'none';
    }
  }
  $('fuel').style.width = fuel + '%';
  $('fuel').style.background = fuel < 25 ? '#ff5a5a' : 'linear-gradient(90deg, #ff5fa2, #ffd86b)';
  $('stars').textContent = starCount;
}

function drawBackground() {
  ctx.fillStyle = '#070a1f';
  ctx.fillRect(0, 0, w, h);
  const n1 = ctx.createRadialGradient(w * 0.2, h * 0.3, 0, w * 0.2, h * 0.3, w * 0.6);
  n1.addColorStop(0, 'rgba(150, 60, 220, 0.18)');
  n1.addColorStop(1, 'rgba(150, 60, 220, 0)');
  ctx.fillStyle = n1; ctx.fillRect(0, 0, w, h);
  const n2 = ctx.createRadialGradient(w * 0.85, h * 0.75, 0, w * 0.85, h * 0.75, w * 0.6);
  n2.addColorStop(0, 'rgba(40, 200, 220, 0.14)');
  n2.addColorStop(1, 'rgba(40, 200, 220, 0)');
  ctx.fillStyle = n2; ctx.fillRect(0, 0, w, h);
  for (const s of bgStars) {
    ctx.globalAlpha = 0.4 + 0.5 * Math.sin(frame * 0.02 + s.t);
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.arc(s.x * w, s.y * h, s.s, 0, Math.PI * 2); ctx.fi
