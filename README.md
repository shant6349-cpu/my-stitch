<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch Wide & Clear</title>
    <style>
        * { box-sizing: border-box; }
        
        body {
            margin: 0; padding: 0;
            display: flex; justify-content: center; align-items: center;
            min-height: 100vh; width: 100vw;
            font-family: 'Segoe UI', sans-serif;
            overflow: hidden;
            transition: background 0.8s ease;
            background: #2c3e50;
        }

        /* ТЕМЫ КОМНАТ */
        body.home { background: radial-gradient(circle, #ff9966, #ff5e62); }
        body.beach { background: radial-gradient(circle, #4ca1af, #2c3e50); }
        body.space { background: radial-gradient(circle, #0f0c29, #302b63, #24243e); }

        #pet-stage {
            width: 100%; height: 100vh;
            display: flex; flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
        }

        /* СТАТИСТИКА */
        .top-ui {
            display: flex; justify-content: space-between; align-items: flex-start;
            width: 100%; padding-top: 10px;
        }
        .stats { flex-grow: 1; margin: 0 15px; }
        .stat-row { margin-bottom: 8px; }
        .bar-bg { width: 100%; height: 12px; background: rgba(0,0,0,0.3); border-radius: 6px; overflow: hidden; }
        .fill { height: 100%; width: 80%; transition: width 0.5s ease; }
        .lvl-box, .coin-box { 
            background: white; border-radius: 12px; padding: 8px; 
            text-align: center; min-width: 60px; box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        }

        /* СТИЧ */
        #stitch-container {
            position: relative; width: 240px; height: 220px;
            margin: auto; cursor: pointer; touch-action: none;
        }
        .head {
            position: absolute; width: 100%; height: 100%;
            background: radial-gradient(circle at 35% 30%, #5d8aff, #3a56d4 60%, #1e2a78 100%);
            border-radius: 50% 50% 46% 46%; z-index: 5;
            box-shadow: inset -5px -10px 20px rgba(0,0,0,0.6), 0 15px 40px rgba(0,0,0,0.4);
        }
        .ear { position: absolute; top: -20px; width: 70px; height: 150px; background: #3a56d4; border-radius: 100% 20% 100% 20%; z-index: 1; }
        .ear.left { left: -50px; transform: rotate(-35deg); }
        .ear.right { right: -50px; transform: rotate(35deg) scaleX(-1); }
        .eye-patch { position: absolute; top: 35px; width: 80px; height: 100px; background: rgba(0,0,0,0.15); border-radius: 50%; z-index: 6; }
        .eye-patch.left { left: 15px; transform: rotate(12deg); }
        .eye-patch.right { right: 15px; transform: rotate(-12deg); }
        .eye { position: absolute; top: 20px; left: 15px; width: 50px; height: 65px; background: #080808; border-radius: 50%; transition: 0.1s; }
        .eye::after { content: ''; position: absolute; top: 10px; left: 10px; width: 15px; height: 25px; background: white; border-radius: 50%; opacity: 0.9; }
        .nose { position: absolute; top: 120px; left: 50%; transform: translateX(-50%); width: 50px; height: 30px; background: #1b2661; border-radius: 50%; z-index: 7; }
        .mouth { position: absolute; bottom: 45px; left: 50%; transform: translateX(-50%); width: 55px; height: 6px; border-bottom: 4px solid #1b2661; border-radius: 50%; z-index: 7; }

        /* ЭФФЕКТЫ */
        .purring { animation: vibrate 0.1s infinite; }
        @keyframes vibrate { 0% { transform: scale(1); } 50% { transform: scale(1.03); } 100% { transform: scale(1); } }
        .eating .mouth { height: 25px; background: #000; border-radius: 50%; }

        /* КНОПКИ */
        .controls { width: 100%; padding-bottom: 10px; }
        .btn-row { display: flex; gap: 10px; margin-top: 10px; }
        button {
            flex: 1; padding: 18px 5px; border: none; border-radius: 20px;
            background: rgba(255, 255, 255, 0.2); color: white; font-weight: bold;
            font-size: 14px; border: 1px solid rgba(255,255,255,0.2);
        }
        button:active { background: #3a56d4; transform: scale(0.95); }
        #status-text { color: white; font-size: 1.4rem; font-weight: bold; text-shadow: 2px 2px 4px rgba(0,0,0,0.5); margin: 10px 0; }
    </style>
</head>
<body class="home">

<div id="pet-stage">
    <div class="top-ui">
        <div class="lvl-box"><small style="color:gray">LVL</small><br><b style="color:#e91e63; font-size:20px">2</b></div>
        <div class="stats">
            <div class="stat-row"><div class="bar-bg"><div id="h-bar" class="fill" style="background:#ff5e62"></div></div></div>
            <div class="stat-row"><div class="bar-bg"><div id="ha-bar" class="fill" style="background:#a8e063"></div></div></div>
        </div>
        <div class="coin-box"><small style="color:gray">💰</small><br><b style="color:#f9d423; font-size:18px">80</b></div>
    </div>

    <div id="stitch-container" onmousedown="startPurr()" onmouseup="stopPurr()" ontouchstart="startPurr()" ontouchend="stopPurr()">
        <div id="stitch-face">
            <div class="ear left"></div><div class="ear right"></div>
            <div class="head">
                <div class="eye-patch left"><div class="eye"></div></div>
                <div class="eye-patch right"><div class="eye"></div></div>
                <div class="nose"></div><div id="mouth" class="mouth"></div>
            </div>
        </div>
    </div>

    <div id="status-text">Алоха! ✨</div>

    <div class="controls">
        <div class="btn-row">
            <button onclick="setWorld('home')">🏠 ДОМ</button>
            <button onclick="setWorld('beach')">🏖️ ПЛЯЖ</button>
            <button onclick="setWorld('space')">🚀 КОСМОС</button>
        </div>
        <div class="btn-row">
            <button onclick="doAction('feed')">🥪 ЕДА</button>
            <button onclick="doAction('play')">🏀 ИГРА</button>
            <button onclick="doAction('hug')">🫂 ОХАНА</button>
            <button onclick="doAction('sleep')">💤 СОН</button>
        </div>
    </div>
</div>

<script>
    let stats = { hunger: 80, happy: 80 };
    let audioCtx, osc;

    // ИСПРАВЛЕННЫЙ ГОЛОС (Медленнее и хриплее)
    function speak(text, pitch = 1.6, rate = 0.8) {
        window.speechSynthesis.cancel(); // Остановить старую речь
        const m = new SpeechSynthesisUtterance(text);
        m.lang = 'ru-RU';
        m.pitch = pitch;
        m.rate = rate; // Замедлили речь
        window.speechSynthesis.speak(m);
    }

    function setWorld(w) {
        document.body.className = w;
        const s = document.getElementById('status-text');
        if(w === 'space') { s.innerText = "Стич летит домой! 🚀"; speak("Моя лететь домой", 1.9); }
        else if(w === 'beach') { s.innerText = "Пляж! Серфинг! 🏄‍♂️"; speak("Абакаба! Пляж", 1.8); }
        else { s.innerText = "Дома уютно! 🏠"; speak("Алоха", 1.6); }
    }

    function doAction(type) {
        const s = document.getElementById('status-text');
        const f = document.getElementById('stitch-face');
        if (type === 'feed') {
            stats.hunger = Math.min(100, stats.hunger + 20);
            f.classList.add('eating');
            speak("Ммм! Очень вкусно", 1.8, 0.7);
            s.innerText = "Ням-ням! 🥪";
            setTimeout(() => f.classList.remove('eating'), 1000);
        } else if (type === 'play') {
            stats.happy = Math.min(100, stats.happy + 25);
            s.innerText = "Уиии! Играем! 🏀";
            speak("Уиии! Абакаба", 2, 0.8);
        } else if (type === 'hug') {
            s.innerText = "Охана — это семья 🫂";
            speak("Охана значит семья", 0.7, 0.7);
        } else if (type === 'sleep') {
            s.innerText = "Стич спит... 💤";
            speak("Хрррр псссс", 0.5, 0.5);
        }
        updateUI();
    }

    function startPurr() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        osc = audioCtx.createOscillator();
        let g = audioCtx.createGain();
        osc.type = 'sine'; osc.frequency.setValueAtTime(60, audioCtx.currentTime);
        g.gain.setValueAtTime(0.04, audioCtx.currentTime);
        osc.connect(g); g.connect(audioCtx.destination);
        osc.start();
        document.getElementById('stitch-container').classList.add('purring');
        document.getElementById('status-text').innerText = "Мррр... ❤️";
    }

    function stopPurr() {
        if (osc) { osc.stop(); osc.disconnect(); }
        document.getElementById('stitch-container').classList.remove('purring');
    }

    function updateUI() {
        document.getElementById('h-bar').style.width = stats.hunger + '%';
        document.getElementById('ha-bar').style.width = stats.happy + '%';
    }
    updateUI();
</script>
</body>
</html>
