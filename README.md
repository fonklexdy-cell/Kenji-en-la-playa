
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kenji en la Playa - Desafío Nica</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { background: linear-gradient(135deg, #00d2ff, #3a7bd5); min-height: 100vh; display: flex; justify-content: center; align-items: center; padding: 15px; }
        .contenedor { background: white; width: 100%; max-width: 600px; border-radius: 25px; padding: 25px; box-shadow: 0 10px 30px rgba(0,0,0,.3); text-align: center; }
        h1 { color: #0277bd; margin-bottom: 20px; }
        .avatar { font-size: 60px; margin: 10px 0; animation: wave 2s infinite; }
        @keyframes wave { 0%, 100% { transform: rotate(-5deg); } 50% { transform: rotate(5deg); } }
        .panel { display: flex; gap: 10px; justify-content: center; margin-bottom: 20px; }
        .caja { background: #e1f5fe; padding: 10px; border-radius: 10px; font-size: 14px; font-weight: bold; flex: 1; }
        .pregunta { font-size: 20px; font-weight: bold; margin: 20px 0; color: #333; }
        .opcion { width: 100%; padding: 15px; margin-bottom: 10px; border: 2px solid #b3e5fc; border-radius: 12px; background: white; font-size: 18px; cursor: pointer; transition: 0.3s; }
        .opcion:hover { background: #e1f5fe; border-color: #03a9f4; }
        .boton { width: 100%; padding: 15px; border: none; border-radius: 12px; background: #ff9800; color: white; font-size: 18px; font-weight: bold; cursor: pointer; }
        .resultado { display: none; }
        .btn-flotante-mateken {
    position: fixed;
    bottom: 25px;
    right: 25px;
    background-color: #FFD166;
    color: #073B4C;
    padding: 15px 22px;
    font-size: 1.1rem;
    font-weight: bold;
    text-decoration: none;
    border-radius: 50px;
    box-shadow: 0 6px 15px rgba(0,0,0,0.4);
    display: flex;
    align-items: center;
    gap: 10px;
    transition: transform 0.2s, background-color 0.2s;
    z-index: 9999;
    font-family: sans-serif;
}
.btn-flotante-mateken:hover {
    transform: scale(1.1);
    background-color: #fffde7;
}
    </style>
</head>
<body>

<div class="contenedor">
    <h1>🏖️ Kenji en la Playa</h1>
    
    <div id="inicio">
        <div class="avatar">😎</div>
        <p>¡Ayuda a Kenji con las cuentas de su paseo a la playa!</p><br>
        <button class="boton" onclick="iniciarJuego()">▶ Empezar Aventura</button>
    </div>

    <div id="juego" style="display:none;">
        <div class="panel">
            <div class="caja">⭐ <span id="puntos">0</span></div>
            <div class="caja">🌊 <span id="numero">1</span>/10</div>
        </div>
        <div id="avatar" class="avatar">🏖️</div>
        <div id="pregunta" class="pregunta"></div>
        <div id="opciones"></div>
    </div>

    <div id="resultado" class="resultado">
        <h2 id="rango"></h2>
        <div class="avatar" id="insignia"></div>
        <h1 id="porcentaje"></h1>
        <button class="boton" onclick="location.reload()">🔄 Jugar de nuevo</button>
    </div>
</div>

<script>
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    
    // Función para reproducir sonidos sintéticos
    function playTone(freq, type, duration) {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = type;
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
        osc.start();
        osc.stop(audioCtx.currentTime + duration);
    }

    const sonidos = {
        correcto: () => playTone(880, 'sine', 0.3),
        error: () => playTone(150, 'sawtooth', 0.5),
        victoria: () => { playTone(500, 'sine', 0.2); setTimeout(() => playTone(700, 'sine', 0.2), 200); setTimeout(() => playTone(900, 'sine', 0.5), 400); }
    };

    const actividades = [
        { icon: "🥥", q: "Compra 3 cocos a C$40 c/u. ¿Total?", r: 120 },
        { icon: "🍦", q: "Compra 5 helados a C$30 c/u. ¿Total?", r: 150 },
        { icon: "🍟", q: "2 platos de papas a C$80 c/u. ¿Total?", r: 160 },
        { icon: "🎟️", q: "Entrada playa C$50 x 4 personas. ¿Total?", r: 200 },
        { icon: "🏐", q: "Pelota C$120. Pagas con C$200. ¿Vuelto?", r: 80 },
        { icon: "🥤", q: "4 refrescos a C$25 c/u. ¿Total?", r: 100 },
        { icon: "🕶️", q: "Lentes C$150. Tenías C$200. ¿Te queda?", r: 50 },
        { icon: "🛶", q: "Kayak C$200 por 2 horas. ¿Total?", r: 200 },
        { icon: "🐟", q: "Pescado C$250. Pagas con C$300. ¿Vuelto?", r: 50 },
        { icon: "☀️", q: "Bloqueador C$130. Pagas con C$200. ¿Vuelto?", r: 70 }
    ];

    let preguntas = [], indice = 0, aciertos = 0;

    function iniciarJuego() {
        audioCtx.resume();
        document.getElementById("inicio").style.display = "none";
        document.getElementById("juego").style.display = "block";
        preguntas = [...actividades].sort(() => Math.random() - 0.5);
        indice = 0; aciertos = 0;
        mostrarPregunta();
    }

    function mostrarPregunta() {
        let p = preguntas[indice];
        document.getElementById("numero").innerText = indice + 1;
        document.getElementById("pregunta").innerHTML = `${p.icon} ${p.q}`;
        let opts = [p.r, p.r + 20, p.r - 20, p.r + 50].sort(() => Math.random() - 0.5);
        let html = "";
        opts.forEach(v => html += `<button class="opcion" onclick="verificar(${v}, ${p.r})">C$ ${v}</button>`);
        document.getElementById("opciones").innerHTML = html;
    }

    function verificar(sel, real) {
        if(sel === real) {
            aciertos++;
            sonidos.correcto();
            document.getElementById("puntos").innerText = aciertos * 10;
        } else {
            sonidos.error();
        }
        indice++;
        if(indice < 10) mostrarPregunta();
        else finalizar();
    }

    function finalizar() {
        sonidos.victoria();
        document.getElementById("juego").style.display = "none";
        document.getElementById("resultado").style.display = "block";
        let pct = (aciertos / 10) * 100;
        document.getElementById("porcentaje").innerText = pct + "% Aciertos";
        document.getElementById("rango").innerText = pct >= 80 ? "¡Salvavidas Experto!" : "¡Buen Paseo!";
        document.getElementById("insignia").innerText = pct >= 80 ? "🥇" : "🥉";
    }
</script>

<a href="https://mateken2.pages.dev/" target="_blank" class="btn-flotante-mateken">
    🤖 Ir a MateKen2
</a>
</body>
</html>

