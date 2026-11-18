<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Fahr zu Emna</title>
<style>
:root{--bg:#2f5d3a;--ink:#fff;--panel:#1d3a25;--road:#4a4a4f;color-scheme:dark}
@media (prefers-color-scheme: light){:root{--bg:#3f7a4c;--panel:#234a2e}}
*{box-sizing:border-box}
html,body{height:100%;margin:0}
body{background:var(--bg);color:var(--ink);font-family:"Trebuchet MS",Verdana,sans-serif;display:flex;flex-direction:column;align-items:center;padding:env(safe-area-inset-top,0px) 10px env(safe-area-inset-bottom,0px);overflow:hidden}
h1{font-size:1.3rem;margin:10px 0 4px}
#hud{display:flex;gap:16px;font-size:1.1rem;margin-bottom:6px}
#wrap{position:relative;flex:1;min-height:0;width:100%;max-width:420px;display:flex;justify-content:center}
canvas{height:100%;max-width:100%;background:var(--bg);border-radius:8px;touch-action:none}
#menu{position:absolute;inset:0;background:rgba(20,40,25,.92);border-radius:8px;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:12px;padding:16px;text-align:center}
#menu.hide{display:none}
.cars{display:flex;gap:10px}
.car{background:var(--panel);border:3px solid transparent;border-radius:10px;padding:10px 8px;color:var(--ink);font:inherit;cursor:pointer;min-width:84px}
.car:focus-visible{outline:3px solid #fff}
.car.sel{border-color:#ffd84a}
.car i{display:block;width:30px;height:52px;border-radius:8px;margin:0 auto 6px;border:2px solid #0006}
#go{background:#ffd84a;color:#222;border:0;border-radius:10px;padding:12px 28px;font:700 1.1rem inherit;font-family:inherit;cursor:pointer}
#ctl{display:flex;gap:12px;margin:8px 0}
#ctl button{width:90px;height:56px;font-size:1.6rem;border:0;border-radius:12px;background:var(--panel);color:var(--ink)}
small{opacity:.8}
</style>
</head>
<body>
<h1>Fahr zu Emna</h1>
<div id="hud"><span>Punkte: <b id="score">0</b></span><span>Fahrzeug: <b id="who">Sonia</b></span></div>
<div id="wrap">
  <canvas id="c" width="360" height="560"></canvas>
  <div id="menu">
    <h2 style="margin:0">Wähle dein Fahrzeug</h2>
    <div class="cars" id="cars"></div>
    <small>Fahre gegen den Baum Emna und sammle Punkte.<br>Steuern: ← → Tasten oder Buttons</small>
    <button id="go">Los geht’s</button>
  </div>
</div>
<div id="ctl"><button id="L" aria-label="Links">◀</button><button id="R" aria-label="Rechts">▶</button><button id="M" aria-label="Menü" style="font-size:1rem">Menü</button></div>
<script>
const CARS=[{name:"Sonia",color:"#e63946"},{name:"Anes",color:"#2a6fdb"},{name:"Fares",color:"#2fb344"}];
const cv=document.getElementById("c"),ctx=cv.getContext("2d");
const W=cv.width,H=cv.height,roadL=60,roadR=W-60;
let car=CARS[0],score=0,running=false,x,speed,offset=0,tree,keys={L:false,R:false};

const cars=document.getElementById("cars");
CARS.forEach((c,i)=>{
  const b=document.createElement("button");
  b.className="car"+(i===0?" sel":"");
  b.innerHTML='<i style="background:'+c.color+'"></i>'+c.name;
  b.onclick=()=>{car=c;[...cars.children].forEach(e=>e.classList.remove("sel"));b.classList.add("sel");document.getElementById("who").textContent=c.name};
  cars.appendChild(b);
});

function newTree(){tree={x:roadL+30+Math.random()*(roadR-roadL-60),y:-80,hit:false}}
function start(){score=0;x=W/2;speed=3;newTree();running=true;document.getElementById("score").textContent=0;document.getElementById("menu").classList.add("hide")}
document.getElementById("go").onclick=start;
document.getElementById("M").onclick=()=>{running=false;document.getElementById("menu").classList.remove("hide")};

addEventListener("keydown",e=>{if(e.key==="ArrowLeft")keys.L=true;if(e.key==="ArrowRight")keys.R=true});
addEventListener("keyup",e=>{if(e.key==="ArrowLeft")keys.L=false;if(e.key==="ArrowRight")keys.R=false});
[["L","L"],["R","R"]].forEach(([id,k])=>{
  const b=document.getElementById(id);
  ["pointerdown"].forEach(ev=>b.addEventListener(ev,e=>{e.preventDefault();keys[k]=true}));
  ["pointerup","pointerleave","pointercancel"].forEach(ev=>b.addEventListener(ev,()=>keys[k]=false));
});

function drawCar(px,py){
  ctx.fillStyle="#0004";ctx.fillRect(px-17,py-28,38,64);
  ctx.fillStyle=car.color;ctx.beginPath();ctx.roundRect(px-18,py-32,36,64,8);ctx.fill();
  ctx.fillStyle="#bfe3ff";ctx.fillRect(px-13,py-18,26,14);
  ctx.fillStyle="#111";[[-22,-22],[18,-22],[-22,12],[18,12]].forEach(([dx,dy])=>ctx.fillRect(px+dx,py+dy,6,16));
}
function drawTree(t){
  ctx.fillStyle="#6b4423";ctx.fillRect(t.x-6,t.y,12,30);
  ctx.fillStyle="#1f7a2e";
  [[0,-14,26],[-12,-2,20],[12,-2,20]].forEach(([dx,dy,r])=>{ctx.beginPath();ctx.arc(t.x+dx,t.y+dy,r,0,7);ctx.fill()});
  ctx.fillStyle="#fff";ctx.font="bold 15px Trebuchet MS";ctx.textAlign="center";
  ctx.fillText("Emna",t.x,t.y-44);
}

function loop(){
  ctx.fillStyle=getComputedStyle(document.body).backgroundColor;ctx.fillRect(0,0,W,H);
  ctx.fillStyle="#4a4a4f";ctx.fillRect(roadL,0,roadR-roadL,H);
  ctx.fillStyle="#fff";offset=(offset+(running?speed:1))%60;
  for(let y=-60;y<H;y+=60)ctx.fillRect(W/2-3,y+offset,6,30);
  ctx.fillStyle="#ffd84a";ctx.fillRect(roadL-4,0,4,H);ctx.fillRect(roadR,0,4,H);

  if(running){
    if(keys.L)x-=5;if(keys.R)x+=5;
    x=Math.max(roadL+22,Math.min(roadR-22,x));
    tree.y+=speed;
    const dx=Math.abs(tree.x-x),dy=Math.abs(tree.y+10-(H-110));
    if(dx<30&&dy<45){
      score++;speed=Math.min(8,3+score*0.25);
      document.getElementById("score").textContent=score;newTree();
    }else if(tree.y>H+60){newTree()}
  }
  if(tree)drawTree(tree);
  drawCar(running?x:W/2,H-110);
  requestAnimationFrame(loop);
}
x=W/2;newTree();loop();
</script>
</body>
</html>
