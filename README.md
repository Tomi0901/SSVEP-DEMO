<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>SSVEP Demo - Estímulos Visuales</title>

<style>
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 25px;
    display: flex;
    flex-direction: column;
    align-items: center;
    transition: background 0.3s, color 0.3s;
}

.light {
    background: #ffffff;
    color: #000000;
}

.dark {
    background: #121212;
    color: #e6e6e6;
}

h1 {
    margin-bottom: 5px;
    text-align: center;
}

button {
    padding: 12px 20px;
    margin: 8px;
    font-size: 16px;
    cursor: pointer;
    border: none;
    background: #222;
    color: white;
    border-radius: 5px;
}
button:hover {
    background: #444;
}

canvas {
    border: 1px solid #000;
    margin-top: 25px;
}

#containers {
    display: flex;
    justify-content: space-around;
    width: 100%;
    max-width: 1100px;
    margin-top: 20px;
}

#stimulusBox {
    width: 200px;
    height: 200px;
    border: 3px solid black;
    margin-right: 40px;
}

#analysisBox, #infoBox {
    border: 1px solid #222;
    padding: 15px;
    margin-top: 20px;
    width: 850px;
    background: #f3f3f3;
}

.dark #analysisBox, .dark #infoBox {
    background: #1e1e1e;
    border-color: #555;
}
</style>
</head>
<body class="light">

<h1>Comunicación Cerebro-Computador en base a estímulos visuales</h1>

<div id="meta">
    <p><strong>Autores:</strong> Bravo, T. – t.bravod@udd.cl — Burgos, V. – v.burgosb@udd.cl — Yabrudy, E. – e.yabrudyr@udd.cl</p>
    <p><strong>Profesores guía:</strong> Gustavo Adolfo Castro P. — René Ignacio Pino P.</p>
</div>

<button onclick="toggleDarkMode()">Cambiar modo claro/oscuro</button>

<h2>Selecciona la frecuencia del estímulo</h2>
<div>
    <button onclick="setFrequency(10)">10 Hz</button>
    <button onclick="setFrequency(12)">12 Hz</button>
    <button onclick="setFrequency(15)">15 Hz</button>
    <button onclick="setFrequency(20)">20 Hz</button>
</div>

<!-- CONTENEDOR ESTÍMULO + CURVA -->
<div id="containers">
    <div>
        <h3>Estímulo visual</h3>
        <div id="stimulusBox"></div>
    </div>

    <div>
        <h3>Función sinusoidal que modela el estímulo</h3>
        <canvas id="ssvepCanvas" width="800" height="300"></canvas>
    </div>
</div>

<!-- ANÁLISIS -->
<div id="analysisBox">
    <h3>Análisis matemático de la función</h3>
    <p><strong>Frecuencia:</strong> <span id="freqVal"></span></p>
    <p><strong>Período:</strong> <span id="periodVal"></span></p>
    <p><strong>Amplitud:</strong> <span id="ampVal"></span></p>
    <p><strong>Desfase:</strong> <span id="phaseVal"></span></p>
    <p><strong>Monotonía (t=0):</strong> <span id="monoVal"></span></p>
</div>

<!-- PANEL EXPLICATIVO -->
<div id="infoBox">
    <h3>¿Qué es un SSVEP?</h3>
    <p>
        Los Potenciales Evocados Visuales de Estado Estacionario (SSVEP) son respuestas oscilatorias
        del cerebro generadas cuando una persona observa un estímulo visual parpadeante a una
        frecuencia constante.  
    </p>
    <p>
        El cerebro replica dicha frecuencia, lo que permite utilizar estos patrones en interfaces
        cerebro-computador (BCI) para seleccionar opciones, escribir o interactuar sin usar músculos.
    </p>
    <p>
        En esta página se muestran:
        <br>— El estímulo visual real (parpadeo).  
        <br>— Su correspondiente modelamiento matemático como onda sinusoidal.  
        <br>— Un análisis automático de su forma (amplitud, período, fase y monotonía).  
    </p>
</div>

<script>
/* ===== PARÁMETROS ===== */
const canvas = document.getElementById("ssvepCanvas");
const ctx = canvas.getContext("2d");
const box = document.getElementById("stimulusBox");

let freq = 10;

const duration = 5;
const minGray = 0.318;
const maxGray = 0.613;

const amp = (maxGray - minGray) / 2;
const offset = (maxGray + minGray) / 2;
const phase = Math.PI / 2;

/* ===== FUNCIÓN SSVEP ===== */
function f(t) {
    return amp * Math.sin(2 * Math.PI * freq * t + phase) + offset;
}

/* ===== ESTÍMULO PARPADEANTE ===== */
function animateStimulus() {
    const t = performance.now() / 1000;
    const luminance = f(t);

    const gray = Math.round(luminance * 255);
    box.style.background = `rgb(${gray},${gray},${gray})`;

    requestAnimationFrame(animateStimulus);
}
animateStimulus();

/* ===== GRAFICAR EJES ===== */
function drawAxes() {
    ctx.strokeStyle = "#000";
    ctx.lineWidth = 1;

    ctx.beginPath();
    ctx.moveTo(60, 280);
    ctx.lineTo(780, 280);
    ctx.stroke();

    ctx.fillText("Tiempo (s)", 720, 295);

    for (let t = 0; t <= duration; t++) {
        let x = 60 + t * (720 / duration);
        ctx.fillText(t, x - 3, 295);

        ctx.beginPath();
        ctx.moveTo(x, 275);
        ctx.lineTo(x, 285);
        ctx.stroke();
    }

    ctx.beginPath();
    ctx.moveTo(60, 40);
    ctx.lineTo(60, 280);
    ctx.stroke();

    ctx.fillText("Escala de grises", 5, 35);

    for (let i = 0; i <= 10; i++) {
        const val = minGray + (i / 10) * (maxGray - minGray);
        const y = 280 - (val - minGray) * (240 / (maxGray - minGray));

        ctx.fillText(val.toFixed(3), 5, y + 3);

        ctx.beginPath();
        ctx.moveTo(55, y);
        ctx.lineTo(60, y);
        ctx.stroke();
    }
}

/* ===== GRAFICAR ONDA ===== */
function animateCurve() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawAxes();

    ctx.strokeStyle = "#000";
    ctx.lineWidth = 2;
    ctx.beginPath();

    const now = performance.now() / 1000;

    for (let px = 60; px <= 780; px++) {
        let tNorm = ((px - 60) / 720) * duration;
        let t = (tNorm + now) % duration;

        let yVal = f(t);
        let y = 280 - (yVal - minGray) * (240 / (maxGray - minGray));

        if (px === 60) ctx.moveTo(px, y);
        else ctx.lineTo(px, y);
    }

    ctx.stroke();
    requestAnimationFrame(animateCurve);
}
animateCurve();

/* ===== ANÁLISIS ===== */
function updateAnalysis() {
    document.getElementById("freqVal").innerText = freq + " Hz";
    document.getElementById("periodVal").innerText = (1 / freq).toFixed(3) + " s";
    document.getElementById("ampVal").innerText = amp.toFixed(3);
    document.getElementById("phaseVal").innerText = phase.toFixed(3);

    const deriv = amp * Math.cos(phase) * 2 * Math.PI * freq;
    document.getElementById("monoVal").innerText = deriv > 0 ? "Creciente" : "Decreciente";
}

/* ===== CAMBIO DE FRECUENCIA ===== */
function setFrequency(f) {
    freq = f;
    updateAnalysis();
}
setFrequency(10);

/* ===== MODO OSCURO ===== */
function toggleDarkMode() {
    const body = document.body;
    body.classList.toggle("dark");
    body.classList.toggle("light");
}
</script>

</body>
</html>
