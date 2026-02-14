<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch: Magic Edition</title>
    <style>
        body {
            margin: 0; display: flex; justify-content: center; align-items: center; height: 100vh;
            background: linear-gradient(-45deg, #0f0c29, #302b63, #24243e, #1a2a6c, #b21f1f);
            background-size: 400% 400%; animation: gradientBG 15s ease infinite;
            font-family: 'Segoe UI', sans-serif; overflow: hidden; transition: 2s;
        }

        body.night-mode { background: #000428 !important; }

        @keyframes gradientBG {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        #pet-stage {
            width: 350px; padding: 25px; background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px);
            border-radius: 40px; text-align: center; border: 1px solid rgba(255, 255, 255, 0.2);
            box-shadow: 0 25px 80px rgba(0,0,0,0.5); color: white; position: relative; z-index: 10;
        }

        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px; }
        .stat-label { font-size: 10px; font-weight: bold; color: rgba(255,255,255,0.7); display: block; }
        .bar-bg { width: 100%; height: 8px; background: rgba(0,0,0,0.3); border-radius: 5px; overflow: hidden; margin-top: 4px; }
        .fill { height: 100%; width: 100%; transition: width 0.5s ease; }

        /* ЛИЦО СТИЧА */
        #stitch-container { position: relative; width: 180px; height: 160px; margin: 30px auto; cursor: pointer; touch-action: none; }
        
        /* Эффект мурчания */
        @keyframes purr {
            0% { transform: scale(1); }
            50% { transform: scale(1.03) rotate(1deg); }
            100% { transform: scale(1); }
        }
        .purring { animation: purr 0.15s infinite; filter: drop-shadow(0 0 15px #ff69b4); }

        .head {
            position: absolute; width: 100%; height: 100%;
            background: radial-gradient(circle at 35% 30%, #5d8aff, #3a56d4 60%, #1e2a78 100%);
            border-radius: 50% 50% 46% 46%; z-index: 5;
            box-shadow: inset -5px -10px 20px rgba(0,0,0,0.6), 0 10px 20px rgba(0,0,0,0.3);
        }

        .ear { position: absolute; top: -20px; width: 55px; height: 120px; background: #3a56d4; border-radius: 100% 20% 100% 20%; z-index: 1; transition: 0.6s; }
        .ear.left { left: -35px; transform: rotate(-35deg); }
        .ear.right { right: -35px; transform: rotate(35deg) scaleX(-1); }

        .eye-patch { position: absolute; top: 25px; width: 65px; height: 85px; background: rgba(173, 216, 230, 0.2); border-radius: 50%; z-index: 6; }
        .eye-patch.left { left: 15px; transform: rotate(12deg); }
        .eye-patch.right { right: 15px; transform: rotate(-12deg); }

        .eye { position: absolute; top: 18px; left: 10px; width: 44px; height: 56px; background: #080808; border-radius: 50%; transition: 0.3s; }
        .eye::before { content: ''; position: absolute; top: 8px; left: 8px; width: 12px; height: 20px; background: white; border-radius: 50%; opacity: 0.9; }

        .nose { position: absolute; top: 95px; left: 50%; transform: translateX(-50%); width: 44px; height: 24px; background: #1b2661; border-radius: 50% 50% 40% 40%; z-index: 7; }
        .mouth { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); width: 45px; height: 6px; border-bottom: 3px solid #1b2661; border-radius: 50%; z-index: 7; transition: 0.3s; }

        .sleeping .eye { height: 4px; top: 35px; }
        .eating .mouth { height: 18px; background: #000; border-radius: 50%; }

        .btns { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 15px; }
        button {
            padding: 12px; border: none; border-radius: 15px; cursor: pointer;
            background: rgba(255, 255, 255, 0.15); color: white; font-weight: bold; 
            border: 1px solid rgba(255,255,255,0.1); backdrop-filter: blur(5px);
        }
        button:active { transform: scale(0.95); background: #3a56d4; }
    </style>
</head>
<body>

<div id="pet-stage">
    <div class="stats-grid">
        <div class="stat-box"><span class="stat-label">🍖 ГОЛОД</span><div class="bar-bg"><div id="h-fill" class="fill" style="background: #ff5e62;"></div></div></div>
        <div class="stat-box"><span class="stat-label">😊 СЧАСТЬЕ</span><div class="bar-bg"><div id="ha-fill" class="fill" style="background: #a8e063;"></div></div></div>
        <div class="stat-box"><span class="stat-label">⚡ ЭНЕРГИЯ</span><div class="bar-bg"><div id="e-fill" class="fill" style="background: #00c6ff;"></div></div></div>
        <div class="stat-box"><span class="stat-label">🌈 ДУША</span><div class="bar-bg"><div id="m-fill" class="fill" style="background: #f9d423;"></div></div></div>
    </div>

    <div id="stitch-container" 
         onmousedown="startPurr()" onmouseup="stopPurr()" 
         ontouchstart="startPurr()" ontouchend="stopPurr()">
        <div id="stitch-face">
            <div class="ear left"></div><div class="ear right"></div>
            <div class="head">
                <div class="eye-patch left"><div class="eye"></div></div>
                <div class="eye-patch right"><div class="eye"></div></div>
                <div class="nose"></div><div id="mouth" class="mouth"></div>
            </div>
        </div>
    </div>

    <h3 id="status">Алоха! ✨</h3>

    <div class="btns">
        <button onclick="update('feed')">🥪 КОРМИТЬ</button>
        <button onclick="update('play')">🏄‍♂️ ИГРАТЬ</button>
        <button onclick="update('hug')">🫂 ОБНЯТЬ</button>
        <button onclick="update('sleep')">💤 СПАТЬ</button>
    </div>
</div>

<script>
    let stats = { hunger: 80, happy: 80, energy: 80, mood: 80 };
    let isSleeping = false;
    let audioCtx, oscillator;

    // Функция речи
    function say(text, pitch = 1.5, rate = 1) {
        const msg = new SpeechSynthesisUtterance(text);
        msg.lang = 'ru-RU';
        msg.pitch = pitch;
        msg.rate = rate;
        window.speechSynthesis.speak(msg);
    }

    // Мурчание
    function startPurr() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        oscillator = audioCtx.createOscillator();
        let gain = audioCtx.createGain();
        oscillator.type = 'sine';
        oscillator.frequency.setValueAtTime(55, audioCtx.currentTime); 
        gain.gain.setValueAtTime(0.05, audioCtx.currentTime);
        oscillator.connect(gain); gain.connect(audioCtx.destination);
        oscillator.start();
        document.getElementById('stitch-container').classList.add('purring');
        document.getElementById('status').innerText = "Мрррр... ❤️";
    }

    function stopPurr() {
        if (oscillator) { oscillator.stop(); oscillator.disconnect(); }
        document.getElementById('stitch-container').classList.remove('purring');
    }

    function update(action) {
        const face = document.getElementById('stitch-face');
        const msg = document.getElementById('status');
        if (isSleeping && action !== 'sleep') return;

        switch(action) {
            case 'feed':
                stats.hunger = Math.min(100, stats.hunger + 25);
                face.classList.add('eating');
                say("Вкусно!", 1.8, 1.2);
                setTimeout(() => face.classList.remove('eating'), 1000);
                msg.innerText = "Ням-ням! 🥪";
                break;
            case 'play':
                if (stats.energy < 20) return;
                stats.happy = Math.min(100, stats.happy + 20);
                stats.energy -= 25;
                say("Уиии!", 2, 1.5);
                msg.innerText = "Серфинг! 🏄‍♂️";
                break;
            case 'hug':
                stats.mood = Math.min(100, stats.mood + 30);
                say("Охана значит семья", 0.9, 0.8);
                msg.innerText = "Охана! 🫂";
                break;
            case 'sleep':
                isSleeping = !isSleeping;
                document.body.classList.toggle('night-mode');
                face.classList.toggle('sleeping');
                msg.innerText = isSleeping ? "Хррр-пссс... 💤" : "Алоха! ✨";
                break;
        }
        refreshUI();
    }

    function refreshUI() {
        document.getElementById('h-fill').style.width = stats.hunger + '%';
        document.getElementById('ha-fill').style.width = stats.happy + '%';
        document.getElementById('e-fill').style.width = stats.energy + '%';
        document.getElementById('m-fill').style.width = stats.mood + '%';
    }

    // Таймер голода и просьбы поесть
    setInterval(() => {
        if (!isSleeping) {
            stats.hunger = Math.max(0, stats.hunger - 2);
            if (stats.hunger < 30 && stats.hunger > 0) {
                say("Дай покушать", 1.5, 1);
                document.getElementById('status').innerText = "Я голоден! 😭";
            }
        } else {
            stats.energy = Math.min(100, stats.energy + 5);
        }
        refreshUI();
    }, 5000);
</script>
</body>
</html>
