<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch: Ultimate Edition</title>
    <style>
        :root {
            --stitch-main: #3a56d4;
            --stitch-light: #5d8aff;
            --stitch-dark: #1b2661;
            --accent: #ff477e;
        }

        body {
            margin: 0; padding: 0; display: flex; justify-content: center; align-items: center;
            height: 100vh; background: #000; font-family: 'SF Pro Display', sans-serif; overflow: hidden;
        }

        #app {
            width: 100%; max-width: 450px; height: 100vh;
            position: relative; display: flex; flex-direction: column;
            transition: background 1s cubic-bezier(0.4, 0, 0.2, 1);
        }

        /* ТЕМЫ КОМНАТ */
        .kitchen { background: radial-gradient(circle, #ff9a9e, #fad0c4); }
        .playroom { background: radial-gradient(circle, #a1c4fd, #c2e9fb); }
        .bathroom { background: radial-gradient(circle, #84fab0, #8fd3f4); }
        .bedroom { background: radial-gradient(circle, #302b63, #0f0c29); }

        /* ВЕРХНИЙ ИНТЕРФЕЙС */
        .header {
            padding: 30px 20px; display: flex; align-items: center; gap: 15px;
            background: rgba(255,255,255,0.15); backdrop-filter: blur(20px);
            border-bottom: 1px solid rgba(255,255,255,0.2); z-index: 10;
        }
        .lvl {
            width: 55px; height: 55px; background: var(--accent);
            border-radius: 18px; border: 3px solid white;
            display: flex; justify-content: center; align-items: center;
            color: white; font-weight: 900; font-size: 24px;
            box-shadow: 0 8px 15px rgba(255, 71, 126, 0.4);
        }

        /* ГЕЙМПЛЕЙНАЯ ЗОНА */
        #stage {
            flex-grow: 1; display: flex; justify-content: center; align-items: center; position: relative;
        }

        /* ПРЕДМЕТЫ */
        .item {
            position: absolute; font-size: 60px; cursor: pointer;
            filter: drop-shadow(0 10px 15px rgba(0,0,0,0.2));
            transition: 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            z-index: 6;
        }
        .item:hover { transform: scale(1.2) rotate(15deg); }

        /* ТО САМОЕ ЛИЦО СТИЧА */
        .stitch {
            width: 200px; height: 180px; position: relative;
            cursor: pointer; animation: idle 3s ease-in-out infinite;
        }
        @keyframes idle {
            0%, 100% { transform: translateY(0) scale(1); }
            50% { transform: translateY(-5px) scale(1.02); }
        }

        .head {
            width: 100%; height: 100%; background: var(--stitch-main);
            border-radius: 50% 50% 45% 45%; position: relative;
            box-shadow: inset -8px -12px 25px rgba(0,0,0,0.4), 0 15px 35px rgba(0,0,0,0.3);
            border: 3px solid var(--stitch-dark); overflow: hidden;
        }

        .ear {
            position: absolute; top: -20px; width: 60px; height: 130px;
            background: var(--stitch-main); border-radius: 100% 20%;
            border: 3px solid var(--stitch-dark); z-index: -1;
        }
        .ear.l { left: -35px; transform: rotate(-35deg); }
        .ear.r { right: -35px; transform: rotate(35deg) scaleX(-1); }

        /* Глаза как в старых версиях */
        .patch {
            position: absolute; top: 25px; width: 70px; height: 95px;
            background: rgba(173, 216, 230, 0.25); border-radius: 50%;
        }
        .patch.l { left: 15px; transform: rotate(15deg); }
        .patch.r { right: 15px; transform: rotate(-15deg); }

        .eye {
            width: 48px; height: 65px; background: #0a0a0a;
            border-radius: 50%; position: absolute; top: 15px; left: 11px;
        }
        .eye::before {
            content: ''; position: absolute; top: 10px; left: 10px;
            width: 14px; height: 22px; background: white; border-radius: 50%;
        }

        .nose {
            width: 45px; height: 26px; background: var(--stitch-dark);
            position: absolute; top: 105px; left: 50%;
            transform: translateX(-50%); border-radius: 50% 50% 40% 40%;
        }

        /* НИЖНЯЯ ПАНЕЛЬ */
        .navbar {
            height: 110px; display: grid; grid-template-columns: repeat(4, 1fr);
            gap: 15px; padding: 15px 25px;
            background: rgba(255,255,255,0.25); backdrop-filter: blur(25px);
            border-radius: 45px 45px 0 0; border-top: 1px solid rgba(255,255,255,0.3);
        }
        .nav-btn {
            background: white; border: none; border-radius: 25px;
            font-size: 32px; cursor: pointer; transition: 0.2s;
            box-shadow: 0 6px 0 #ccc; display: flex; justify-content: center; align-items: center;
        }
        .nav-btn.active { background: var(--accent); color: white; transform: translateY(4px); box-shadow: none; }

    </style>
</head>
<body>

<div id="app" class="kitchen">
    <div class="header">
        <div class="lvl" id="lvl-counter">1</div>
        <div style="flex-grow: 1;">
            <div style="font-size: 11px; font-weight: 800; color: rgba(0,0,0,0.5); letter-spacing: 1px;">ENERGY</div>
            <div style="width: 100%; height: 14px; background: rgba(0,0,0,0.1); border-radius: 7px; overflow: hidden; margin-top: 5px;">
                <div id="energy-bar" style="width: 85%; height: 100%; background: #47ffb0; box-shadow: 0 0 10px #47ffb0;"></div>
            </div>
        </div>
    </div>

    <div id="stage">
        <div id="prop-k" class="item" style="top:60%; left:10%;" onclick="use('🥪')">🍱</div>
        <div id="prop-p" class="item" style="top:60%; right:10%; display:none;" onclick="use('⚽')">🎡</div>
        <div id="prop-b" class="item" style="top:65%; left:15%; display:none;" onclick="use('🫧')">🧼</div>
        <div id="prop-s" class="item" style="top:55%; right:15%; display:none;" onclick="use('✨')">🌙</div>

        <div class="stitch" onclick="jump()">
            <div class="ear l"></div><div class="ear r"></div>
            <div class="head">
                <div class="patch l"><div class="eye"></div></div>
                <div class="patch r"><div class="eye"></div></div>
                <div class="nose"></div>
                <div style="width: 40px; height: 4px; background: var(--stitch-dark); position: absolute; bottom: 35px; left: 50%; transform: translateX(-50%); border-radius: 2px; opacity: 0.6;"></div>
            </div>
        </div>
    </div>

    <div class="navbar">
        <button class="nav-btn active" onclick="go('kitchen', this)">🍴</button>
        <button class="nav-btn" onclick="go('playroom', this)">🧸</button>
        <button class="nav-btn" onclick="go('bathroom', this)">🧼</button>
        <button class="nav-btn" onclick="go('bedroom', this)">🛌</button>
    </div>
</div>

<script>
    const app = document.getElementById('app');
    let level = 1;
    let audio;

    function go(room, btn) {
        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        app.className = room;

        // Показ предметов
        document.querySelectorAll('.item').forEach(i => i.style.display = 'none');
        if(room === 'kitchen') { document.getElementById('prop-k').style.display = 'block'; say("Стич хочет сэндвич!", 1.8); }
        if(room === 'playroom') { document.getElementById('prop-p').style.display = 'block'; say("Абакаба! Поиграем?", 2); }
        if(room === 'bathroom') { document.getElementById('prop-b').style.display = 'block'; say("Стич моет ушки", 1.6); }
        if(room === 'bedroom') { document.getElementById('prop-s').style.display = 'block'; say("Хррр-псссс", 0.8); }
        
        beep(300, 0.1);
    }

    function use(emoji) {
        jump();
        level++;
        document.getElementById('lvl-counter').innerText = level;
        say("Оооо, " + emoji + "! Круто!", 2.2);
        beep(600, 0.1); beep(900, 0.1);
    }

    function jump() {
        const s = document.querySelector('.stitch');
        s.style.transition = '0.1s';
        s.style.transform = 'scale(1.1) translateY(-30px)';
        setTimeout(() => { 
            s.style.transition = '0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275)';
            s.style.transform = 'scale(1) translateY(0)'; 
        }, 150);
    }

    function say(txt, p) {
        const m = new SpeechSynthesisUtterance(txt);
        m.lang = 'ru-RU'; m.pitch = p; m.rate = 1.3;
        window.speechSynthesis.speak(m);
    }

    function beep(f, d) {
        if (!audio) audio = new (window.AudioContext || window.webkitAudioContext)();
        let o = audio.createOscillator();
        let g = audio.createGain();
        o.frequency.setValueAtTime(f, audio.currentTime);
        g.gain.setValueAtTime(0.1, audio.currentTime);
        g.gain.exponentialRampToValueAtTime(0.01, audio.currentTime + d);
        o.connect(g); g.connect(audio.destination);
        o.start(); o.stop(audio.currentTime + d);
    }
</script>

</body>
</html>
