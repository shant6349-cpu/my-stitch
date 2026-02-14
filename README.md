<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch: Classic Interface</title>
    <style>
        body {
            margin: 0; display: flex; justify-content: center; align-items: center; height: 100vh;
            background: #a8e063; font-family: 'Segoe UI', sans-serif; overflow: hidden; transition: 1s;
        }
        body.home { background: #ff9966; }
        body.beach { background: #4ca1af; }
        body.space { background: #24243e; }

        #pet-stage {
            width: 350px; height: 650px; background: rgba(255, 255, 255, 0.2);
            backdrop-filter: blur(15px); border-radius: 40px; text-align: center;
            border: 2px solid white; box-shadow: 0 20px 50px rgba(0,0,0,0.2);
            display: flex; flex-direction: column; justify-content: space-between; padding: 20px;
        }

        /* ВЕРХНЯЯ ПАНЕЛЬ (КАК НА ВИДЕО) */
        .top-nav { display: flex; justify-content: space-between; align-items: center; }
        .lvl-box { background: white; border-radius: 15px; padding: 10px; width: 60px; }
        .stats-bars { flex-grow: 1; margin: 0 10px; }
        .coin-box { background: white; border-radius: 15px; padding: 10px; width: 60px; }
        
        .bar-container { width: 100%; height: 10px; background: rgba(0,0,0,0.1); border-radius: 5px; margin: 5px 0; overflow: hidden; }
        .fill { height: 100%; transition: 0.5s; }

        /* СТИЧ */
        #stitch-container { position: relative; width: 200px; height: 180px; margin: auto; cursor: pointer; }
        .head {
            width: 100%; height: 100%; background: #3a56d4; border-radius: 50% 50% 45% 45%;
            position: relative; z-index: 5; box-shadow: inset -5px -10px 15px rgba(0,0,0,0.3);
        }
        .ear { position: absolute; top: -15px; width: 60px; height: 130px; background: #3a56d4; border-radius: 100% 20%; z-index: 1; }
        .ear.left { left: -40px; transform: rotate(-30deg); }
        .ear.right { right: -40px; transform: rotate(30deg) scaleX(-1); }
        .eye { width: 45px; height: 60px; background: #000; border-radius: 50%; position: absolute; top: 30px; }
        .eye.left { left: 35px; } .eye.right { right: 35px; }
        .eye::after { content: ''; position: absolute; top: 10px; left: 10px; width: 12px; height: 20px; background: #fff; border-radius: 50%; }
        .nose { width: 40px; height: 20px; background: #1b2661; position: absolute; top: 110px; left: 50%; transform: translateX(-50%); border-radius: 50%; z-index: 6; }
        .mouth { width: 40px; height: 5px; border-bottom: 3px solid #1b2661; position: absolute; bottom: 35px; left: 50%; transform: translateX(-50%); border-radius: 50%; z-index: 6; }

        /* НИЖНИЕ КНОПКИ-КАРТОЧКИ */
        .bottom-menu { display: flex; justify-content: space-around; background: white; border-radius: 30px; padding: 10px; }
        .menu-btn { width: 60px; height: 70px; border-radius: 20px; display: flex; align-items: center; justify-content: center; font-size: 24px; cursor: pointer; transition: 0.2s; border: none; background: #f0f0f0; }
        .menu-btn:active { transform: scale(0.9); background: #ff4081; color: white; }

        .eating .mouth { height: 20px; background: black; border-radius: 50%; }
    </style>
</head>
<body class="home">

<div id="pet-stage">
    <div class="top-nav">
        <div class="lvl-box"><small>LVL</small><br><b style="color:#ff4081">2</b></div>
        <div class="stats-bars">
            <div class="bar-container"><div id="h-fill" class="fill" style="background:#ff5e62; width:80%"></div></div>
            <div class="bar-container"><div id="ha-fill" class="fill" style="background:#a8e063; width:80%"></div></div>
        </div>
        <div class="coin-box">💰<br><b>80</b></div>
    </div>

    <div id="stitch-container" onclick="purr()">
        <div id="stitch-face">
            <div class="ear left"></div><div class="ear right"></div>
            <div class="head">
                <div class="eye left"></div><div class="eye right"></div>
                <div class="nose"></div><div id="mouth" class="mouth"></div>
            </div>
        </div>
    </div>

    <h3 id="status" style="color: white; text-shadow: 1px 1px 3px rgba(0,0,0,0.3);">Алоха! ✨</h3>

    <div class="bottom-menu">
        <button class="menu-btn" onclick="act('feed')">🍴</button>
        <button class="menu-btn" onclick="act('play')">🧸</button>
        <button class="menu-btn" onclick="act('room')">🏠</button>
        <button class="menu-btn" onclick="act('sleep')">🛏️</button>
    </div>
</div>

<script>
    let rooms = ['home', 'beach', 'space'];
    let currentRoom = 0;

    // МЕДЛЕННЫЙ ГОЛОС
    function say(txt, p=1.6) {
        window.speechSynthesis.cancel();
        const m = new SpeechSynthesisUtterance(txt);
        m.lang = 'ru-RU'; m.pitch = p; m.rate = 0.75; // Скорость 0.75 - как просил
        window.speechSynthesis.speak(m);
    }

    function act(type) {
        const s = document.getElementById('status');
        const f = document.getElementById('stitch-face');
        if(type === 'feed') {
            f.classList.add('eating');
            s.innerText = "ЛЮБЛЮ мАРЯМА"; say("Ммм! Очень вкусно");
            setTimeout(() => f.classList.remove('eating'), 1000);
        } else if(type === 'play') {
            s.innerText = "Играем! 🧸"; say("Уиии! Абакаба", 2);
        } else if(type === 'room') {
            currentRoom = (currentRoom + 1) % rooms.length;
            document.body.className = rooms[currentRoom];
            s.innerText = "Меняем комнату!"; say("О! Другое место");
        } else if(type === 'sleep') {
            s.innerText = "Стич спит... 💤"; say("Хррр пссс", 0.5);
        }
    }

    function purr() {
        document.getElementById('status').innerText = "Мррр... ❤️";
        say("Охана значит семья", 0.8);
    }
</script>
</body>
</html>
