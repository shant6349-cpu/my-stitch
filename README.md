<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch: Real Tamagotchi</title>
    <style>
        :root {
            --stitch-blue: #3a56d4;
            --stitch-dark: #1b2661;
            --ui-pink: #ff477e;
            --gold: #ffd700;
            --bar-bg: rgba(255, 255, 255, 0.3);
        }

        body {
            margin: 0; padding: 0; display: flex; justify-content: center; align-items: center;
            height: 100vh; background: #050505; font-family: 'Arial Rounded MT Bold', sans-serif; overflow: hidden;
        }

        #game-window {
            width: 100%; max-width: 450px; height: 100vh;
            position: relative; display: flex; flex-direction: column;
            transition: 0.5s ease;
        }

        /* КОМНАТЫ */
        .kitchen { background: radial-gradient(#ff9a9e, #fad0c4); }
        .playroom { background: radial-gradient(#a1c4fd, #c2e9fb); }
        .bathroom { background: radial-gradient(#84fab0, #8fd3f4); }
        .bedroom { background: radial-gradient(#302b63, #0f0c29); }

        /* ИНТЕРФЕЙС ПАНЕЛИ СТАТУСА */
        .hud {
            padding: 15px; background: rgba(0, 0, 0, 0.2); backdrop-filter: blur(20px);
            display: grid; grid-template-columns: 1fr 2fr 1fr; gap: 10px; z-index: 100;
        }
        
        .stats-col { display: flex; flex-direction: column; gap: 5px; }
        
        .bar-container { width: 100%; height: 8px; background: var(--bar-bg); border-radius: 4px; overflow: hidden; position: relative; }
        .bar-fill { height: 100%; transition: width 0.3s ease, background 0.3s; }
        
        .label { font-size: 9px; color: white; font-weight: bold; text-transform: uppercase; margin-bottom: 2px; display: block; }
        
        .stat-card { background: white; border-radius: 12px; padding: 5px; text-align: center; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }

        /* СТИЧ */
        #stage { flex-grow: 1; display: flex; justify-content: center; align-items: center; position: relative; }
        .stitch-container { width: 200px; height: 180px; position: relative; cursor: pointer; transition: 0.2s; animation: breathe 3s ease-in-out infinite; }
        @keyframes breathe { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.03); } }

        .head { width: 100%; height: 100%; background: var(--stitch-blue); border-radius: 50% 50% 45% 45%; border: 4px solid var(--stitch-dark); box-shadow: inset -8px -12px 30px rgba(0,0,0,0.4); }
        .ear { position: absolute; top: -25px; width: 55px; height: 130px; background: var(--stitch-blue); border: 4px solid var(--stitch-dark); border-radius: 100% 20%; z-index: -1; }
        .ear.l { left: -35px; transform: rotate(-35deg); }
        .ear.r { right: -35px; transform: rotate(35deg) scaleX(-1); }
        .patch { position: absolute; top: 25px; width: 75px; height: 100px; background: rgba(173, 216, 230, 0.3); border-radius: 50%; }
        .patch.l { left: 15px; transform: rotate(15deg); }
        .patch.r { right: 15px; transform: rotate(-15deg); }
        .eye { width: 50px; height: 70px; background: #000; border-radius: 50%; position: absolute; top: 15px; left: 12px; }
        .eye::after { content: ''; position: absolute; top: 10px; left: 10px; width: 15px; height: 25px; background: white; border-radius: 50%; }
        .nose { width: 45px; height: 25px; background: var(--stitch-dark); position: absolute; top: 105px; left: 50%; transform: translateX(-50%); border-radius: 50% 50% 40% 40%; }

        /* ПРЕДМЕТЫ */
        .toy { position: absolute; font-size: 60px; cursor: pointer; filter: drop-shadow(0 8px 10px rgba(0,0,0,0.2)); z-index: 10; transition: 0.2s; }
        .toy:active { transform: scale(1.2); }

        /* НИЖНЯЯ ПАНЕЛЬ */
        .nav { height: 100px; display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; padding: 10px 20px; background: rgba(255,255,255,0.2); backdrop-filter: blur(30px); border-radius: 35px 35px 0 0; }
        .btn { background: white; border: none; border-radius: 20px; font-size: 32px; cursor: pointer; transition: 0.2s; box-shadow: 0 5px 0 #cbd5e0; }
        .btn.active { background: var(--ui-pink); transform: translateY(4px); box-shadow: none; color: white; }

        /* ЭФФЕКТЫ */
        .particle { position: absolute; pointer-events: none; animation: flyUp 1s forwards; font-size: 24px; z-index: 1000; }
        @keyframes flyUp { to { transform: translateY(-100px) opacity: 0; } }
    </style>
</head>
<body>

<div id="game-window" class="kitchen">
    <div class="hud">
        <div class="stat-card">
            <span style="font-size: 10px; font-weight: bold;">LVL</span>
            <div id="lvl" style="font-size: 18px; font-weight: 900; color: var(--ui-pink);">1</div>
        </div>
        
        <div class="stats-col">
            <div>
                <span class="label">🍖 Голод</span>
                <div class="bar-container"><div id="hunger-bar" class="bar-fill" style="width: 80%; background: #ff4b2b;"></div></div>
            </div>
            <div>
                <span class="label">🧼 Чистота</span>
                <div class="bar-container"><div id="clean-bar" class="bar-fill" style="width: 100%; background: #4facfe;"></div></div>
            </div>
            <div>
                <span class="label">⚡ Энергия</span>
                <div class="bar-container"><div id="energy-bar" class="bar-fill" style="width: 100%; background: #f9d423;"></div></div>
            </div>
        </div>

        <div class="stat-card">
            <span style="font-size: 18px;">💰</span>
            <div id="coins" style="font-size: 14px; font-weight: bold; color: #b8860b;">100</div>
        </div>
    </div>

    <div id="stage" onclick="createHeart(event)">
        <div id="t-k" class="toy" style="bottom:20%; left:10%;" onclick="action('eat')">🍱</div>
        <div id="t-p" class="toy" style="bottom:15%; right:10%; display:none;" onclick="action('play')">🏀</div>
        <div id="t-b" class="toy" style="bottom:25%; left:20%; display:none;" onclick="action('wash')">🧼</div>
        <div id="t-s" class="toy" style="bottom:15%; right:20%; display:none;" onclick="action('sleep')">🌙</div>

        <div class="stitch-container" id="stitch" onclick="dance()">
            <div class="ear l"></div><div class="ear r"></div>
            <div class="head">
                <div class="patch l"><div class="eye"></div></div>
                <div class="patch r"><div class="eye"></div></div>
                <div class="nose"></div>
            </div>
        </div>
    </div>

    <div class="nav">
        <button class="btn active" onclick="nav('kitchen', this)">🍴</button>
        <button class="btn" onclick="nav('playroom', this)">🧸</button>
        <button class="btn" onclick="nav('bathroom', this)">🧼</button>
        <button class="btn" onclick="nav('bedroom', this)">🛌</button>
    </div>
</div>

<script>
    let state = {
        lvl: 1, exp: 0, coins: 100,
        hunger: 80, clean: 100, energy: 100
    };
    let audio;

    function talk(txt, p) {
        window.speechSynthesis.cancel(); 
        const m = new SpeechSynthesisUtterance(txt);
        m.lang = 'ru-RU'; m.pitch = p; m.rate = 1.4;
        window.speechSynthesis.speak(m);
    }

    function nav(room, btn) {
        window.speechSynthesis.cancel();
        document.querySelectorAll('.btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        document.getElementById('game-window').className = room;
        document.querySelectorAll('.toy').forEach(t => t.style.display = 'none');
        
        if(room === 'kitchen') { document.getElementById('t-k').style.display = 'block'; talk("Кушать!", 1.8); }
        if(room === 'playroom') { document.getElementById('t-p').style.display = 'block'; talk("Уиии!", 2.2); }
        if(room === 'bathroom') { document.getElementById('t-b').style.display = 'block'; talk("Вода!", 1.5); }
        if(room === 'bedroom') { document.getElementById('t-s').style.display = 'block'; talk("Хррр", 0.8); }
        beep(200, 0.1);
    }

    function action(type) {
        window.speechSynthesis.cancel();
        dance();

        if(type === 'eat') {
            if(state.coins >= 10) {
                state.coins -= 10; state.hunger = Math.min(100, state.hunger + 30);
                state.exp += 20; talk("Вкусно!", 2);
            } else { talk("Нет денег!", 1.5); }
        }
        if(type === 'play') {
            if(state.energy >= 15) {
                state.energy -= 15; state.coins += 20; state.hunger -= 10;
                state.exp += 25; talk("Абакаба!", 2.5);
            } else { talk("Устал...", 1.2); }
        }
        if(type === 'wash') {
            state.clean = 100; state.exp += 15; talk("Чисто!", 1.7);
        }
        if(type === 'sleep') {
            state.energy = 100; state.exp += 10; talk("Сплю", 0.8);
        }

        if(state.exp >= 100) { state.lvl++; state.exp = 0; talk("Левел ап!", 2.1); }
        updateUI();
        beep(500, 0.1);
    }

    function updateUI() {
        document.getElementById('lvl').innerText = state.lvl;
        document.getElementById('coins').innerText = state.coins;
        document.getElementById('hunger-bar').style.width = state.hunger + '%';
        document.getElementById('clean-bar').style.width = state.clean + '%';
        document.getElementById('energy-bar').style.width = state.energy + '%';
    }

    // ТАЙМЕР: Показатели падают сами со временем
    setInterval(() => {
        state.hunger = Math.max(0, state.hunger - 1);
        state.clean = Math.max(0, state.clean - 0.5);
        state.energy = Math.max(0, state.energy - 0.3);
        updateUI();
        
        if(state.hunger < 20) { document.getElementById('stitch').style.filter = 'grayscale(0.5)'; }
        else { document.getElementById('stitch').style.filter = 'none'; }
    }, 3000);

    function dance() {
        const s = document.getElementById('stitch');
        s.style.transform = 'translateY(-40px) scale(1.1)';
        setTimeout(() => s.style.transform = 'translateY(0) scale(1)', 150);
    }

    function createHeart(e) {
        const h = document.createElement('div');
        h.className = 'particle'; h.innerText = '❤️';
        h.style.left = e.clientX + 'px'; h.style.top = e.clientY + 'px';
        document.body.appendChild(h);
        setTimeout(() => h.remove(), 1000);
    }

    function beep(f, d) {
        if (!audio) audio = new (window.AudioContext || window.webkitAudioContext)();
        let o = audio.createOscillator(), g = audio.createGain();
        o.frequency.setValueAtTime(f, audio.currentTime);
        g.gain.setValueAtTime(0.05, audio.currentTime);
        o.connect(g); g.connect(audio.destination);
        o.start(); o.stop(audio.currentTime + d);
    }
</script>
</body>
</html>
