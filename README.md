<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Auto Roulette • PREVIEW</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0" />

<style>
body{
  margin:0;
  background:#0b0f14;
  color:#fff;
  font-family:Arial, sans-serif;
  text-align:center;
}
header{
  padding:12px;
  background:#121821;
  display:flex;
  justify-content:space-between;
  align-items:center;
}
button{
  padding:12px 18px;
  border:none;
  border-radius:10px;
  font-weight:bold;
  cursor:pointer;
}
.red{background:#c62828;color:#fff}
.black{background:#000;color:#fff}
.gray{background:#374151;color:#fff}
.green{background:#2e7d32;color:#fff}
.container{padding:10px}
canvas{
  background:#0b0f14;
  border-radius:50%;
}
.spinning{filter:blur(3px)}
.card{
  background:#1a2230;
  margin:10px auto;
  padding:12px;
  border-radius:12px;
  max-width:420px;
}
.admin{display:none}
.admin.active{display:block}
#dealer{
  margin-top:10px;
  padding:10px;
  background:#000;
  border-radius:10px;
}
#dealer span{color:red;font-weight:bold}
@media(max-width:768px){
  canvas{width:100%!important;height:auto!important}
}
</style>
</head>

<body>

<header>
  <b>🎰 AUTO ROULETTE <small>PREVIEW</small></b>
  <button class="gray" onclick="toggleAdmin()">ADMIN</button>
</header>

<div class="container">

<div class="card">
  Balance: <b id="balance">1000</b><br/>
  Bet: <input id="bet" type="number" value="10" style="width:80px">
</div>

<div class="card">
  <button class="red" onclick="setColor('red')">RED</button>
  <button class="black" onclick="setColor('black')">BLACK</button>
  <button class="green" onclick="spin()">▶ SPIN</button>
</div>

<div class="card">
  <canvas id="wheel" width="300" height="300"></canvas>
  <div id="result">—</div>
</div>

<div id="dealer" class="card">
  <span>● LIVE</span><br/>
  <div id="dealerText">Waiting for bets…</div>
</div>

<div class="card">
  <b>RTP</b>
  <canvas id="rtpChart" width="300" height="120"></canvas>
</div>

<div class="card">
  History:
  <div id="history"></div>
</div>

<!-- ADMIN -->
<div class="card admin" id="admin">
  <h3>ADMIN PREVIEW</h3>
  Target RTP:
  <input id="targetRtp" type="number" value="96" style="width:60px"> %
  <br/><br/>
  Fake players online: <b>12</b><br/>
  Jurisdiction: MGA (MOCK)<br/>
</div>

</div>

<!-- SOUNDS -->
<audio id="sndSpin" src="https://assets.mixkit.co/sfx/preview/mixkit-slot-machine-spin-1930.mp3"></audio>
<audio id="sndWin" src="https://assets.mixkit.co/sfx/preview/mixkit-winning-notification-2018.mp3"></audio>
<audio id="sndLose" src="https://assets.mixkit.co/sfx/preview/mixkit-arcade-retro-game-over-213.wav"></audio>

<script>
/* GAME STATE */
let balance=1000, betColor="red", spinning=false;
let stats={bet:0,win:0,rtp:[]};
let angle=0;

/* CANVAS */
const ctx=document.getElementById("wheel").getContext("2d");
const rtpCtx=document.getElementById("rtpChart").getContext("2d");

function drawWheel(a){
  const r=150,slice=2*Math.PI/37;
  ctx.clearRect(0,0,300,300);
  for(let i=0;i<37;i++){
    ctx.beginPath();
    ctx.moveTo(r,r);
    ctx.arc(r,r,r,i*slice+a,(i+1)*slice+a);
    ctx.fillStyle=i===0?"green":i%2?"red":"black";
    ctx.fill();
  }
  ctx.fillStyle="white";
  ctx.fillRect(r-5,0,10,20);
}

drawWheel(0);

/* GAME */
function setColor(c){betColor=c;}

function spin(){
  if(spinning||balance<+bet.value)return;
  spinning=true;
  balance-=+bet.value;
  stats.bet+=+bet.value;
  updateUI();

  document.getElementById("sndSpin").play();
  dealer("No more bets…");

  const number=Math.floor(Math.random()*37);
  const target=10*Math.PI+(36-number)*(2*Math.PI/37);
  const start=angle;
  let t0=null;

  document.getElementById("wheel").classList.add("spinning");

  function easeOut(t){return 1-Math.pow(1-t,3);}

  function anim(t){
    if(!t0)t0=t;
    const p=Math.min((t-t0)/4000,1);
    angle=start+(target-start)*easeOut(p);
    drawWheel(angle);
    if(p<1)requestAnimationFrame(anim);
    else finish(number);
  }
  requestAnimationFrame(anim);
}

function finish(n){
  spinning=false;
  document.getElementById("wheel").classList.remove("spinning");

  const color=n===0?"green":n%2?"red":"black";
  if(color===betColor){
    balance+=+bet.value*2;
    stats.win+=+bet.value*2;
    document.getElementById("sndWin").play();
    result.innerText="WIN "+n+" ("+color+")";
    dealer("Winning number "+n);
  }else{
    document.getElementById("sndLose").play();
    result.innerText="LOSE "+n+" ("+color+")";
    dealer("Result "+n);
  }

  stats.rtp.push(stats.win/stats.bet*100||0);
  history.innerHTML="<span>"+n+"</span> "+history.innerHTML;
  drawRtp();
  updateUI();
}

function updateUI(){
  balanceEl.innerText=balance;
}

/* RTP GRAPH */
function drawRtp(){
  rtpCtx.clearRect(0,0,300,120);
  rtpCtx.beginPath();
  stats.rtp.forEach((v,i)=>{
    const x=i*(300/stats.rtp.length);
    const y=120-v;
    i?rtpCtx.lineTo(x,y):rtpCtx.moveTo(x,y);
  });
  rtpCtx.strokeStyle="#4cff4c";
  rtpCtx.stroke();
}

/* LIVE DEALER TEXT */
function dealer(t){
  dealerText.innerText=t;
}

/* ADMIN */
function toggleAdmin(){
  admin.classList.toggle("active");
}

const balanceEl=document.getElementById("balance");
</script>

</body>
</html>
