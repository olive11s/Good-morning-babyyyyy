# Good-morning-babyyyyy
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Precision Archer - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: none; }
        html, body { width: 100%; height: 100%; overflow: hidden; background: #000; position: fixed; font-family: -apple-system, system-ui, sans-serif; }

        #sky { position: fixed; inset: 0; background: linear-gradient(to bottom, #020111 0%, #000000 100%); transition: background 2s; z-index: -2; }
        #sun { position: absolute; bottom: -150px; left: 50%; transform: translateX(-50%); width: 140px; height: 140px; background: radial-gradient(circle, #fff700, #ff8c00); border-radius: 50%; box-shadow: 0 0 80px #ff8c00; z-index: -1; transition: bottom 2s; }
        
        #basket { 
            position: absolute; bottom: -20px; width: 120%; left: -10%; height: 25%; 
            background: #5d4037; border-top: 10px solid #3e2723; border-radius: 50% 50% 0 0; 
            z-index: 10; transition: transform 1.5s ease-in;
        }

        /* Aiming Line */
        #aim-line { position: absolute; bottom: 15%; left: 50%; width: 2px; height: 100px; background: rgba(255,255,255,0.3); transform-origin: bottom center; display: none; z-index: 15; }

        #bow-container { position: absolute; bottom: 5%; left: 50%; transform: translateX(-50%); width: 100px; height: 150px; z-index: 20; transition: transform 1.5s; }
        #bow-visual { font-size: 80px; text-align: center; pointer-events: none; transition: transform 0.1s; }

        .target { position: absolute; z-index: 5; transition: font-size 0.3s; }
        .projectile { position: absolute; width: 6px; height: 40px; background: gold; z-index: 15; border-radius: 3px; box-shadow: 0 0 10px orange; }

        #hud { position: absolute; top: 60px; width: 100%; display: flex; justify-content: space-around; z-index: 100; color: white; font-weight: bold; text-shadow: 0 0 10px #000; }
        .screen { position: fixed; inset: 0; z-index: 200; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; padding: 30px; }
        #start-screen { background: rgba(0,0,0,0.9); color: white; }
        #win-screen { display: none; background: rgba(255, 255, 255, 0.98); color: #333; }
        #fail-screen { display: none; background: rgba(0,0,0,0.95); color: white; }
        .btn { padding: 15px 35px; border-radius: 50px; border: none; background: #fbc02d; font-weight: bold; margin-top: 15px; color: #000; }
    </style>
</head>
<body>

<div id="sky"></div>
<div id="sun"></div>
<div id="basket"></div>
<div id="aim-line"></div>

<div id="bow-container">
    <div id="bow-visual">🏹</div>
</div>

<div id="start-screen" class="screen">
    <h1 style="color: #fbc02d;">Pro Archer Mode ❤️</h1>
    <p style="margin: 15px 0;">Drag & Aim to shoot!<br>The sunbeams get smaller as you rise.</p>
    <button class="btn" onclick="startGame()">Start 🎈</button>
</div>

<div id="hud">
    <div>Sunbeams: <span id="score">0</span>/10</div>
    <div>HP: <span id="lives-ui">❤️❤️❤️</span></div>
</div>

<div id="fail-screen" class="screen">
    <h1 style="color: #ff4d4d;">THE BALLOON FELL!</h1>
    <button class="btn" onclick="location.reload()">Try Again</button>
</div>

<div id="win-screen" class="screen">
    <h1 style="color: #f57c00;">Sunrise Secured! ☀️</h1>
    <p>Good morning, my love! Have a perfect day.</p>
    <div id="timerText" style="margin: 15px 0; font-weight: bold; color: #f57c00;"></div>
    <button class="btn" onclick="showLetter()" id="letterBtn">Open My Letter 💌</button>
    <div id="secretLetter" style="display:none; padding: 15px; font-style: italic;">"Giselle, you are beautiful and perfect. I love you!"</div>
</div>

<audio id="gameMusic" loop><source src="https://www.bensound.com/bensound-music/bensound-tomorrow.mp3" type="audio/mpeg"></audio>

<script>
    let score = 0, lives = 3, gameActive = false;
    let dragStartX, dragStartY, currentAngle = 0;
    
    const bowContainer = document.getElementById('bow-container');
    const bowVisual = document.getElementById('bow-visual');
    const aimLine = document.getElementById('aim-line');
    const basket = document.getElementById('basket');
    const sky = document.getElementById('sky');
    const sun = document.getElementById('sun');
    const music = document.getElementById('gameMusic');

    function startGame() {
        document.getElementById('start-screen').style.display = 'none';
        gameActive = true;
        music.play().catch(e => console.log("Audio play blocked"));
        updateCountdown();
        spawnLoop();
    }

    // Advanced Aiming Controls
    document.addEventListener('touchstart', (e) => {
        if (!gameActive) return;
        dragStartX = e.touches[0].clientX;
        dragStartY = e.touches[0].clientY;
        aimLine.style.display = 'block';
    });

    document.addEventListener('touchmove', (e) => {
        if (!gameActive) return;
        const touchX = e.touches[0].clientX;
        const touchY = e.touches[0].clientY;
        
        const diffX = touchX - dragStartX;
        const diffY = touchY - dragStartY;
        
        // Calculate angle based on drag
        currentAngle = Math.atan2(diffX, -diffY) * (180 / Math.PI);
        bowVisual.style.transform = `rotate(${currentAngle}deg)`;
        aimLine.style.transform = `translateX(-50%) rotate(${currentAngle}deg)`;
    });

    document.addEventListener('touchend', (e) => {
        if (!gameActive) return;
        aimLine.style.display = 'none';
        fireArrow(currentAngle);
    });

    function fireArrow(angle) {
        const proj = document.createElement('div');
        proj.className = 'projectile';
        proj.style.left = '50%';
        proj.style.bottom = '15%';
        proj.style.transform = `translateX(-50%) rotate(${angle}deg)`;
        document.body.appendChild(proj);

        let dist = 15;
        const radians = angle * (Math.PI / 180);
        const vx = Math.sin(radians) * 8;
        const vy = Math.cos(radians) * 8;
        let posX = 50; // percentage

        const fly = setInterval(() => {
            dist += vy;
            posX += (vx / window.innerWidth) * 100;
            proj.style.bottom = dist + '%';
            proj.style.left = posX + '%';
            
            checkCollision(proj, fly);
            if (dist > 110 || posX < -10 || posX > 110) { clearInterval(fly); proj.remove(); }
        }, 10);
    }

    function spawnItem() {
        if (!gameActive) return;
        const item = document.createElement('div');
        item.className = 'target';
        const rand = Math.random();
        let type = 'rain';
        if (rand > 0.8) type = 'sun';
        else if (rand > 0.6) type = 'cloud';

        item.innerHTML = (type === 'sun') ? '☀️' : (type === 'cloud' ? '☁️' : '💧');
        item.dataset.type = type;
        
        const sunSize = type === 'sun' ? Math.max(25, 50 - (score * 3)) : 45;
        item.style.fontSize = sunSize + 'px';
        
        let xPos = Math.random() * 80 + 10;
        item.style.left = xPos + '%';
        item.style.top = '-50px';
        document.body.appendChild(item);

        let yPos = -50;
        const fall = setInterval(() => {
            yPos += (type === 'sun' ? 3 : 6);
            item.style.top = yPos + 'px';
            if (yPos > window.innerHeight) { clearInterval(fall); item.remove(); }
        }, 20);
        item.dataset.intervalId = fall;
    }

    function checkCollision(proj, flyInterval) {
        const targets = document.querySelectorAll('.target');
        const pRect = proj.getBoundingClientRect();
        targets.forEach(t => {
            const tRect = t.getBoundingClientRect();
            if (pRect.left < tRect.right && pRect.right > tRect.left && pRect.top < tRect.bottom && pRect.bottom > tRect.top) {
                if (t.dataset.type === 'sun') {
                    score++;
                    updateEnv();
                } else {
                    lives--;
                    updateLivesUI();
                    if (lives <= 0) gameFail();
                }
                document.getElementById('score').innerText = score;
                clearInterval(t.dataset.intervalId);
                t.remove(); clearInterval(flyInterval); proj.remove();
                if (score >= 10) win();
            }
        });
    }

    function updateLivesUI() { document.getElementById('lives-ui').innerText = "❤️".repeat(Math.max(0, lives)); }

    function updateEnv() {
        const colors = ['#020111', '#1a1a40', '#4d2c5e', '#a34d5d', '#ff9a44', '#ffcf71'];
        sky.style.background = colors[Math.floor(score/2)] || '#ffcf71';
        sun.style.bottom = (-150 + (score * 35)) + 'px';
    }

    function gameFail() {
        gameActive = false;
        basket.style.transform = "translateY(600px) rotate(20deg)";
        bowContainer.style.transform = "translateY(600px)";
        setTimeout(() => { document.getElementById('fail-screen').style.display = 'flex'; }, 800);
    }

    function win() {
        gameActive = false;
        document.getElementById('win-screen').style.display = 'flex';
        document.querySelectorAll('.target').forEach(t => t.remove());
    }

    function showLetter() {
        document.getElementById('secretLetter').style.display = 'block';
        document.getElementById('letterBtn').style.display = 'none';
    }

    function updateCountdown() {
        const trip = new Date("February 14, 2026 00:00:00").getTime();
        const diff = trip - new Date().getTime();
        const d = Math.floor(diff / (1000*60*60*24));
        const h = Math.floor((diff % (1000*60*60*24)) / (1000*60*60));
        document.getElementById('timerText').innerText = `Seattle Trip: ${d}d ${h}h Left!`;
    }

    function spawnLoop() { 
        if(gameActive && score < 10) { 
            spawnItem(); 
            setTimeout(() => { if(gameActive) spawnItem(); }, 250);
            setTimeout(spawnLoop, 700); 
        } 
    }
</script>
</body>
</html>
