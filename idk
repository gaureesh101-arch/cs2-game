<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>Tactical FPS Demo — browser</title>
<meta name="viewport" content="width=device-width,initial-scale=1" />
<style>
  :root {
    --bg:#0f1720; --panel:#0b1220; --accent:#22c1c3; --muted:#9aa6b2;
  }
  html,body { height:100%; margin:0; font-family:Inter,system-ui,Segoe UI,Roboto,Arial; background:var(--bg); color:#e6eef6; }
  #wrapper { display:flex; gap:12px; height:100%; padding:12px; box-sizing:border-box; align-items:stretch; }
  #game-panel { flex:1; position:relative; border-radius:10px; overflow:hidden; background:linear-gradient(180deg,#061017 0%, #071623 100%); box-shadow:0 8px 30px rgba(0,0,0,0.6); }
  canvas { display:block; width:100%; height:100%; background:#15202b; }
  #hud { position:absolute; left:12px; bottom:12px; display:flex; gap:10px; align-items:center; pointer-events:none; }
  .stat { background:rgba(0,0,0,0.35); padding:8px 10px; border-radius:8px; font-size:14px; color:var(--muted); }
  .hp { background:linear-gradient(90deg,#ff4b4b,#ffb86b); color:#051017; font-weight:700; }
  #crosshair { position:absolute; left:50%; top:50%; width:32px; height:32px; transform:translate(-50%,-50%); pointer-events:none; mix-blend-mode:screen; }
  #panel-right { width:360px; display:flex; flex-direction:column; gap:12px; }
  .card { background:var(--panel); border-radius:10px; padding:12px; box-shadow:0 6px 18px rgba(0,0,0,0.6); }
  h3 { margin:0 0 8px 0; font-size:16px; color:var(--accent); }
  .controls { font-size:13px; color:var(--muted); line-height:1.6; }
  button { background:transparent; color:var(--accent); border:1px solid rgba(34,193,195,0.12); padding:8px 10px; border-radius:8px; cursor:pointer; }
  .mini-map { width:100%; height:170px; background:linear-gradient(180deg,#061017,#0b1b24); border-radius:8px; position:relative; overflow:hidden; }
  .footer { font-size:12px; color:var(--muted); margin-top:8px; }
  .center-tip { position:absolute; left:50%; top:12px; transform:translateX(-50%); background:rgba(0,0,0,0.4); padding:6px 10px; border-radius:8px; color:var(--muted); font-size:13px; }
</style>
</head>
<body>
<div id="wrapper">
  <div id="game-panel" class="card">
    <div class="center-tip">Click to play — pointer locked. WASD move • Mouse aim • Click to shoot</div>
    <canvas id="view" width="1280" height="720"></canvas>

    <div id="crosshair">
      <svg width="32" height="32" viewBox="0 0 32 32" aria-hidden>
        <circle cx="16" cy="16" r="1.6" fill="#e6eef6" opacity="0.95"/>
        <rect x="15" y="2" width="2" height="6" rx="1" fill="#e6eef6" opacity="0.85"/>
        <rect x="15" y="24" width="2" height="6" rx="1" fill="#e6eef6" opacity="0.85"/>
        <rect x="2" y="15" width="6" height="2" rx="1" fill="#e6eef6" opacity="0.85"/>
        <rect x="24" y="15" width="6" height="2" rx="1" fill="#e6eef6" opacity="0.85"/>
      </svg>
    </div>

    <div id="hud">
      <div class="stat hp" id="hp">HP 100</div>
      <div class="stat" id="ammo">AK | 30 / 90</div>
      <div class="stat" id="weapon">Weapon: Assault</div>
      <div class="stat" id="score">Kills: 0</div>
    </div>
  </div>

  <div id="panel-right">
    <div class="card">
      <h3>Controls</h3>
      <div class="controls">
        <strong>Move:</strong> W A S D • <strong>Jump:</strong> Space • <strong>Aim/Look:</strong> Mouse (pointer lock)<br>
        <strong>Shoot:</strong> Left click • <strong>Reload:</strong> R • <strong>Switch:</strong> 1 / 2
      </div>
    </div>

    <div class="card">
      <h3>Mini-map</h3>
      <div class="mini-map" id="minimap"></div>
      <div class="footer">Green = you • Red = enemies • Gray = walls</div>
    </div>

    <div class="card">
      <h3>Info</h3>
      <div class="controls">
        This is a tiny original tactical-FPS demo (movement, recoil, hitscan shooting, basic AI). No external assets used — simple shapes and generated audio.
        <div style="margin-top:8px;"><button id="resetBtn">Reset Spawn</button></div>
      </div>
    </div>
  </div>
</div>

<script>
/* Tactical FPS Demo — single-file
   - lightweight 2.5D feel: player moves in 2D world, view rendered as first-person "viewport" (floor + sprites)
   - hitscan shooting (ray from center), recoil, two weapons, enemies are rectangles in world.
   - Uses pointer lock for mouse look.
*/

// -------- Config & world ----------
const canvas = document.getElementById('view');
const ctx = canvas.getContext('2d');
const minimap = document.getElementById('minimap');
const mmctx = minimap.getContext?.('2d') || null;

const hudHP = document.getElementById('hp');
const hudAmmo = document.getElementById('ammo');
const hudWeapon = document.getElementById('weapon');
const hudScore = document.getElementById('score');
const resetBtn = document.getElementById('resetBtn');

const W = canvas.width, H = canvas.height;
const HALF_W = W/2, HALF_H = H/2;

const world = {
  size: 1400,
  walls: [
    // rectangles: x,y,w,h
    {x:100,y:100,w:1200,h:10},
    {x:100,y:1290,w:1200,h:10},
    {x:100,y:100,w:10,h:1200},
    {x:1290,y:100,w:10,h:1200},
    // inner obstacles
    {x:300,y:350,w:200,h:20},
    {x:600,y:600,w:20,h:400},
    {x:820,y:300,w:100,h:20},
    {x:1000,y:800,w:200,h:20},
  ],
  enemies: []
};

// player state
const player = {
  x: 700, y: 900, z: 0, // z for jump
  rot: 0, // radians (0 = right)
  pitch: 0, // not used heavily
  speed: 220, // px/sec
  sprint: 1.0,
  radius: 14,
  hp: 100,
  ammoInv: 90,
  kills: 0
};

// weapons
const weapons = {
  1: { name: 'Assault', clipMax:30, clip:30, dmg:28, rpm:600, recoil:6, spread:0.9 },
  2: { name: 'Pistol', clipMax:15, clip:15, dmg:18, rpm:400, recoil:3, spread:2.2 }
};
let currentWeapon = 1;
let lastShot = 0;
let firing = false;

const bullets = []; // for visual trails
const particles = []; // for impact effects

// enemies generator
function spawnEnemies(n=5){
  world.enemies = [];
  for(let i=0;i<n;i++){
    const e = {
      id: i+1,
      x: 220 + Math.random()*880,
      y: 220 + Math.random()*880,
      w: 28, h:28,
      hp: 40,
      speed: 70 + Math.random()*50,
      lastMove: performance.now(),
      dir: Math.random()*Math.PI*2,
      alive: true
    };
    world.enemies.push(e);
  }
}
spawnEnemies(6);

// pointer lock + mouse control
let pointerLocked = false;
const sensitivity = 0.0026;
canvas.requestPointerLock = canvas.requestPointerLock || canvas.mozRequestPointerLock;
document.exitPointerLock = document.exitPointerLock || document.mozExitPointerLock;

canvas.addEventListener('click', ()=> {
  if(!pointerLocked) canvas.requestPointerLock();
});
document.addEventListener('pointerlockchange', lockChange, false);
document.addEventListener('mozpointerlockchange', lockChange, false);
function lockChange(){
  pointerLocked = (document.pointerLockElement === canvas || document.mozPointerLockElement === canvas);
}

// mouse look
document.addEventListener('mousemove', (e)=>{
  if(!pointerLocked) return;
  player.rot += e.movementX * sensitivity;
  player.pitch += e.movementY * sensitivity * 0.3;
  if(player.pitch > 0.6) player.pitch = 0.6;
  if(player.pitch < -0.6) player.pitch = -0.6;
});

// controls
const keys = {};
document.addEventListener('keydown', e=> { keys[e.key.toLowerCase()] = true; if(e.key==='1') switchWeapon(1); if(e.key==='2') switchWeapon(2); });
document.addEventListener('keyup', e=> { keys[e.key.toLowerCase()] = false; });

canvas.addEventListener('mousedown', e=>{
  if(e.button===0) firing = true;
});
canvas.addEventListener('mouseup', e=>{
  if(e.button===0) firing = false;
});

// simple audio helper (beeps)
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
function tone(freq, time=0.06, type='sine'){
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = type; o.frequency.value = freq;
  g.gain.value = 0.0001;
  o.connect(g); g.connect(audioCtx.destination);
  o.start();
  g.gain.exponentialRampToValueAtTime(0.12, audioCtx.currentTime + 0.01);
  g.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + time);
  o.stop(audioCtx.currentTime + time + 0.01);
}

// simple collision helpers
function rectsOverlap(a,b){
  return !(a.x+b.w < b.x || a.x > b.x + b.w || a.y + b.h < b.y || a.y > b.y + b.h);
}

function circleRectCollision(cx,cy,r,rect){
  const nearestX = Math.max(rect.x, Math.min(cx, rect.x+rect.w));
  const nearestY = Math.max(rect.y, Math.min(cy, rect.y+rect.h));
  const dx = cx - nearestX, dy = cy - nearestY;
  return (dx*dx + dy*dy) < r*r;
}

// move player with collision
function movePlayer(dx, dy, dt){
  const nx = player.x + dx;
  const ny = player.y + dy;
  const pr = player.radius;
  // check world walls
  for(const w of world.walls){
    if(circleRectCollision(nx, ny, pr+1, w)) {
      // attempt axis-separated movement to slide
      if(!circleRectCollision(player.x, ny, pr+1, w)) { player.x = player.x; player.y = ny; return; }
      if(!circleRectCollision(nx, player.y, pr+1, w)) { player.x = nx; player.y = player.y; return; }
      return; // blocked
    }
  }
  // bounds
  if(nx < 20 || nx > world.size-20 || ny < 20 || ny > world.size-20) return;
  player.x = nx; player.y = ny;
}

// shooting logic (hitscan)
function shoot(){
  const w = weapons[currentWeapon];
  const now = performance.now();
  const msPerShot = 60000 / w.rpm;
  if(now - lastShot < msPerShot) return;
  if(w.clip <= 0){ tone(240,0.08,'sawtooth'); return; }
  lastShot = now;
  w.clip -= 1;

  // recoil: small random angle offset
  const spread = (Math.random() - 0.5) * w.spread * 0.01 + (Math.random() - 0.5) * 0.01;
  const angle = player.rot + (Math.random()-0.5)*(w.recoil*0.001) + spread;
  // create visual bullet trail
  bullets.push({x:player.x, y:player.y, angle, t:0, max:1.0, speed:1800});

  // perform hitscan ray against enemies
  // ray param: p = player + t*(cos,sin)
  const dx = Math.cos(angle), dy = Math.sin(angle);
  const maxDist = 1400;
  let hit = null;
  for(const e of world.enemies){
    if(!e.alive) continue;
    // approximate enemy as circle
    const ex = e.x + e.w/2, ey = e.y + e.h/2;
    // project centre onto ray
    const toEx = ex - player.x, toEy = ey - player.y;
    const proj = toEx*dx + toEy*dy;
    if(proj < 0 || proj > maxDist) continue;
    // distance from center to ray
    const closestX = player.x + dx*proj;
    const closestY = player.y + dy*proj;
    const distSq = (closestX-ex)*(closestX-ex) + (closestY-ey)*(closestY-ey);
    const radius = Math.max(e.w,e.h)/2;
    if(distSq <= radius*radius){
      // determine exact t distance
      const dist = Math.sqrt((player.x-ex)*(player.x-ex)+(player.y-ey)*(player.y-ey));
      if(!hit || proj < hit.proj){
        hit = { enemy: e, proj };
      }
    }
  }
  if(hit){
    // apply damage
    hit.enemy.hp -= w.dmg;
    // small hit particle
    particles.push({x: hit.enemy.x + hit.enemy.w/2, y: hit.enemy.y + hit.enemy.h/2, life:400});
    tone(1200, 0.04, 'triangle');
    if(hit.enemy.hp <= 0){
      hit.enemy.alive = false;
      player.kills++;
      hudScore.textContent = 'Kills: '+player.kills;
      tone(900, 0.12, 'sine');
    }
  } else {
    // wall hit (or miss)
    tone(600, 0.04, 'sine');
  }
  updateHUD();
}

// reload
function reload(){
  const w = weapons[currentWeapon];
  if(w.clip >= w.clipMax || player.ammoInv <= 0) return;
  const need = w.clipMax - w.clip;
  const take = Math.min(need, player.ammoInv);
  w.clip += take;
  player.ammoInv -= take;
  tone(420, 0.08, 'sine');
  updateHUD();
}

function switchWeapon(which){
  if(!weapons[which]) return;
  currentWeapon = which;
  updateHUD();
  tone(800,0.06,'square');
}

function updateHUD(){
  const w = weapons[currentWeapon];
  hudHP.textContent = 'HP ' + Math.max(0,Math.round(player.hp));
  hudAmmo.textContent = `${w.name} | ${w.clip} / ${player.ammoInv}`;
  hudWeapon.textContent = 'Weapon: ' + w.name;
}

// enemy simple AI: wander and chase if near
function updateEnemies(dt){
  for(const e of world.enemies){
    if(!e.alive) continue;
    const dx = player.x - e.x, dy = player.y - e.y;
    const dist = Math.hypot(dx,dy);
    if(dist < 240){
      // chase
      e.dir = Math.atan2(dy,dx);
      const nx = e.x + Math.cos(e.dir) * e.speed * dt;
      const ny = e.y + Math.sin(e.dir) * e.speed * dt;
      // wall collision check (simple)
      let blocked = false;
      for(const w of world.walls){
        if(circleRectCollision(nx, ny, e.w/2, w)) { blocked = true; break; }
      }
      if(!blocked){ e.x = nx; e.y = ny; }
      // if very close, damage player
      if(dist < 28 && (Math.random() < 0.02)) {
        player.hp -= 6;
        tone(200,0.06,'sawtooth');
        if(player.hp <= 0){ player.hp = 0; }
      }
    } else {
      // wander
      if(Math.random() < 0.01) e.dir += (Math.random()-0.5)*1.4;
      e.x += Math.cos(e.dir) * e.speed * 0.4 * dt;
      e.y += Math.sin(e.dir) * e.speed * 0.4 * dt;
      // keep inside
      e.x = Math.max(120, Math.min(world.size-120, e.x));
      e.y = Math.max(120, Math.min(world.size-120, e.y));
    }
  }
}

// render world from player perspective (simple)
function renderScene(){
  // sky & floor bands for simple 3D feeling
  const skyGrad = ctx.createLinearGradient(0,0,0,HALF_H);
  skyGrad.addColorStop(0,'#102033'); skyGrad.addColorStop(1,'#0b1c28');
  ctx.fillStyle = skyGrad; ctx.fillRect(0,0,W,HALF_H);

  const floorGrad = ctx.createLinearGradient(0,HALF_H,0,H);
  floorGrad.addColorStop(0,'#0b1218'); floorGrad.addColorStop(1,'#081016');
  ctx.fillStyle = floorGrad; ctx.fillRect(0,HALF_H,W,HALF_H);

  // simple center vignette
  ctx.save();
  ctx.globalAlpha = 0.12;
  ctx.fillStyle = '#000'; ctx.fillRect(0,0,W,H);
  ctx.restore();

  // draw weapon (2D sprite-like) anchored to bottom center
  const w = weapons[currentWeapon];
  drawWeaponFrame(w);
  // draw muzzle flashes
  for(const b of bullets){
    // small bright line at center representing bullet trail into distance
    if(b.t < 0.06){
      const progress = Math.min(1, b.t / 0.06);
      ctx.save();
      ctx.translate(HALF_W, HALF_H + player.pitch*40);
      ctx.rotate( (b.angle - player.rot) );
      ctx.fillStyle = 'rgba(255,230,180,' + (0.9 - progress) +')';
      ctx.fillRect(20, -2, 220* (1-progress), 3);
      ctx.restore();
    }
  }

  // draw simple hit markers (particles)
  for(const p of particles){
    const alpha = Math.max(0, p.life/400);
    ctx.fillStyle = `rgba(255,150,50,${alpha})`;
    const sx = HALF_W + (p.x - player.x) * 0.02;
    const sy = HALF_H + (p.y - player.y) * 0.02;
    ctx.beginPath(); ctx.arc(sx, sy, 6*alpha, 0, Math.PI*2); ctx.fill();
  }
}

// draw simple weapon at bottom center
function drawWeaponFrame(weapon){
  const baseW = 420, baseH = 220;
  const x = (W - baseW)/2, y = H - baseH;
  ctx.save();
  // weapon shadow panel
  ctx.translate(0,0);
  // body (rectangular stylized)
  ctx.fillStyle = '#081016';
  ctx.fillRect(x+baseW*0.18, y+baseH*0.48, baseW*0.64, baseH*0.36);
  ctx.fillStyle = '#1f2933';
  ctx.fillRect(x+baseW*0.25, y+baseH*0.5, baseW*0.5, baseH*0.28);
  // muzzle
  ctx.fillStyle = '#222';
  ctx.fillRect(x+baseW*0.72, y+baseH*0.54, baseW*0.06, baseH*0.08);
  // simple iron sight
  ctx.fillStyle = '#e6eef6';
  ctx.fillRect(HALF_W-3, y+20, 6, 26);
  ctx.restore();
}

// minimap rendering
function renderMinimap(){
  if(!mmctx) return;
  const mmW = minimap.clientWidth, mmH = minimap.clientHeight;
  mmctx.clearRect(0,0,mmW,mmH);
  mmctx.fillStyle = '#071018'; mmctx.fillRect(0,0,mmW,mmH);
  const scale = 0.22;
  // walls
  mmctx.fillStyle = '#9aa6b2';
  for(const w of world.walls){
    mmctx.fillRect(w.x*scale, w.y*scale, w.w*scale, w.h*scale);
  }
  // enemies
  for(const e of world.enemies){
    mmctx.fillStyle = e.alive ? '#ff6464' : 'rgba(255,255,255,0.12)';
    mmctx.fillRect(e.x*scale - 4, e.y*scale - 4, 8, 8);
  }
  // player
  mmctx.fillStyle = '#22c1c3';
  mmctx.beginPath(); mmctx.arc(player.x*scale, player.y*scale, 6, 0, Math.PI*2); mmctx.fill();
  // player view cone
  mmctx.strokeStyle = 'rgba(34,193,195,0.2)';
  mmctx.beginPath();
  mmctx.moveTo(player.x*scale, player.y*scale);
  mmctx.lineTo((player.x + Math.cos(player.rot)*70)*scale, (player.y + Math.sin(player.rot)*70)*scale);
  mmctx.stroke();
}

// update loop
let last = performance.now();
function loop(now){
  const dt = Math.min(0.04, (now - last) / 1000);
  last = now;

  // movement
  let vx = 0, vy = 0;
  if(keys['w']) { vx += Math.cos(player.rot) * player.speed * dt; vy += Math.sin(player.rot) * player.speed * dt; }
  if(keys['s']) { vx -= Math.cos(player.rot) * player.speed * dt; vy -= Math.sin(player.rot) * player.speed * dt; }
  if(keys['a']) { vx += Math.cos(player.rot - Math.PI/2) * player.speed * dt; vy += Math.sin(player.rot - Math.PI/2) * player.speed * dt; }
  if(keys['d']) { vx += Math.cos(player.rot + Math.PI/2) * player.speed * dt; vy += Math.sin(player.rot + Math.PI/2) * player.speed * dt; }

  movePlayer(vx, vy, dt);

  // firing
  if(firing) shoot();

  // update bullets (visual)
  for(let i=bullets.length-1;i>=0;i--){
    const b = bullets[i];
    b.t += dt;
    if(b.t > b.max) bullets.splice(i,1);
  }
  // particles
  for(let i=particles.length-1;i>=0;i--){
    const p = particles[i];
    p.life -= dt*1000;
    if(p.life <= 0) particles.splice(i,1);
  }

  // enemies update
  updateEnemies(dt);

  // render
  ctx.clearRect(0,0,W,H);
  renderScene();
  renderMinimap();

  // HUD update periodically
  hudHP.textContent = 'HP ' + Math.max(0,Math.round(player.hp));
  const cw = weapons[currentWeapon];
  hudAmmo.textContent = `${cw.name} | ${cw.clip} / ${player.ammoInv}`;

  requestAnimationFrame(loop);
}

updateHUD();
requestAnimationFrame(loop);

// input actions
document.addEventListener('keydown', e=>{
  if(e.key.toLowerCase()==='r'){ reload(); }
  if(e.code === 'Space') { /* simple jump impulse; not visualized much */ player.z = 1; }
});
resetBtn.addEventListener('click', ()=> {
  player.x = 700; player.y = 900; player.hp = 100; player.kills = 0; player.ammoInv = 90;
  for(const k in weapons) weapons[k].clip = weapons[k].clipMax;
  spawnEnemies(6);
  updateHUD();
});

// resume audio on user gesture
document.addEventListener('click', ()=> {
  if(audioCtx.state === 'suspended') audioCtx.resume();
});
</script>
</body>
</html>
