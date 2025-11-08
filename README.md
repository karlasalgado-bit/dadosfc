<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Juego de Dados por Etapas</title>
<style>
html, body {
  height: 100%;
  margin: 0;
  overflow: hidden;
}
body {
  font-family: Arial, sans-serif;
  color: #f1f5f9;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  position: relative;
}

/* ---- Fondo animado ---- */
#bgCanvas {
  position: fixed;
  top: 0; left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  background: radial-gradient(circle at center, #0f172a, #020617);
}

/* ---- Contenido principal ---- */
h1 { font-size: 2rem; margin-bottom: 15px; z-index: 1; }

.dice-container {
  display: flex;
  gap: 20px;
  justify-content: center;
  flex-wrap: wrap;
  width: 100%;
  z-index: 1;
}

/* ---- ESTILO DE LOS DADOS ---- */
.die {
  width: 40vw;
  aspect-ratio: 1/1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 4vw;
  border-radius: 20px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.6);
  transition: transform 0.5s, box-shadow 0.3s;
  user-select: none;
  padding: 10px;
  text-align: center;
  overflow: hidden;
  word-wrap: break-word;
  line-height: 1.1;
}
#die1 { background: #10b981; color: #fff; }  /* Verde menta */
#die2 { background: #a855f7; color: #fff; }  /* Lila */
.rolling { transform: rotate(720deg) scale(1.1); box-shadow: 0 0 30px rgba(255,255,255,0.7); }

button {
  margin-top: 20px;
  background: #38bdf8;
  color: #0f172a;
  border: none;
  padding: 14px 24px;
  border-radius: 12px;
  cursor: pointer;
  font-size: 1.2rem;
  z-index: 1;
  transition: background 0.3s;
}
button:hover { background: #7dd3fc; }

#etapa, #mensaje {
  margin-top: 15px;
  z-index: 1;
}
#etapa { font-size: 1.5rem; color: #facc15; }
#mensaje { font-size: 1.2rem; color: #4ade80; }
</style>
</head>
<body>

<!-- Fondo de luces animadas -->
<canvas id="bgCanvas"></canvas>

<h1>🎲 Juego de Dados con Etapas</h1>

<div class="dice-container">
  <div id="die1" class="die">?</div>
  <div id="die2" class="die">?</div>
</div>

<button onclick="rollDice()">Lanzar</button>
<button onclick="enableShakeSensor()">📱 Activar sensor de movimiento</button>

<div id="etapa">Etapa: 1</div>
<div id="mensaje"></div>

<script>
/* ======= ANIMACIÓN DE LUCES ======= */
const canvas = document.getElementById("bgCanvas");
const ctx = canvas.getContext("2d");
let w, h, lights = [];

function resizeCanvas(){
  w = canvas.width = window.innerWidth;
  h = canvas.height = window.innerHeight;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

for(let i=0; i<40; i++){
  lights.push({
    x: Math.random()*w,
    y: Math.random()*h,
    r: Math.random()*3 + 2,
    dx: (Math.random()-0.5)*0.5,
    dy: (Math.random()-0.5)*0.5,
    color: `hsl(${Math.random()*360}, 80%, 60%)`
  });
}

function animateLights(){
  ctx.clearRect(0,0,w,h);
  lights.forEach(l=>{
    ctx.beginPath();
    ctx.arc(l.x, l.y, l.r, 0, Math.PI*2);
    ctx.fillStyle = l.color;
    ctx.shadowColor = l.color;
    ctx.shadowBlur = 15;
    ctx.fill();
    l.x += l.dx;
    l.y += l.dy;
    if(l.x<0 || l.x>w) l.dx *= -1;
    if(l.y<0 || l.y>h) l.dy *= -1;
  });
  requestAnimationFrame(animateLights);
}
animateLights();

/* ======= LÓGICA DE LOS DADOS ======= */
let faces1 = ["Pararse y sentarse x 10","Elevar talones x 5","Elevar brazos x 10","Marcha 10 segundos","Remo de brazos x 10","Flexión de cadera x 10"];
let faces2 = ["Resuelva 3 x 5","Nombre la ciudad de Colombia","¿De qué país es la torre Eiffel?","Resuelva 6 + 8","Resuelva 6 : 2","¿Qué se celebra el 18 de Septiembre?"];

let combinaciones = new Set();
let etapa = 1;

function rollDice(){
  const die1 = document.getElementById("die1");
  const die2 = document.getElementById("die2");
  die1.classList.add("rolling");
  die2.classList.add("rolling");
  setTimeout(()=>{
    die1.classList.remove("rolling");
    die2.classList.remove("rolling");
    const val1 = faces1[Math.floor(Math.random()*faces1.length)];
    const val2 = faces2[Math.floor(Math.random()*faces2.length)];
    die1.textContent = val1;
    die2.textContent = val2;
    ajustarTamañoTexto(die1);
    ajustarTamañoTexto(die2);
    registrarCombinacion(val1, val2);
  }, 500);
}

function ajustarTamañoTexto(dado){
  const texto = dado.textContent.length;
  if (texto < 15) dado.style.fontSize = "5vw";
  else if (texto < 30) dado.style.fontSize = "4vw";
  else if (texto < 45) dado.style.fontSize = "3.5vw";
  else dado.style.fontSize = "3vw";
}

function registrarCombinacion(v1, v2){
  const combo = `${v1}-${v2}`;
  if(!combinaciones.has(combo)){
    combinaciones.add(combo);
    document.getElementById("mensaje").textContent = `¡Nueva combinación: ${combo}! (${combinaciones.size}/4)`;
  } else {
    document.getElementById("mensaje").textContent = `Combinación repetida (${combinaciones.size}/4)`;
  }

  if(combinaciones.size >= 4){
    avanzarEtapa();
  }
}

function avanzarEtapa(){
  etapa++;
  combinaciones.clear();
  document.getElementById("etapa").textContent = `Etapa: ${etapa}`;
  document.getElementById("mensaje").textContent = `🎉 ¡Avanzaste a la etapa ${etapa}! 🎉`;
  const colores = [
    ["#10b981","#a855f7"],
    ["#22c55e","#c084fc"],
    ["#059669","#9333ea"],
    ["#16a34a","#a855f7"]
  ];
  const par = colores[etapa % colores.length];
  document.getElementById("die1").style.background = par[0];
  document.getElementById("die2").style.background = par[1];
}

/* ======= SENSOR DE MOVIMIENTO ======= */
let lastShake = 0;
const shakeThreshold = 15; 
let sensorActive = false;

function enableShakeSensor(){
  if (typeof DeviceMotionEvent.requestPermission === 'function') {
    DeviceMotionEvent.requestPermission().then(response => {
      if (response === 'granted') {
        window.addEventListener('devicemotion', handleMotion);
        sensorActive = true;
        alert('Sensor activado ✅ Agita el teléfono para lanzar los dados.');
      } else {
        alert('Permiso denegado ❌');
      }
    }).catch(console.error);
  } else {
    window.addEventListener('devicemotion', handleMotion);
    sensorActive = true;
    alert('Sensor activado ✅ Agita el teléfono para lanzar los dados.');
  }
}

function handleMotion(event){
  const acc = event.accelerationIncludingGravity;
  if(!acc) return;
  const total = Math.sqrt(acc.x**2 + acc.y**2 + acc.z**2);
  const now = Date.now();
  if(total > shakeThreshold && (now - lastShake) > 1500){
    lastShake = now;
    rollDice();
  }
}
</script>

</body>
<!-- ======= CÓDIGO QR FLOTANTE ======= -->
<div id="qrFloating"></div>

<!-- Librería QRCode.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<script>
  // 💡 Reemplaza este enlace cuando el juego esté online
  const gameURL = "https://example.com/mi-juego-de-dados";

  // Crear el QR dentro del contenedor flotante
  const qrDiv = document.getElementById("qrFloating");
  new QRCode(qrDiv, {
    text: gameURL,
    width: 120,
    height: 120,
    colorDark : "#22d3ee",
    colorLight : "#0f172a",
    correctLevel : QRCode.CorrectLevel.H
  });

  // Estilo flotante en la esquina
  qrDiv.style.position = "fixed";
  qrDiv.style.bottom = "20px";
  qrDiv.style.right = "20px";
  qrDiv.style.zIndex = "10";
  qrDiv.style.padding = "8px";
  qrDiv.style.borderRadius = "15px";
  qrDiv.style.boxShadow = "0 0 20px rgba(34,211,238,0.6)";
  qrDiv.style.transition = "transform 0.3s ease";
  qrDiv.style.background = "rgba(15,23,42,0.8)";
  qrDiv.onmouseenter = () => qrDiv.style.transform = "scale(1.2)";
  qrDiv.onmouseleave = () => qrDiv.style.transform = "scale(1)";
</script>


</html>
