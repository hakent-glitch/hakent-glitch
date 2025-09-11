<h1 align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&weight=700&size=40&duration=3000&pause=1000&color=00F7FF&center=true&vCenter=true&width=600&height=60&lines=hakent;Creative+Coder;Open+Source+Lover;Always+Learning+%F0%9F%9A%80" alt="Typing SVG" />
</h1>

<p align="center">
  <img src="https://raw.githubusercontent.com/rodrigograca31/rodrigograca31/master/matrix.svg" alt="Matrix animation" />
</p>

---

### 🚀 About Me  
- 💻 Passionate about coding & open-source  
- 🌌 Exploring **Web Dev | AI | Creative Coding**  
- ⚡ Motto: *"Keep creating, keep evolving."*  

---

### 🌐 Connect with Me  
<p align="center">
  <a href="https://github.com/hakent-glitch/hakent-glitch">
    <img src="https://img.shields.io/badge/-GitHub-181717?style=for-the-badge&logo=github&logoColor=white" />
  </a>
  <a href="https://instagram.com/hakentkh">
    <img src="https://img.shields.io/badge/-Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white" />
  </a>
  <a href="https://www.tiktok.com/@kentt271" target="_blank">
  <img src="https://img.shields.io/badge/TikTok-%23000000.svg?style=for-the-badge&logo=tiktok&logoColor=white" alt="TikTok Badge"/>
</a>


<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Pac‑Man — Simple HTML5 Game</title>
  <style>
    :root{--bg:#000;--wall:#0b4;--pellet:#ffd700;--pac:#ffeb3b;--ghost1:#ff5252;--ghost2:#448aff}
    html,body{height:100%;margin:0;background:var(--bg);display:flex;align-items:center;justify-content:center;font-family:system-ui,Arial}
    canvas{border:10px solid #111;border-radius:8px;display:block}
    .info{color:#eee;margin-top:12px;text-align:center}
    .controls{margin-top:8px;color:#bbb;font-size:14px}
  </style>
</head>
<body>
  <div>
    <canvas id="game" width="560" height="620" aria-label="Pac-Man game"></canvas>
    <div class="info">
      <div id="score">Score: 0</div>
      <div id="msg" style="color:#ffeb3b;margin-top:6px"></div>
      <div class="controls">Arrow keys / WASD to move. Refresh to restart.</div>
    </div>
  </div>

<script>
// --- Simple Pac-Man clone (small, self-contained) ---
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const TILE = 20; // grid tile size
const COLS = 28; // classic-ish layout width
const ROWS = 31; // height
const OFFSET_X = 0;
const OFFSET_Y = 0;
let scoreEl = document.getElementById('score');
let msgEl = document.getElementById('msg');

// Map legend: 0 empty, 1 wall, 2 pellet, 3 power pellet
// A small hand-made map (28x31) simplified. We'll generate walls around and some corridors.
// For brevity create a simple maze programmatically with outer walls and inner blocks + pellets.
let map = [];
function initMap(){
  map = new Array(ROWS).fill(0).map(()=>new Array(COLS).fill(2));
  // outer walls
  for(let r=0;r<ROWS;r++){
    for(let c=0;c<COLS;c++){
      if(r===0||r===ROWS-1||c===0||c===COLS-1) map[r][c]=1;
    }
  }
  // add some blocks
  for(let r=2;r<ROWS-2;r+=4){
    for(let c=2;c<COLS-2;c+=6){
      for(let rr=0;rr<2;rr++) for(let cc=0;cc<4;cc++) map[r+rr][c+cc]=1;
    }
  }
  // corridors and clear center
  for(let r=12;r<19;r++) for(let c=8;c<20;c++) map[r][c]=0;
  // place power pellets (corners inside)
  map[3][3]=3; map[3][COLS-4]=3; map[ROWS-4][3]=3; map[ROWS-4][COLS-4]=3;
  // ensure pellets not on walls
  for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++) if(map[r][c]===0) map[r][c]=0; // keep empty
}

initMap();

// Entities
let pac = {r:ROWS-3,c:2,dir:{r:0,c:0},nextDir:{r:0,c:0},speed:6, mouth:0, alive:true};
let ghosts = [
  {r:14,c:13,dir:{r:0,c:1},color:'#ff5252',fright:false},
  {r:14,c:14,dir:{r:0,c:-1},color:'#448aff',fright:false}
];
let score = 0; let pelletsTotal = 0;

function countPellets(){ pelletsTotal=0; for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++) if(map[r][c]===2||map[r][c]===3) pelletsTotal++; }
countPellets();

// Helpers
function tileToXY(r,c){ return {x:c*TILE+TILE/2+OFFSET_X, y:r*TILE+TILE/2+OFFSET_Y}; }
function isWall(r,c){ if(r<0||c<0||r>=ROWS||c>=COLS) return true; return map[r][c]===1; }

// Input
const keyMap = {
  ArrowUp:[-1,0], ArrowDown:[1,0], ArrowLeft:[0,-1], ArrowRight:[0,1],
  w:[-1,0], s:[1,0], a:[0,-1], d:[0,1]
};
window.addEventListener('keydown',e=>{
  const k = e.key;
  if(keyMap[k]){
    const [dr,dc]=keyMap[k];
    pac.nextDir={r:dr,c:dc};
    e.preventDefault();
  }
});

// Movement & logic in tile-space with interpolation
let lastTime=0;
function update(dt){
  if(!pac.alive) return;
  // try to turn
  if(pac.nextDir){
    let nr=pac.r+pac.nextDir.r;
    let nc=pac.c+pac.nextDir.c;
    if(!isWall(nr,nc)) pac.dir = {...pac.nextDir};
  }
  // move pac
  let targetR = pac.r + pac.dir.r;
  let targetC = pac.c + pac.dir.c;
  if(!isWall(targetR,targetC)){
    // move stepwise based on speed and dt
    // here simply move by partial tiles per frame
    // For simplicity jump tile by tile per interval
    moveEntity(pac, pac.dir);
  }
  // ghosts simple AI: random turns not into walls
  for(let g of ghosts){
    if(Math.random()<0.02) {
      // pick new dir
      const dirs=[{r:-1,c:0},{r:1,c:0},{r:0,c:-1},{r:0,c:1}];
      for(let i=0;i<10;i++){
        let d = dirs[Math.floor(Math.random()*4)];
        if(!isWall(g.r+d.r,g.c+d.c)) { g.dir=d; break; }
      }
    }
    moveEntity(g,g.dir);
  }
  // collisions with ghosts
  for(let g of ghosts){
    if(g.r===pac.r&&g.c===pac.c){
      pac.alive=false; msgEl.textContent='Game Over — Press Refresh to restart';
    }
  }
}

function moveEntity(ent, dir){
  const nr = ent.r + dir.r;
  const nc = ent.c + dir.c;
  if(!isWall(nr,nc)){
    ent.r = nr; ent.c = nc;
    // pellet collision for pac
    if(ent===pac){
      const val = map[ent.r][ent.c];
      if(val===2){ score+=10; map[ent.r][ent.c]=0; pelletsTotal--; scoreEl.textContent='Score: '+score; }
      if(val===3){ score+=50; map[ent.r][ent.c]=0; pelletsTotal--; scoreEl.textContent='Score: '+score; activateFright(); }
      if(pelletsTotal<=0){ msgEl.textContent='You Win! 🎉'; pac.alive=false; }
    }
  } else {
    // stop if wall
    if(ent===pac) ent.dir={r:0,c:0};
  }
}

function activateFright(){ for(let g of ghosts) g.fright=true; setTimeout(()=>{for(let g of ghosts) g.fright=false;},6000); }

// Rendering
function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);
  // background
  ctx.fillStyle='#000'; ctx.fillRect(0,0,canvas.width,canvas.height);
  // draw map
  for(let r=0;r<ROWS;r++){
    for(let c=0;c<COLS;c++){
      const x=c*TILE, y=r*TILE;
      if(map[r][c]===1){ ctx.fillStyle='#005500'; ctx.fillRect(x+2,y+2,TILE-4,TILE-4); }
      if(map[r][c]===2){ ctx.beginPath(); ctx.fillStyle='#ffd700'; ctx.arc(x+TILE/2,y+TILE/2,2,0,Math.PI*2); ctx.fill(); }
      if(map[r][c]===3){ ctx.beginPath(); ctx.fillStyle='#fff'; ctx.arc(x+TILE/2,y+TILE/2,6,0,Math.PI*2); ctx.fill(); }
    }
  }
  // draw pac-man
  let pxy=tileToXY(pac.r,pac.c);
  ctx.save();
  ctx.translate(pxy.x,pxy.y);
  // mouth animation
  pac.mouth = (pac.mouth + 0.25) % 2;
  const mouthAng = 0.25 + Math.abs(Math.sin(pac.mouth))*0.35;
  let angle = 0;
  if(pac.dir.r===-1) angle = -Math.PI/2; else if(pac.dir.r===1) angle = Math.PI/2; else if(pac.dir.c===-1) angle = Math.PI; else angle=0;
  ctx.rotate(angle);
  ctx.beginPath(); ctx.fillStyle='#ffeb3b'; ctx.moveTo(0,0);
  ctx.arc(0,0,8, mouthAng, Math.PI*2-mouthAng);
  ctx.closePath(); ctx.fill(); ctx.restore();

  // draw ghosts
  for(let g of ghosts){
    let gxy=tileToXY(g.r,g.c);
    ctx.beginPath(); ctx.fillStyle = g.fright ? '#88a' : g.color; ctx.rect(gxy.x-8,gxy.y-8,16,16); ctx.fill();
  }
}

function loop(t){
  if(!lastTime) lastTime=t;
  const dt=(t-lastTime)/1000; lastTime=t;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}

requestAnimationFrame(loop);

// Accessibility - basic touch buttons (small) — optional overlay for mobile
(function addTouchControls(){
  const isTouch = 'ontouchstart' in window || navigator.maxTouchPoints>0;
  if(!isTouch) return;
  const wrap = document.createElement('div');
  wrap.style.position='fixed'; wrap.style.right='12px'; wrap.style.bottom='12px'; wrap.style.zIndex=9999; wrap.style.display='grid'; wrap.style.gridTemplateColumns='repeat(3,48px)'; wrap.style.gap='6px';
  const btn = (label,dir)=>{ const b=document.createElement('button'); b.innerText=label; b.style.width='48px'; b.style.height='48px'; b.style.borderRadius='8px'; b.style.border='none'; b.style.background='#222'; b.style.color='#fff'; b.style.fontSize='16px'; b.addEventListener('touchstart',e=>{ e.preventDefault(); pac.nextDir=dir; }); return b; };
  wrap.append(btn('',{})); wrap.append(btn('↑',{r:-1,c:0})); wrap.append(btn('',{})); wrap.append(btn('←',{r:0,c:-1})); wrap.append(btn('↓',{r:1,c:0})); wrap.append(btn('→',{r:0,c:1}));
  document.body.appendChild(wrap);
})();

// Simple instructions in console for contributors
console.log('Pac-Man game ready. Open pacman.html (this file) in browser or enable GitHub Pages to host it.');
</script>
</body>
</html>

</p>

---

<svg xmlns="http://www.w3.org/2000/svg" width="100%" height="150" viewBox="0 0 800 150">
  <rect width="100%" height="100%" fill="black"/>
  <text x="50%" y="50%" text-anchor="middle" fill="none" stroke="#25F4EE" stroke-width="2" font-size="40" font-family="Arial Black, sans-serif" dy=".3em">
    <tspan>Thanks for Watching</tspan>
    <animate attributeName="stroke-dasharray" from="0,500" to="500,0" dur="5s" repeatCount="indefinite"/>
    <animate attributeName="stroke" values="#25F4EE;#FE2C55;#25F4EE" dur="4s" repeatCount="indefinite"/>
  </text>
</svg>

