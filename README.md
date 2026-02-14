<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch Wide World</title>
    <style>
        /* ОСНОВНАЯ НАСТРОЙКА ЭКРАНА */
        * { box-sizing: border-box; }
        
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            width: 100vw;
            font-family: 'Segoe UI', Roboto, sans-serif;
            overflow: hidden;
            transition: background 0.8s ease;
            background: #2c3e50;
        }

        /* ТЕМЫ КОМНАТ */
        body.home { background: radial-gradient(circle, #ff9966, #ff5e62); }
        body.beach { background: radial-gradient(circle, #4ca1af, #2c3e50); }
        body.space { background: radial-gradient(circle, #0f0c29, #302b63, #24243e); }

        /* ВЕСЬ ИНТЕРФЕЙС */
        #pet-stage {
            width: 100%;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 15px;
            background: rgba(255, 255, 255, 0.08);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
        }

        /* ВЕРХНЯЯ ПАНЕЛЬ СТАТИСТИКИ */
        .stats-panel {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            width: 100%;
            padding: 10px;
        }
        .stat-label { font-size: 12px; font-weight: 800; color: white; text-shadow: 1px 1px 3px rgba(0,0,0,0.5); display: block; margin-bottom: 4px; }
        .bar-bg { width: 100%; height: 12px; background: rgba(0,0,0,0.3); border-radius: 6px; overflow: hidden; }
        .fill { height: 100%; width: 80%; transition: width 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); }

        /* ЦЕНТР: СТИЧ */
        #stitch-container {
            position: relative;
            width: 220px;
            height: 200px;
            margin: auto;
            cursor: pointer;
            touch-action: none; /* Чтобы телефон не скроллил при мурчании */
        }

        /* Лицо Стича */
        .head {
            position: absolute; width: 100%; height: 100%;
            background: radial-gradient(circle at 35% 30%, #5d8aff, #3a56d4 60%, #1e2a78 100%);
            border-radius: 50% 50% 46% 46%; z-index: 5;
            box-shadow: inset -5px -10px 20px rgba(0,0,0,0.6), 0 15px 35px rgba(0,0,0,0.5);
        }
        .ear { position: absolute; top: -20px; width: 65px; height: 140px; background: #3a56d4; border-radius: 100% 20% 100% 20%; z-index: 1; transition: 0.5s; }
        .ear.left { left: -45px; transform: rotate(-35deg); }
        .ear.right { right: -45px; transform: rotate(35deg) scaleX(-1); }
        .eye-patch { position: absolute; top: 30px; width: 75px; height: 95px; background: rgba(0, 0, 0, 0.12); border-radius: 50%; z-index: 6; }
        .eye-patch.left { left: 15px; transform: rotate(12deg); }
        .eye-patch.right { right: 15px; transform: rotate(-12deg); }
        .eye { position: absolute; top: 18px; left: 14px; width: 48px; height: 62px; background: #080808; border-radius: 50%; transition: 0.1s; }
        .eye::after { content: ''; position: absolute; top: 8px; left: 8px; width: 14px; height: 24px; background: white; border-radius: 50%; opacity: 0.9; }
        .nose { position: absolute; top: 110px; left: 50%; transform: translateX(-50%); width: 48px; height: 28px; background: #1b2661; border-radius: 50% 50% 40% 40%; z-index: 7; }
        .mouth { position: absolute; bottom: 40px; left: 50%; transform: translateX(-50%); width: 50px; height: 6px; border-bottom: 4px solid #1b2661; border-radius: 50%; z-index: 7; transition: 0.3s; }

        /* ЭФФЕКТЫ */
        .purring-active { animation: shake 0.1s infinite; }
        @keyframes shake { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.04); } }
        .is-blinking .eye { height: 2px; top: 48px; }
        .is-eating .mouth { height: 22px; background: #000; border-radius: 50%; }

        /* НИЖНЯЯ ПАНЕЛЬ КНОПОК */
        .controls { width: 100%; padding-bottom: 10px; }
        .label { font-size: 10px; font-weight: bold; color: white; text-transform: uppercase; letter-spacing: 2px; opacity: 0.6; margin: 15px 0 8px; }
        .btn-row { display: flex; gap: 8px; width: 100%; justify-content: center; }
        
        button {
            flex: 1;
            padding: 15px 5px;
            border: none;
            border-radius: 16px;
            background: rgba(255, 255, 255, 0.15);
            color: white;
            font-weight: bold;
            font-size: 13px;
            cursor: pointer;
            border: 1px solid rgba(255, 255, 255, 0.1);
            transition: 0.2s active;
        }
        button:active { background: #3a56d4; transform: scale(0.92); }

        #status-text { margin: 10px 0; font-size: 1.3rem; font-weight: bold; color: white; text-shadow: 2px 2px 5px rgba(0,0,0,0.4); }
    </style>
</head>
<body class="home">

<div id="pet-stage">
    <div class="stats-panel">
        <div>
            <span class="stat-label">🍖 КОРМ</span>
            <div class="bar-bg"><div id="h-bar" class="fill" style="background: #ff5e62;"></div></div>
        </div>
        <div>
            <span class="stat-label">🌈 СЧАСТЬЕ</span>
            <div class="bar-bg"><div id="ha-bar" class="fill" style="background: #a8e063;"></div></div>
        </div>
    </div>

    <div id="stitch-container" 
         onmousedown="startPurr()" onmouseup="stopPurr()" onmouseleave="stopPurr()"
         ontouchstart="startPurr()" ontouchend="stopPurr()">
        <div id="stitch-face-anim">
            <div class="ear left"></div><div class="ear right"></div>
            <div class="head">
                <div class="eye-patch left"><div class="eye"></div></div>
                <div class="eye-patch right"><div class="eye"></div></div>
                <div class="nose"></div>
                <div id="mouth" class="mouth"></div>
            </div>
        </div>
    </div>

    <div id="status-text">Алоха! ✨</div>

    <div class="controls">
        <div class="label">Выбрать мир</div>
        <div class="btn-row">
            <button onclick="setWorld('home')">🏠 ДОМ</button>
            <button onclick="setWorld('beach')">🏖️ ПЛЯЖ</button>
            <button onclick="setWorld('space')">🚀 КОСМОС</button>
        </div>

        <div class="label">Забота</div>
        <div class="btn-row">
            <button onclick="doAction('feed')">🥪 ЕДА</button>
            <button onclick="doAction('play')">🎸 ГИТАРА</button>
            <button onclick="doAction('hug')">🫂 ОБНЯТЬ</button>
            <button onclick="doAction('sleep')">💤 СОН</button>
        </div>
    </div>
</div>

<script>
    let stats = { hunger: 85, happy: 70 };
    let audioCtx, purrOsc;

    // Голос Стича
    function speak(text, pitch = 1.8) {
        const utterance = new SpeechSynthesisUtterance(text);
        utterance.lang = 'ru-RU';
        utterance.pitch = pitch;
        utterance.rate = 1.1;
        window.speechSynthesis.speak(utterance);
    }

    // Моргает сам по себе
    setInterval(() => {
        const face = document.getElementById('stitch-face-anim');
        face.classList.add('is-blinking');
        setTimeout(() => face.classList.remove('is-blinking'), 150);
    }, 3500);

    function setWorld(room) {
        document.body.className = room;
        const status = document.getElementById('status-text');
        if(room === 'space') {
            status.innerText = "Стич летит домой! 🚀";
            speak("Моя лететь домой!", 2.1);
        } else if(room === 'beach') {
            status.innerText = "Пляж! Серфинг! 🏄‍♂️";
            speak("Абакаба! Пляж!", 2);
        } else {
            status.innerText = "Дома уютно! 🏠";
            speak("Алоха!", 1.8);
        }
    }

    function doAction(type) {
        const status = document.getElementById('status-text');
        const face = document.getElementById('stitch-face-anim');
        
        if (type === 'feed') {
            stats.hunger = Math.min(100, stats.hunger + 20);
            face.classList.add('is-eating');
            speak("Ммм! Вкусно!", 2);
            status.innerText = "Ням-ням! 🥪";
            setTimeout(() => face.classList.remove('is-eating'), 1000);
        } else if (type === 'play') {
            stats.happy = Math.min(100, stats.happy + 25);
            status.innerText = "Уиии! Танцуем! 🎸";
            speak("Уииии! Абакаба!", 2.2);
        } else if (type === 'hug') {
            stats.happy = Math.min(100, stats.happy + 15);
            status.innerText = "Охана — это семья 🫂";
            speak("Охана значит семья", 0.7);
        } else if (type === 'sleep') {
            status.innerText = "Стич видит сны... 💤";
            speak("Хрррр псссс", 0.5);
        }
        updateBars();
    }

    function startPurr() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        purrOsc = audioCtx.createOscillator();
        const gainNode = audioCtx.createGain();
        
        purrOsc.type = 'sine';
        purrOsc.frequency.setValueAtTime(60, audioCtx.currentTime);
        gainNode.gain.setValueAtTime(0.04, audioCtx.currentTime);
        
        purrOsc.connect(gainNode);
        gainNode.connect(audioCtx.destination);
        purrOsc.start();
        
        document.getElementById('stitch-container').classList.add('purring-active');
        document.getElementById('status-text').innerText = "Мррррр... ❤️";
        stats.happy = Math.min(100, stats.happy + 0.3);
        updateBars();
    }

    function stopPurr() {
        if (purrOsc) {
            purrOsc.stop();
            purrOsc.disconnect();
        }
        document.getElementById('stitch-container').classList.remove('purring-active');
    }

    function updateBars() {
        document.getElementById('
