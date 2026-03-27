<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<title>Math App Pro</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap" rel="stylesheet">

<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:Poppins;}

body{
  background: linear-gradient(135deg,#0f2027,#203a43,#2c5364);
  display:flex;
  justify-content:center;
  align-items:center;
  height:100vh;
}

.app{
  width:360px;
  height:720px;
  border-radius:25px;
  overflow:hidden;
  backdrop-filter: blur(20px);
  background: rgba(255,255,255,0.1);
  box-shadow:0 20px 60px rgba(0,0,0,0.5);
  border:1px solid rgba(255,255,255,0.2);
}

.header{
  text-align:center;
  padding:20px;
  font-size:20px;
  color:white;
  font-weight:600;
  background: linear-gradient(135deg,#00c6ff,#0072ff);
}

.screen{
  display:none;
  padding:20px;
  animation:fade 0.5s;
}

.active{display:block;}

@keyframes fade{
  from{opacity:0;transform:translateY(20px);}
  to{opacity:1;transform:translateY(0);}
}

input{
  width:100%;
  padding:12px;
  margin:8px 0;
  border-radius:12px;
  border:none;
  background: rgba(255,255,255,0.2);
  color:white;
}

input::placeholder{color:#ddd;}

button{
  width:100%;
  padding:12px;
  margin-top:10px;
  border:none;
  border-radius:12px;
  background: linear-gradient(135deg,#00c6ff,#0072ff);
  color:white;
  cursor:pointer;
}

.link{
  text-align:center;
  color:#00c6ff;
  margin-top:10px;
  cursor:pointer;
}

#kq{color:white;margin-top:10px;}
#coord{color:#00c6ff;margin-top:5px;}

canvas{
  margin-top:10px;
  border-radius:12px;
  background:white;
}
</style>
</head>

<body>

<div class="app">

<div class="header">🚀 Math App Pro</div>

<!-- LOGIN -->
<div class="screen active" id="login">
  <h3 style="color:white">Đăng nhập</h3>
  <input id="loginUser" placeholder="Username">
  <input id="loginPass" type="password" placeholder="Password">
  <button onclick="login()">Đăng nhập</button>
  <div class="link" onclick="show('register')">Chưa có tài khoản?</div>
</div>

<!-- REGISTER -->
<div class="screen" id="register">
  <h3 style="color:white">Đăng ký</h3>
  <input id="regUser" placeholder="Username">
  <input id="regPass" type="password" placeholder="Password">
  <button onclick="register()">Tạo tài khoản</button>
  <div class="link" onclick="show('login')">Đã có tài khoản?</div>
</div>

<!-- HOME -->
<div class="screen" id="home">
  <h3 style="color:white">📊 Giải PT bậc 2</h3>

  <input id="a" placeholder="Nhập a">
  <input id="b" placeholder="Nhập b">
  <input id="c" placeholder="Nhập c">

  <button onclick="solve()">Giải & Vẽ</button>
  <button onclick="resetGraph()">🔄 Reset</button>
  <button onclick="logout()">Đăng xuất</button>

  <div id="kq"></div>
  <div id="coord"></div>

  <canvas id="graph" width="300" height="220"></canvas>
</div>

</div>

<script>

// UI
function show(id){
  document.querySelectorAll(".screen").forEach(s=>s.classList.remove("active"));
  document.getElementById(id).classList.add("active");
}

// AUTH
function register(){
  localStorage.setItem(regUser.value,regPass.value);
  alert("Đăng ký thành công!");
  show('login');
}

function login(){
  if(localStorage.getItem(loginUser.value)===loginPass.value){
    show('home');
  }else alert("Sai tài khoản!");
}

function logout(){ show('login'); }

// BIẾN
let aInput=document.getElementById("a");
let bInput=document.getElementById("b");
let cInput=document.getElementById("c");
let kq=document.getElementById("kq");
let graph=document.getElementById("graph");

let scale=20, offsetX=0, offsetY=0;
let isDragging=false, startX, startY;

// SOLVE
function solve(){
  let a=+aInput.value,b=+bInput.value,c=+cInput.value;

  if(isNaN(a)||isNaN(b)||isNaN(c)){
    kq.innerHTML="⚠️ Nhập số hợp lệ!";
    return;
  }

  let d=b*b-4*a*c;

  if(a==0){kq.innerHTML="❌ Không hợp lệ";return;}

  if(d<0) kq.innerHTML="❌ Vô nghiệm";
  else if(d==0) kq.innerHTML="✅ x="+(-b/(2*a));
  else{
    let x1=(-b+Math.sqrt(d))/(2*a);
    let x2=(-b-Math.sqrt(d))/(2*a);
    kq.innerHTML=`✨ x1=${x1} | x2=${x2}`;
  }

  drawGraph(a,b,c);
}

// DRAW
function drawGraph(a,b,c){
  let ctx=graph.getContext("2d");
  let W=graph.width,H=graph.height;

  ctx.clearRect(0,0,W,H);

  // grid
  ctx.strokeStyle="#eee";
  for(let i=-W;i<W;i+=scale){
    ctx.beginPath();
    ctx.moveTo(i+W/2+offsetX,0);
    ctx.lineTo(i+W/2+offsetX,H);
    ctx.stroke();
  }

  for(let i=-H;i<H;i+=scale){
    ctx.beginPath();
    ctx.moveTo(0,i+H/2+offsetY);
    ctx.lineTo(W,i+H/2+offsetY);
    ctx.stroke();
  }

  // trục
  ctx.strokeStyle="#999";
  ctx.beginPath();
  ctx.moveTo(0,H/2+offsetY);
  ctx.lineTo(W,H/2+offsetY);
  ctx.moveTo(W/2+offsetX,0);
  ctx.lineTo(W/2+offsetX,H);
  ctx.stroke();

  // parabol
  ctx.beginPath();
  ctx.strokeStyle="#0072ff";
  ctx.lineWidth=3;

  for(let x=-W;x<W;x++){
    let realX=(x-offsetX)/scale;
    let y=a*realX*realX+b*realX+c;

    let px=x;
    let py=H/2 - y*scale + offsetY;

    if(x==-W) ctx.moveTo(px,py);
    else ctx.lineTo(px,py);
  }

  ctx.stroke();

  // 🔴 ĐỈNH
  let xv=-b/(2*a);
  let yv=a*xv*xv+b*xv+c;

  let px=W/2 + xv*scale + offsetX;
  let py=H/2 - yv*scale + offsetY;

  ctx.beginPath();
  ctx.arc(px,py,5,0,Math.PI*2);
  ctx.fillStyle="red";
  ctx.fill();

  ctx.fillStyle="black";
  ctx.fillText(`(${xv.toFixed(2)}, ${yv.toFixed(2)})`, px+5, py-5);
}

// 🎯 TỌA ĐỘ CHUỘT
graph.addEventListener("mousemove",function(e){
  let rect=graph.getBoundingClientRect();
  let x=e.clientX-rect.left;
  let y=e.clientY-rect.top;

  let realX=(x-graph.width/2-offsetX)/scale;
  let realY=(graph.height/2-y+offsetY)/scale;

  document.getElementById("coord").innerHTML =
    `📍 (${realX.toFixed(2)}, ${realY.toFixed(2)})`;
});

// ZOOM
graph.addEventListener("wheel",e=>{
  e.preventDefault();
  scale*=e.deltaY<0?1.1:0.9;
  solve();
});

// DRAG
graph.addEventListener("mousedown",e=>{
  isDragging=true;
  startX=e.clientX;
  startY=e.clientY;
});

graph.addEventListener("mousemove",e=>{
  if(isDragging){
    offsetX+=e.clientX-startX;
    offsetY+=e.clientY-startY;
    startX=e.clientX;
    startY=e.clientY;
    solve();
  }
});

graph.addEventListener("mouseup",()=>isDragging=false);
graph.addEventListener("mouseleave",()=>isDragging=false);

// RESET
function resetGraph(){
  scale=20;
  offsetX=0;
  offsetY=0;
  solve();
}

</script>

</body>
</html>
