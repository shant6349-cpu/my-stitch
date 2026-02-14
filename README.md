<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch: Fullscreen Edition</title>
    <style>
        /* ГЛОБАЛЬНЫЕ СТИЛИ ДЛЯ ШИРОКОГО ЭКРАНА */
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            width: 100vw;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            transition: background 1s ease;
            background: #2c3e50;
        }

        /* ФОНЫ КОМНАТ */
        body.home { background: radial-gradient(circle, #ff9966, #ff5e62); }
        body.beach { background: radial-gradient(circle, #4ca1af, #2c3e50); }
        body.space { background: radial-gradient(circle, #0f0c29, #302b63, #24243e); }

        /* ГЛАВНЫЙ КОНТЕЙНЕР НА ВЕСЬ ЭКРАН */
        #pet-stage {
            width: 100%;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            box-sizing: border-box;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(8px);
            -webkit-backdrop-filter: blur(8px);
        }

        /* ПОЛОСКИ СТАТИСТИКИ */
        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            width: 100%;
            margin-top: 10px;
        }
        .stat-item { text-align: left; }
        .stat-label { font-size: 11px; font-weight: bold; color: white; text-shadow: 1px 1px 2px rgba(0,0,0,0.5); }
        .bar-bg { width: 100%; height: 10px; background: rgba(0,0,0,0.3); border-radius: 5px; overflow: hidden; margin-top: 4px; }
        .fill { height: 100%; width: 80%; transition: width 0.5s ease; }

        /* СТИЧ ПО ЦЕНТРУ */
        #stitch-container {
            position: relative;
            width: 200px;
            height: 180px;
            margin: auto;
            cursor: pointer;
            touch-action: none;
        }

        .head {
            position: absolute; width: 100%; height: 100%;
            background: radial-gradient(circle at 35% 30%, #5d8aff, #3a56d4 60%, #1e2a78 100%);
            border-radius: 50% 50% 46% 46%; z-index: 5;
            box-shadow: inset -5px -10px 20px rgba(0,0,0,0.6), 0 10px 30px rgba(0,0,0,0.4);
        }

        .ear { position: absolute; top: -20px; width: 60px; height: 130px; background: #3a56d4; border-radius: 100% 20% 100% 20%; z-index: 1; transition: 0.5s; }
        .ear.left { left: -40px; transform: rotate(-35deg); }
        .ear.right { right: -40px; transform: rotate(35deg) scaleX(-1); }

        .eye-patch { position: absolute; top: 25px; width: 70px; height: 90px; background: rgba(0, 0, 0, 0.15); border-radius: 50%; z-index: 6; }
        .eye-patch.left { left: 15px; transform: rotate(12deg); }
        .eye-patch.right { right: 15px; transform: rotate(-12deg); }

        .eye { position: absolute; top: 18px; left: 12px; width: 46px; height: 58px; background: #080808; border-radius: 50%; transition: 0.15s; }
        .eye::before { content: ''; position: absolute; top: 8px; left: 8px; width: 12px; height: 22px; background: white; border-radius: 50%; opacity: 0.9; }

        .nose { position: absolute; top: 100px; left: 50%; transform: translateX(-50%); width: 46px; height: 26px; background: #1b2661; border-radius: 50% 50% 40% 40%; z-index: 7; }
        .mouth { position: absolute; bottom: 35px; left: 50%; transform: translateX(-50%); width: 45px; height: 6px; border-bottom: 4px solid #1b2661; border-radius: 50%; z-index: 7; transition: 0.3s; }

        /* АНИМАЦИИ */
        .purring { animation: vibe 0.1s infinite; }
        @keyframes vibe { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.03); } }
        .blinking .eye { height: 2px; top: 45px; }
        .eating .mouth { height: 20px; background: #000; border-radius: 50%; }

        /* БЛОК КНОПОК ВНИЗУ */
        .controls-area { width: 100%; margin-bottom: 10px; }
        .label-text { font-size: 10px; font-weight: bold; opacity: 0.7; margin: 10px 0 5px; text-transform: uppercase; letter-spacing: 1px; color: white; }
        .btn-group { display: flex; justify-content: center; gap: 8px; flex-wrap: wrap; }
        
        button {
            flex: 1;
            min-width: 80px;
            padding: 15px 5px;
            border: none;
            border-radius: 15px;
            background: rgba(255, 255, 255, 0.2);
            color: white;
            font-weight: bold;
            font-size: 12px;
            border: 1px solid rgba(255,255,255,0.1);
            cursor: pointer;
            transition: 0.2s;
        }
        button:active { background: #3a56d4; transform: scale(0.95); }

        #status { margin: 15px 0; font-size: 1.2rem; text-shadow: 2px 2px 4px rgba(0,0,0,0.3); }
    </style>
</head>
<body class="home">

<div id="pet-stage">
    <div class="stats-grid">
        <div class="stat-item">
            <span class="stat-label">🍖 ГОЛОД</span>
            <div class="bar-bg"><div id="h-bar" class="fill" style="background: #ff5e62; width: 80%;"></div></div>
        </div>
        <div class="stat-item">
            <span class="stat-label">🌈 СЧАСТЬЕ</span>
            <div class="bar-bg"><div id="ha-bar" class="fill" style="background: #a8e063; width: 80%;"></div></div>
        </div>
    </div>

    <div id="stitch-container" 
         onmousedown="startPurr()" onmouseup="stopPurr()" 
         ontouchstart="startPurr()" ontouchend="stopPurr()">
        <div id="stitch-face">
            <div class="ear left"></div><div class="ear right"></div>
            <div class="head">
                <div class="eye-patch left"><div class="eye"></div></div>
                <div class="eye-patch right"><div class="eye"></div></div>
                <div class="nose"></div>
                <div id="mouth" class="mouth"></div>
            </div>
        </div>
    </div>

    <h3 id="status">Алоха! ✨</h3>

    <div class="controls-area">
        <div class="label-text">Выбрать локацию</div>
        <div class="btn-group">
            <button onclick="setRoom('home')">🏠 ДОМ</button>
            <button onclick="setRoom('beach')">🏖️ ПЛЯЖ</button>
            <button onclick="setRoom('space')">🚀 КОСМОС</button>
        </div>

        <div class="label-text">Действия</div>
        <div class="btn-group">
            <button onclick="play('feed')">🥪 ЕДА</button>
            <button onclick="play('uke')">🎸 ГИТАРА</button>
            <button onclick="play('doll')">🧸 КУКЛА</button>
            <button onclick="play('sleep')">💤 СОН</button>
        </div>
    </div>
</div>

<script>
    let stats = { hunger: 80, happy: 80 };
    let audioCtx, osc;

    // ГОЛОС СТИЧА
    function say(text, p = 1.8, r = 1.1) {
        const m = new SpeechSynthesisUtterance(text);
        m.lang = 'ru-RU'; m.pitch = p; m.rate = r;
        window.speechSynthesis.speak(m);
    }

    // МОРГАНИЕ
    setInterval(() => {
        const face = document.getElementById('stitch-face');
        if (!document.getElementById('stitch-container').classList.contains('purring')) {
            face.classList.add('blinking');
            setTimeout(() => face.classList.remove('blinking'), 150);
        }
    }, 4000);

    // СМЕНА КОМНАТ
    function setRoom(name) {
        document.body.className = name;
        const msgs = { home: "Стич дома! 🏠", beach: "Пляж! Серфинг! 🏄‍♂️", space: "Космос! Абакаба! 🚀" };
        document.getElementById('status').innerText = msgs[name];
        say(name === 'space' ? "Моя лететь домой!" : "Алоха!", 2);
    }

    // ДЕЙСТВИЯ
    function play(act) {
        const s = document.getElementById('status');
        const face = document.getElementById('stitch-face');
        
        if (act === 'feed') {
            stats.hunger = Math.min(100, stats.hunger + 25);
            face.classList.add('eating');
            say("Ммм, ням-ням!", 2, 1.3);
            s.innerText = "Вкусно! 🥪";
            setTimeout(() => face.classList.remove('eating'), 1000);
        }
        else if (act === 'uke') {
            stats.happy = Math.min(100, stats.happy + 30);
            s.innerText = "Играем музыку! 🎸";
            say("Уииии! Абакаба!", 2.2, 1.4);
        }
        else if (act === 'doll') {
            stats.happy = Math.min(100, stats.happy + 15);
            s.innerText = "Охана — это семья 🧸";
            say("Охана значит семья", 0.6, 0.8);
        }
        else if (act === 'sleep') {
            s.innerText = "Стич видит сны... 💤";
            say("Хрррр псссс", 0.5, 0.7);
        }
        updateUI();
    }

    // МУРЧАНИЕ ПРИ НАЖАТИИ
    function startPurr() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        osc = audioCtx.createOscillator();
        let g = audioCtx.createGain();
        osc.type = 'sine'; osc.frequency.setValueAtTime(60, audioCtx.currentTime);
        g.gain.setValueAtTime(0.05, audioCtx.currentTime);
        osc.connect(g); g.connect(audioCtx.destination);
        osc.start();
        document.getElementById('stitch-container').classList.add('purring');
        document.getElementById('status').innerText = "Мрррр... ❤️";
        stats.happy = Math.min(100, stats.happy + 0.2);
        updateUI();
    }

    function stopPurr() {
        if (osc) { osc.stop(); osc.disconnect(); }
        document.getElementById('stitch-container').classList.remove('purring');
    }

    function updateUI() {
        document.getElementById('h-bar').style.width = stats.hunger + '%';
        document.getElementById('ha-bar').style.width = stats.happy + '%';
    }

    // ПАДЕНИЕ СТАТОВ
    setInterval(() => {
        stats.hunger = Math.max(0, stats.hunger - 1);
        stats.happy = Math.max(0, stats.happy - 1);
        updateUI();
    }, 6000);
</script>

</body>
</html>
