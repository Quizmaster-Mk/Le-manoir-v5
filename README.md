<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>La Malédiction du Manoir — Version évoluée</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Creepster&display=swap');

  :root{
    --accent:#ff3c00;
    --bg-dark:#060607;
    --panel:rgba(0,0,0,0.78);
    --muted:rgba(255,255,255,0.65);
    --good: #00ff99;
    --bad: #ff4d4d;
  }

  html,body{height:100%;margin:0;background:radial-gradient(circle at 20% 10%, #050506 10%, #0b0b0e 60%);color:#eee;font-family:"Creepster", cursive}
  #bg{position:fixed; inset:0; z-index:-3; background:url('https://i.ibb.co/ZTDCxmr/haunted-mansion-night.jpg') center/cover no-repeat; filter:brightness(0.28) contrast(1.05)}
  #fog{position:fixed; inset:0; z-index:-2; pointer-events:none; mix-blend-mode:screen; opacity:0.9}
  #flash{position:fixed; inset:0; background:#fff; z-index:9999; opacity:0; pointer-events:none; transition:opacity .08s linear}

  header{display:flex; justify-content:space-between; align-items:center; padding:12px 16px}
  .title{font-size:clamp(20px,4.5vw,36px); color:var(--accent); text-shadow:0 0 10px rgba(255,60,0,0.6)}
  .controls{display:flex; gap:8px; align-items:center}
  .toggle{background:transparent; border:2px solid rgba(255,255,255,0.06); color:#fff; padding:8px 12px; border-radius:10px; cursor:pointer; font-family:inherit}
  .toggle.active{background:var(--accent); color:#000; border-color:transparent}

  .hud{max-width:980px; margin:12px auto; display:flex; gap:12px; align-items:center; justify-content:center; flex-wrap:wrap}
  .bar{background:#141414; border-radius:10px; padding:6px 8px; border:2px solid rgba(255,255,255,0.04); min-width:160px}
  .bar .label{font-size:12px; color:var(--muted); text-align:left}
  .meter{height:14px; background:#111; border-radius:8px; overflow:hidden; margin-top:6px; border:1px solid rgba(255,255,255,0.02)}
  .fill{height:100%; width:100%; transition:width .35s, background .35s}
  #sanityFill{background:linear-gradient(90deg,#00ff99,#0099ff)}
  .small{font-size:12px; color:var(--muted); margin-top:6px; text-align:center}

  main{max-width:900px; margin:10px auto 48px; padding:18px}
  #story{background:var(--panel); border:3px solid rgba(255,60,0,0.16); border-radius:12px; padding:20px; font-size:1.05rem; line-height:1.6; box-shadow:0 12px 40px rgba(0,0,0,.6); transition:transform .2s}
  .choices{display:flex; flex-direction:column; gap:10px; margin-top:14px}
  button.choice{background:#121214; color:var(--accent); border:2px solid var(--accent); padding:12px; border-radius:10px; cursor:pointer; font-size:1rem}
  button.choice:hover{background:var(--accent); color:#111; transform:scale(1.03)}
  .muted{color:var(--muted); font-size:13px}

  /* visual effects */
  .shake{animation:shake .45s}
  @keyframes shake{0%{transform:translateX(0)}25%{transform:translateX(-12px)}50%{transform:translateX(10px)}75%{transform:translateX(-8px)}100%{transform:translateX(0)}}
  .glitch { filter: url(#snoise); } /* placeholder for SVG filter if desired */
  .low-sanity { filter: hue-rotate(-20deg) saturate(.6) blur(0.4px) }
  .text-warp { font-family: 'Creepster', cursive; transform-origin:center; display:inline-block }

  /* mini-game box */
  #ritualBox{display:none; margin-top:14px; background:rgba(255,255,255,0.02); padding:12px; border-radius:8px}
  #ritualInput{font-size:1.1rem; padding:10px; width:60%; max-width:260px; border-radius:8px; border:none; text-align:center}

  footer{text-align:center; color:rgba(255,255,255,0.45); margin-top:12px; font-size:13px}

  /* responsive */
  @media(max-width:640px){ main{padding:12px} button.choice{font-size:.95rem; padding:10px} .bar{min-width:140px} }
</style>
</head>
<body class="custom-cursor">

<div id="bg"></div>
<canvas id="fog"></canvas>
<div id="flash"></div>

<header>
  <div class="title">🏚️ La Malédiction du Manoir — Évolué</div>
  <div class="controls">
    <button id="soundBtn" class="toggle active">🔊 Son</button>
    <button id="fastBtn" class="toggle">⚡ Fast</button>
    <button id="inspectBtn" class="toggle">🔎 Inspect</button>
  </div>
</header>

<div class="hud">
  <div class="bar" title="Santé mentale">
    <div class="label">Santé mentale</div>
    <div class="meter"><div id="sanityFill" class="fill" style="width:100%"></div></div>
    <div class="small" id="sanityText">100 / 100</div>
  </div>

  <div class="bar" title="Curiosité (influence certains choix)">
    <div class="label">Curiosité</div>
    <div class="meter"><div id="curiosityFill" class="fill" style="width:0%; background:linear-gradient(90deg,#66ccff,#00ddff)"></div></div>
    <div class="small" id="curiosityText">0</div>
  </div>

  <div class="bar" title="Corruption (plus tu l'alimentes, pires sont les fins)">
    <div class="label">Corruption</div>
    <div class="meter"><div id="corruptionFill" class="fill" style="width:0%; background:linear-gradient(90deg,#ffcc00,#ff4d4d)"></div></div>
    <div class="small" id="corruptionText">0</div>
  </div>
</div>

<main>
  <div id="story" role="main" aria-live="polite"></div>
</main>

<footer>Conseil : certains choix modifient Curiosité / Corruption — montre ce que tu deviens au fil des runs.</footer>

<!-- sons -->
<audio id="amb" src="https://cdn.pixabay.com/download/audio/2022/10/28/audio_5e6f6e7f2c.mp3?filename=horror-ambient-113870.mp3" loop></audio>
<audio id="thunder" src="https://actions.google.com/sounds/v1/weather/thunder_strike.ogg"></audio>
<audio id="shock" src="https://actions.google.com/sounds/v1/foley/metal_drop.ogg"></audio>
<audio id="typeok" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
<audio id="typebad" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>

<script>
/* ============================
   État du joueur (variables cachées visibles via Inspect)
   ============================ */
let sanity = 100;
let curiosity = 0;
let corruption = 0;
let fastMode = false;
let soundOn = true;
let inspect = false;

/* DOM refs */
const storyEl = document.getElementById('story');
const sanityFill = document.getElementById('sanityFill');
const sanityText = document.getElementById('sanityText');
const curiosityFill = document.getElementById('curiosityFill');
const curiosityText = document.getElementById('curiosityText');
const corruptionFill = document.getElementById('corruptionFill');
const corruptionText = document.getElementById('corruptionText');

const amb = document.getElementById('amb');
const thunder = document.getElementById('thunder');
const shock = document.getElementById('shock');
const typeok = document.getElementById('typeok');
const typebad = document.getElementById('typebad');
const flash = document.getElementById('flash');
const fogCanvas = document.getElementById('fog');

const soundBtn = document.getElementById('soundBtn');
const fastBtn = document.getElementById('fastBtn');
const inspectBtn = document.getElementById('inspectBtn');

/* Scenes with effects + variable deltas
   fields:
    - sanity: delta applied on enter
    - curiosity: delta
    - corruption: delta
    - flash / shake / fog: visual flags
    - ritual: true -> show mini-game
    - choices: array of {text,next}
    - end: text -> terminal scene
*/
const scenes = {
  start: {
    text:"La pluie fouette tes épaules. Devant toi, un manoir aux volets clos. Une porte s'entrouvre... Entrer ou rebrousser chemin ?",
    sanity:0, choices:[
      {text:"Entrer", next:"hall", curiosity:2},
      {text:"Fuir", next:"outside"}
    ]
  },
  outside: { text:"Tu fuis. La pluie t'engloutit. Tu as échappé au manoir, pour l'instant.", sanity:+5, end:"Parfois la peur sauve." },
  hall: {
    text:"Le hall est vaste. Des portraits aux regards froids. Une bougie s'éteint seule. Où aller ?",
    sanity:-6, flash:true, choices:[
      {text:"Suivre une ombre", next:"shadow", curiosity:3},
      {text:"Aller à la bibliothèque", next:"library", curiosity:2},
      {text:"Chercher une trappe", next:"trapdoor", curiosity:2}
    ]
  },
  shadow: {
    text:"L'ombre glisse vers un miroir ancien dont le verre semble vivant. Ton reflet te salue étrangement.",
    sanity:-12, curiosity:+1, corruption:+3, flash:true, shake:true,
    choices:[
      {text:"Briser le miroir", next:"shatter", corruption:+10},
      {text:"Regarder de plus près", next:"mirrorTrap", curiosity:+2}
    ]
  },
  shatter: {
    text:"Le miroir explose. Un courant sombre te traverse... Quelque chose a été réveillé.",
    sanity:-20, corruption:+20, end:"Une force t'a consommé. Le manoir gagne un nouveau gardien."
  },
  mirrorTrap: {
    text:"Ton reflet t'attire. Tu sens le monde se tordre...",
    sanity:-25, corruption:+15, end:"Ton reflet t'emprisonne. Tu es perdu."
  },
  library: {
    text:"La bibliothèque sent la poussière et le parchemin. Un grimoire posé sur un pupitre est ouvert à une page marquée.",
    sanity:-5, curiosity:+4, choices:[
      {text:"Lire la page", next:"ritualPage", curiosity:+3},
      {text:"Refermer et partir", next:"hall", sanity:+2}
    ]
  },
  trapdoor: {
    text:"Sous le tapis, une trappe. Un souffle glacé s'en échappe.",
    sanity:-7, curiosity:+2, choices:[
      {text:"Descendre", next:"cellar", curiosity:+2},
      {text:"Revenir en arrière", next:"hall", sanity:+1}
    ]
  },
  cellar: {
    text:"Le sous-sol est humide; des symboles gravés sur une dalle semblent récents.",
    sanity:-10, curiosity:+2, corruption:+2, choices:[
      {text:"S'approcher de la dalle", next:"crypt"},
      {text:"Remonter", next:"hall", sanity:+1}
    ]
  },
  crypt: {
    text:"Une crypte secrète. Un cercle rituel est tracé. Ici, le grimoire parlait d'un mot-clef à prononcer... mais il y a aussi une option d'invoquer le pouvoir pour toi.",
    sanity:-12, choices:[
      {text:"Faire le rituel (taper LUMEN)", next:"ritual", ritual:true},
      {text:"Ignorer et fuir", next:"escape", sanity:+4}
    ]
  },
  ritual: {
    text:"Tu prépares le rituel. Tape le mot mystique *LUMEN* dans la boîte en moins de temps pour tenter de *libérer* les âmes (réussite) ou sinon risquer d'augmenter la corruption (échec).",
    sanity:-8, ritual:true
  },
  /* Ritual success/fail handled in mini-game logic */
  ritualSuccess: {
    text:"Les cierges s'élèvent, une brise de lumière purifie la crypte.",
    sanity:+18, curiosity:+5, corruption:-12, end:"Les esprits sont libérés. Le manoir exhale un dernier soupir."
  },
  ritualFail: {
    text:"Le mot est prononcé à moitié ; une ombre t'enveloppe.",
    sanity:-25, corruption:+20, end:"La tentative a raté. Le manoir se gorge d'une nouvelle malédiction."
  },
  collapse: { text:"Le rituel échoue, le sol se fissure et tout s'effondre.", sanity:-20, end:"Le manoir s'effondre. Ton cri se perd dans la nuit."},
  escape: { text:"Tu trouves la sortie et échappes au manoir...", sanity:+6, end:"Tu es libre mais hanté."},
  insanity: { text:"Ta raison t'abandonne. Les murs respirent. Tu entends des voix.", sanity:-100, end:"La folie t'a pris."}
};

/* ============================
   Utilitaires & UI updates
   ============================ */

function clamp(v,min,max){ return Math.max(min,Math.min(max,v)); }

function updateHUD(){
  sanity = clamp(sanity,0,100);
  curiosity = clamp(curiosity,0,100);
  corruption = clamp(corruption,0,100);
  sanityFill.style.width = sanity + "%";
  document.getElementById('sanityText').innerText = `${sanity} / 100`;
  curiosityFill.style.width = curiosity + "%";
  curiosityText.innerText = curiosity;
  corruptionFill.style.width = corruption + "%";
  corruptionText.innerText = corruption;

  // sanity color
  if(sanity>60) sanityFill.style.background = "linear-gradient(90deg,#00ff99,#0099ff)";
  else if(sanity>30) sanityFill.style.background = "linear-gradient(90deg,#ffcc00,#ff6600)";
  else sanityFill.style.background = "linear-gradient(90deg,#ff0000,#660000)";

  // visual effects based on sanity
  if(sanity <= 25) storyEl.classList.add('low-sanity'); else storyEl.classList.remove('low-sanity');
  if(sanity <= 15) { /* heavy insanity hints could be added */ }
  // music volume adapt
  if(soundOn){
    const vol = clamp(0.08 + (100 - sanity)/220, 0.04, 0.45);
    amb.volume = vol;
    if(amb.paused) amb.play().catch(()=>{/* ignore */});
  } else { amb.pause(); }
}

/* flash and shake helpers */
function triggerFlash(){
  flash.style.opacity = '1';
  thunder.currentTime = 0;
  if(soundOn) thunder.play();
  setTimeout(()=> flash.style.opacity = '0', 120);
}
function triggerShake(){
  storyEl.classList.add('shake');
  shock.currentTime = 0;
  if(soundOn) shock.play();
  setTimeout(()=> storyEl.classList.remove('shake'), 460);
}

/* fog canvas */
const fctx = fogCanvas.getContext('2d');
let FWIDTH, FHEIGHT, fogParts=[];
function resizeFog(){ fogCanvas.width = window.innerWidth; fogCanvas.height = window.innerHeight; FWIDTH=fogCanvas.width; FHEIGHT=fogCanvas.height; }
window.addEventListener('resize', resizeFog);
function spawnFog(intensity=0.2){
  fogParts = [];
  const count = Math.round(25 + intensity*100);
  for(let i=0;i<count;i++) fogParts.push({ x:Math.random()*FWIDTH, y:Math.random()*FHEIGHT, r:60+Math.random()*120, vx:(Math.random()-0.5)*0.3, vy:(Math.random()-0.5)*0.3, a:0.06+Math.random()*0.25, hue:200-Math.random()*40 });
}
function drawFog(){
  fctx.clearRect(0,0,FWIDTH,FHEIGHT);
  fogParts.forEach(p=>{
    const g = fctx.createRadialGradient(p.x,p.y,10,p.x,p.y,p.r);
    g.addColorStop(0, `hsla(${p.hue},80%,90%,${p.a*0.08})`);
    g.addColorStop(1, `hsla(${p.hue},80%,40%,0)`);
    fctx.fillStyle=g; fctx.fillRect(p.x-p.r,p.y-p.r,p.r*2,p.r*2);
    p.x += p.vx; p.y += p.vy;
    if(p.x<-p.r) p.x=FWIDTH+p.r; if(p.x>FWIDTH+p.r) p.x=-p.r;
    if(p.y<-p.r) p.y=FHEIGHT+p.r; if(p.y>FHEIGHT+p.r) p.y=-p.r;
  });
  requestAnimationFrame(drawFog);
}
resizeFog(); spawnFog(0.25); drawFog();

/* ============================
   Scene display & logic
   ============================ */

let currentScene = 'start';
function applyDeltas(params){
  if(!params) return;
  if(params.sanity) sanity += params.sanity;
  if(params.curiosity) curiosity += params.curiosity;
  if(params.corruption) corruption += params.corruption;
  updateHUD();
}

function showScene(name){
  const scene = scenes[name];
  if(!scene) return;
  currentScene = name;

  // apply deltas (some scenes call with numeric deltas attached to choices)
  // note: choice-linked deltas handled when clicked; here we apply scene base deltas
  if(typeof scene.sanity !== 'undefined') sanity += scene.sanity;
  if(typeof scene.curiosity !== 'undefined') curiosity += scene.curiosity;
  if(typeof scene.corruption !== 'undefined') corruption += scene.corruption;

  // visual effects
  if(scene.flash) triggerFlash();
  if(scene.shake) triggerShake();
  spawnFog(scene.fog || 0.25);
  updateHUD();

  // build scene html
  let html = `<p class="text-warp">${scene.text}</p>`;

  if(scene.choices){
    html += `<div class="choices">`;
    scene.choices.forEach(choice=>{
      // choices can carry deltas too
      const dataAttrs = [
        choice.curiosity ? `data-curiosity="${choice.curiosity}"` : '',
        choice.corruption ? `data-corruption="${choice.corruption}"` : '',
        choice.sanity ? `data-sanity="${choice.sanity}"` : ''
      ].join(' ');
      html += `<button class="choice" ${dataAttrs} data-next="${choice.next}" onclick="choiceClick(event)">${choice.text}</button>`;
    });
    html += `</div>`;
    storyEl.innerHTML = html;

    // fast mode auto-advance to first choice
    if(fastMode){
      setTimeout(()=> {
        const first = storyEl.querySelector('.choice');
        if(first) first.click();
      }, 600);
    }

  } else if(scene.end){
    html += `<div id="end" style="margin-top:16px">${scene.end}</div>`;
    html += `<div class="choices" style="margin-top:12px"><button class="choice" onclick="restart()">🔁 Rejouer</button></div>`;
    storyEl.innerHTML = html;
  } else if(scene.ritual){
    // Shouldn't reach here; ritual scenes handled separately
    storyEl.innerHTML = html;
  } else {
    storyEl.innerHTML = html;
  }

  // sanity-based subtle effects
  if(sanity <= 20) { storyEl.classList.add('low-sanity'); }
  else storyEl.classList.remove('low-sanity');

  // if sanity <=0 immediate insanity
  if(sanity <= 0){ showEnd('insanity'); }
}

/* choice clicked - read data attrs and update variables and next */
function choiceClick(e){
  const btn = e.currentTarget;
  const next = btn.getAttribute('data-next');
  const s = parseInt(btn.getAttribute('data-sanity')||0,10);
  const c = parseInt(btn.getAttribute('data-curiosity')||0,10);
  const corr = parseInt(btn.getAttribute('data-corruption')||0,10);
  if(s) sanity += s;
  if(c) curiosity += c;
  if(corr) corruption += corr;
  // if choice leads to ritual scene that needs mini-game, we handle below when showScene detects ritual or name 'ritual'
  // special: if next === 'ritual' we start mini-game
  if(next === 'ritual'){
    // show the ritual mini-game UI directly
    startRitualMiniGame();
    return;
  }
  // otherwise show next normally (some scenes like 'ritual' defined separately)
  showScene(next);
}

/* show end helper */
function showEnd(name){
  const scene = scenes[name] || scenes['insanity'];
  let html = `<p>${scene.text}</p><div id="end">${scene.end || '...'}</div>`;
  html += `<div class="choices" style="margin-top:12px"><button class="choice" onclick="restart()">🔁 Rejouer</button></div>`;
  storyEl.innerHTML = html;
}

/* restart */
function restart(){
  sanity=100; curiosity=0; corruption=0; updateHUD();
  spawnFog(0.25); showScene('start');
}

/* ============================
   Ritual mini-game (typing)
   ============================ */

let ritualTimeout = null;
let ritualActive = false;
function startRitualMiniGame(){
  // render a special UI inside storyEl
  ritualActive = true;
  // small delay: show instructions then start timer
  const timeLimit = 7000; // ms to type LUMEN
  let html = `<p class="text-warp">${scenes['ritual'].text}</p>`;
  html += `<div id="ritualBox"><div class="muted">Tape le mot <b>LUMEN</b> dans <span id="ritualTimer">${Math.ceil(timeLimit/1000)}</span>s pour tenter de libérer (succès) ; sinon risque de corruption (échec).</div>`;
  html += `<div style="margin-top:10px"><input id="ritualInput" autocomplete="off" autocapitalize="off" placeholder="Tape ici..." maxlength="20"></div>`;
  html += `<div style="margin-top:10px" class="muted">Astuce : majuscules/minuscules non sensibles.</div></div>`;
  storyEl.innerHTML = html;

  const input = document.getElementById('ritualInput');
  const timerSpan = document.getElementById('ritualTimer');
  input.focus();

  // start countdown
  let remaining = Math.ceil(timeLimit/1000);
  timerSpan.innerText = remaining;
  ritualTimeout = setInterval(()=>{
    remaining--;
    if(remaining <= 0){
      clearInterval(ritualTimeout);
      ritualFail();
    } else timerSpan.innerText = remaining;
  }, 1000);

  // also final time-based fail if not typed by limit
  const deadline = Date.now() + timeLimit;
  const checkInterval = setInterval(()=>{
    if(!ritualActive){ clearInterval(checkInterval); return; }
    if(Date.now() >= deadline){
      clearInterval(checkInterval);
      if(ritualActive) ritualFail();
    }
  }, 200);

  // listen for input
  const onInput = (ev) => {
    const v = input.value.trim().toLowerCase();
    if(v === 'lumen'){
      // success
      ritualActive = false;
      clearInterval(ritualTimeout);
      input.removeEventListener('input', onInput);
      typeok.currentTime = 0; if(soundOn) typeok.play();
      ritualSuccess();
    }
  };
  input.addEventListener('input', onInput);

  // also provide a cancel if user navigates away
  // Minor: pressing Enter triggers check
  input.addEventListener('keydown', (e)=>{
    if(e.key === 'Enter') {
      if(input.value.trim().toLowerCase() === 'lumen'){
        ritualActive = false;
        clearInterval(ritualTimeout);
        typeok.currentTime = 0; if(soundOn) typeok.play();
        ritualSuccess();
      } else {
        // small penalty
        typebad.currentTime = 0; if(soundOn) typebad.play();
        // continue
      }
    }
  });

  // small flash + fog intensify
  triggerFlash();
  spawnFog(0.9);
}

/* ritual outcomes */
function ritualSuccess(){
  // apply deltas
  sanity += 18; curiosity += 6; corruption = Math.max(0, corruption - 12);
  updateHUD();
  showScene('ritualSuccess');
  ritualActive = false;
  spawnFog(0.12);
}

function ritualFail(){
  ritualActive = false;
  // penalty
  sanity -= 22; corruption += 20;
  updateHUD();
  typebad.currentTime = 0; if(soundOn) typebad.play();
  showScene('ritualFail');
  spawnFog(0.8);
}

/* ============================
   Controls binding
   ============================ */
soundBtn.addEventListener('click', ()=>{
  soundOn = !soundOn;
  soundBtn.classList.toggle('active', soundOn);
  soundBtn.innerText = soundOn ? '🔊 Son' : '🔇 Muet';
  updateHUD();
});
fastBtn.addEventListener('click', ()=>{
  fastMode = !fastMode;
  fastBtn.classList.toggle('active', fastMode);
  fastBtn.innerText = fastMode ? '⚡ Fast (ON)' : '⚡ Fast';
});
inspectBtn.addEventListener('click', ()=>{
  inspect = !inspect;
  inspectBtn.classList.toggle('active', inspect);
  inspectBtn.innerText = inspect ? '🔎 Inspect (ON)' : '🔎 Inspect';
  // toggling inspect reveals values briefly
  if(inspect){
    alert(`Variables (debug):\nCuriosité: ${curiosity}\nCorruption: ${corruption}\nSanity: ${sanity}`);
  }
});

/* keyboard quick action: Enter = first choice */
window.addEventListener('keydown', (e)=>{
  if(e.key === 'Enter' && !ritualActive){
    const first = storyEl.querySelector('.choice');
    if(first) first.click();
  }
});

/* clicking background triggers thunder for fun */
document.getElementById('bg').addEventListener('click', ()=> { triggerFlash(); });

/* init */
updateHUD();
spawnFog(0.25);
showScene('start');
if(soundOn){ amb.volume = 0.12; amb.play().catch(()=>{/* may be blocked */}); }

</script>
</body>
</html>
