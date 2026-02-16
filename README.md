<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Finger Explosion: Game AI Berbasis Gerakan</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Inter:wght@400;600&display=swap" rel="stylesheet">

    <style>
        :root { 
            --p1-color: #00d2ff; 
            --p2-color: #ff007f; 
            --gh-bg: #0d1117; 
            --gh-card: #161b22; 
            --gh-border: #30363d;
            --gh-text: #c9d1d9;
            --gh-success: #238636;
        }

        body, html { 
            margin: 0; padding: 0; width: 100%; height: 100%; 
            background: #000; 
            color: var(--gh-text); 
            font-family: 'Inter', sans-serif; 
            overflow: hidden; 
        }

        canvas { width: 100vw; height: 100vh; object-fit: cover; transform: scaleX(-1); }

        /* Overlay Menu ala GitHub Card */
        .overlay { 
            position: fixed; top: 0; left: 0; width: 100%; height: 100%; 
            background: rgba(0,0,0,0.8); 
            z-index: 100; display: flex; justify-content: center; align-items: center; 
        }

        .gh-container { 
            background: var(--gh-card); 
            border: 1px solid var(--gh-border); 
            border-radius: 6px; 
            width: 90%; max-width: 600px; 
            padding: 24px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.5);
        }

        .gh-header { 
            border-bottom: 1px solid var(--gh-border); 
            padding-bottom: 16px; margin-bottom: 20px; 
        }

        h1 { 
            font-family: 'Orbitron'; font-size: 1.5rem; margin: 0; color: #fff; 
            display: flex; align-items: center; gap: 10px;
        }

        .badge-container { display: flex; gap: 8px; margin-top: 10px; }
        .badge { 
            font-size: 12px; font-weight: 600; padding: 0 7px; 
            border-radius: 20px; border: 1px solid var(--gh-border); 
            line-height: 18px; color: #fff;
        }

        /* Form Styling ala GitHub */
        .form-group { margin-bottom: 15px; text-align: left; }
        label { display: block; margin-bottom: 8px; font-weight: 600; font-size: 14px; }
        
        select, input[type="number"] {
            width: 100%; background: var(--gh-bg); border: 1px solid var(--gh-border);
            color: #fff; padding: 8px 12px; border-radius: 6px; outline: none;
            font-family: 'Inter'; box-sizing: border-box;
        }

        .bot-check {
            display: flex; align-items: center; gap: 8px; margin-top: 15px;
            padding: 10px; background: rgba(255, 0, 127, 0.1);
            border-radius: 6px; border: 1px solid rgba(255, 0, 127, 0.2);
            cursor: pointer; color: var(--p2-color); font-weight: bold;
        }

        .btn-green {
            width: 100%; padding: 12px; margin-top: 20px;
            background: var(--gh-success); color: #fff; border: 1px solid rgba(240,246,252,0.1);
            border-radius: 6px; font-weight: 600; font-family: 'Inter'; cursor: pointer;
            font-size: 14px; transition: 0.2s;
        }
        .btn-green:hover { background: #2ea043; }

        /* HUD & Game Elements */
        #countdown-ui { 
            position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); 
            font-size: 8rem; font-family: 'Orbitron'; color: #fff; z-index: 200; 
            text-shadow: 0 0 20px var(--p1-color); display: none; 
        }

        .hud { 
            position: absolute; top: 20px; width: 100%; display: flex; 
            justify-content: space-around; pointer-events: none; z-index: 10; opacity: 0; 
        }
        .score-box { 
            padding: 10px 30px; border-radius: 12px; background: rgba(0,0,0,0.7); 
            border: 2px solid #fff; font-family: 'Orbitron'; text-align: center; 
        }
    </style>
</head>
<body>

<canvas id="output_canvas"></canvas>
<div id="countdown-ui">3</div>

<div id="start-menu" class="overlay">
    <div class="gh-container">
        <div class="gh-header">
            <h1>🖐️ Finger Explosion</h1>
            <div class="badge-container">
                <span class="badge" style="background: #0366d6;">license MIT</span>
                <span class="badge" style="background: #e1e4e8; color: #000;">AI MediaPipe</span>
                <span class="badge" style="background: #f66a0a;">Library jQuery</span>
            </div>
        </div>
        
        <p style="font-size: 14px; color: #8b949e; margin-bottom: 20px;">
            Game web bertema neon di mana jari telunjukmu menjadi senjata! Duel real-time menggunakan tracking tangan AI.
        </p>

        <div class="form-group">
            <label>Target Kemenangan</label>
            <input type="number" id="goal-input" value="10">
        </div>

        <div class="form-group">
            <label>Pilih Webcam</label>
            <select id="cam_select"></select>
        </div>

        <label class="bot-check">
            <input type="checkbox" id="vs-bot" style="width: 18px; height: 18px;"> 
            AKTIFKAN MODE LAWAN BOT (AI)
        </label>

        <button class="btn-green" onclick="startGame()">ENTER ARENA</button>
    </div>
</div>

<div id="winner-display" class="overlay" style="display: none;">
    <div class="gh-container" style="text-align: center;">
        <h1 id="winner-name" style="justify-content: center; font-size: 2rem; margin-bottom: 20px;">PLAYER WINS!</h1>
        <button class="btn-green" onclick="location.reload()">REMATCH</button>
    </div>
</div>

<div class="hud">
    <div class="score-box" style="border-color: var(--p1-color); color: var(--p1-color);">
        <div style="font-size: 12px;">PLAYER 1</div>
        <div id="p1-score" style="font-size: 2rem;">0</div>
    </div>
    <div style="text-align: center; font-family: 'Orbitron';">
        <div style="color: #aaa; font-size: 10px;">GOAL</div>
        <div id="win-goal" style="font-size: 1.2rem;">10</div>
    </div>
    <div class="score-box" style="border-color: var(--p2-color); color: var(--p2-color);">
        <div id="p2-label" style="font-size: 12px;">PLAYER 2</div>
        <div id="p2-score" style="font-size: 2rem;">0</div>
    </div>
</div>

<video id="input_video" style="display:none"></video>

<script>
    // --- LOGIKA GAME (SAMA SEPERTI SEBELUMNYA) ---
    let scores = { p1: 0, p2: 0 }, winScore = 10, isRunning = false, gameActive = false, isSinglePlayer = false;
    let hands, camera, canvasCtx, canvasElement, targets = [], particles = [], trailHistory = { p1: [], p2: [] }, screenShake = 0;
    const SMOOTH_FACTOR = 0.25;
    let smoothPos = { p1: { x: 0, y: 0, active: false }, p2: { x: 0, y: 0, active: false } };

    function lerp(start, end, amt) { return (1 - amt) * start + amt * end; }
    const sfxSlice = new Audio('assets/slice.mp3'), sfxStart = new Audio('assets/game-start.mp3'), sfxOver = new Audio('assets/game-over.mp3');

    function playSfx(audio) { const sound = audio.cloneNode(); sound.volume = 0.5; sound.play().catch(e => {}); }

    function createExplosion(x, y, color) {
        screenShake = 12;
        for (let i = 0; i < 15; i++) {
            particles.push({ x: x, y: y, vx: (Math.random()-0.5)*15, vy: (Math.random()-0.5)*15, life: 1.0, color: color, size: Math.random()*5+2 });
        }
    }

    function init() {
        canvasElement = document.getElementById('output_canvas');
        canvasCtx = canvasElement.getContext('2d');
        hands = new Hands({locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`});
        hands.setOptions({ maxNumHands: 2, modelComplexity: 1, minDetectionConfidence: 0.5, minTrackingConfidence: 0.5 });
        hands.onResults(onResults);
        navigator.mediaDevices.enumerateDevices().then(devices => {
            devices.filter(d => d.kind === 'videoinput').forEach(d => {
                $('#cam_select').append(`<option value="${d.deviceId}">${d.label || 'Kamera'}</option>`);
            });
        });
    }

    function createTarget(side) {
        const minX = side === 0 ? 0.55 : 0.05;
        const maxX = side === 0 ? 0.95 : 0.45;
        return { baseX: Math.random()*(maxX-minX)+minX, baseY: Math.random()*0.6+0.2, side: side, r: 35, color: side === 0 ? '#00d2ff' : '#ff007f', phase: Math.random()*10, speed: 0.02+Math.random()*0.02, active: true };
    }

    function startGame() {
        isSinglePlayer = $('#vs-bot').is(':checked');
        if(isSinglePlayer) $('#p2-label').text("BOT AI");
        winScore = parseInt($('#goal-input').val()) || 10;
        $('#win-goal').text(winScore);
        $('#start-menu').hide();
        $('.hud').css('opacity', '1');
        camera = new Camera(document.getElementById('input_video'), {
            onFrame: async () => { if(isRunning) await hands.send({image: document.getElementById('input_video')}); },
            width: 1280, height: 720, deviceId: $('#cam_select').val()
        });
        camera.start();
        isRunning = true;
        playSfx(sfxStart); 
        let count = 3;
        $('#countdown-ui').show().text(count);
        const timer = setInterval(() => {
            count--;
            if (count > 0) $('#countdown-ui').text(count);
            else if (count === 0) $('#countdown-ui').text("GO!");
            else { clearInterval(timer); $('#countdown-ui').fadeOut(200); for(let i=0; i<winScore; i++) { targets.push(createTarget(0)); targets.push(createTarget(1)); } gameActive = true; }
        }, 1000);
    }

    function onResults(results) {
        canvasElement.width = window.innerWidth; canvasElement.height = window.innerHeight;
        canvasCtx.save();
        if (screenShake > 0) { canvasCtx.translate((Math.random()-0.5)*screenShake, (Math.random()-0.5)*screenShake); screenShake *= 0.9; }
        canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
        canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);

        particles.forEach((p, i) => {
            p.x += p.vx; p.y += p.vy; p.life -= 0.03; if (p.life <= 0) particles.splice(i, 1);
            canvasCtx.beginPath(); canvasCtx.arc(p.x, p.y, p.size, 0, Math.PI*2); canvasCtx.fillStyle = p.color; canvasCtx.globalAlpha = p.life; canvasCtx.fill();
        });
        canvasCtx.globalAlpha = 1.0;

        let detectedP1 = false, detectedP2 = false;
        if (isSinglePlayer && gameActive) {
            let bt = targets.find(t => t.active && t.side === 1);
            if (bt) {
                let tx = (bt.baseX + Math.sin(bt.phase)*0.1)*canvasElement.width, ty = (bt.baseY + Math.cos(bt.phase)*0.1)*canvasElement.height;
                smoothPos.p2.x = lerp(smoothPos.p2.x, tx, 0.12); smoothPos.p2.y = lerp(smoothPos.p2.y, ty, 0.12); smoothPos.p2.active = true; detectedP2 = true;
            }
        }

        if (results.multiHandLandmarks) {
            results.multiHandLandmarks.forEach(landmarks => {
                const rx = landmarks[8].x * canvasElement.width, ry = landmarks[8].y * canvasElement.height;
                const isP1 = landmarks[8].x > 0.5;
                if (isSinglePlayer && !isP1) return;
                const pKey = isP1 ? 'p1' : 'p2';
                if(isP1) detectedP1 = true; else if(!isSinglePlayer) detectedP2 = true;
                if (!smoothPos[pKey].active) { smoothPos[pKey].x = rx; smoothPos[pKey].y = ry; smoothPos[pKey].active = true; }
                else { smoothPos[pKey].x = lerp(smoothPos[pKey].x, rx, SMOOTH_FACTOR); smoothPos[pKey].y = lerp(smoothPos[pKey].y, ry, SMOOTH_FACTOR); }
            });
        }

        ['p1', 'p2'].forEach(pKey => {
            if (!smoothPos[pKey].active) return;
            const fx = smoothPos[pKey].x, fy = smoothPos[pKey].y, col = pKey === 'p1' ? '#00d2ff' : '#ff007f';
            trailHistory[pKey].push({x: fx, y: fy}); if(trailHistory[pKey].length > 15) trailHistory[pKey].shift();
            if(trailHistory[pKey].length > 2) {
                canvasCtx.beginPath(); canvasCtx.lineWidth = 12; canvasCtx.lineCap = 'round'; canvasCtx.strokeStyle = col;
                trailHistory[pKey].forEach((pos, i) => { canvasCtx.globalAlpha = i/trailHistory[pKey].length; if(i==0) canvasCtx.moveTo(pos.x, pos.y); else canvasCtx.lineTo(pos.x, pos.y); });
                canvasCtx.stroke(); canvasCtx.globalAlpha = 1.0;
            }
            if(gameActive) {
                targets.forEach(t => {
                    if (!t.active) return;
                    const tx = (t.baseX + Math.sin(t.phase)*0.1)*canvasElement.width, ty = (t.baseY + Math.cos(t.phase)*0.1)*canvasElement.height;
                    if (Math.sqrt((fx-tx)**2 + (fy-ty)**2) < t.r+20) {
                        if ((t.side===0 && pKey==='p1') || (t.side===1 && pKey==='p2')) { createExplosion(tx, ty, t.color); if(t.side===0) scores.p1++; else scores.p2++; playSfx(sfxSlice); t.active = false; checkWin(); }
                    }
                });
            }
        });

        if(!detectedP1) { smoothPos.p1.active = false; trailHistory.p1 = []; }
        if(!detectedP2 && !isSinglePlayer) { smoothPos.p2.active = false; trailHistory.p2 = []; }

        if(gameActive) {
            targets.forEach(t => {
                if (!t.active) return;
                t.phase += t.speed;
                const tx = (t.baseX + Math.sin(t.phase)*0.1)*canvasElement.width, ty = (t.baseY + Math.cos(t.phase)*0.1)*canvasElement.height;
                canvasCtx.beginPath(); canvasCtx.arc(tx, ty, t.r, 0, Math.PI*2); canvasCtx.fillStyle = t.color; canvasCtx.shadowBlur = 25; canvasCtx.shadowColor = t.color; canvasCtx.fill();
            });
        }
        canvasCtx.restore();
    }

    function checkWin() {
        $('#p1-score').text(scores.p1); $('#p2-score').text(scores.p2);
        if ((scores.p1 >= winScore || scores.p2 >= winScore) && gameActive) {
            gameActive = false; playSfx(sfxOver);
            const winner = scores.p1 >= winScore ? "PLAYER 1" : (isSinglePlayer ? "BOT AI" : "PLAYER 2");
            $('#winner-name').text(winner + " WINS!").css('color', scores.p1 >= winScore ? '#00d2ff' : '#ff007f');
            $('#winner-display').fadeIn(500).css('display', 'flex');
        }
    }
    init();
</script>
</body>
</html>
