# Run-Bakugo
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bakugo rescta a kirishima</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background: linear-gradient(135deg, #ff416c, #ff4b2b);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #fff;
        }

        #game-card {
            background: rgba(0, 0, 0, 0.35);
            padding: 25px;
            border-radius: 16px;
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.4);
            backdrop-filter: blur(10px);
            text-align: center;
            border: 1px solid rgba(255, 255, 255, 0.2);
        }

        h1 {
            font-size: 1.8rem;
            margin-bottom: 15px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
            letter-spacing: 1px;
        }

        canvas {
            border-bottom: 4px solid #ff4b2b;
            border-radius: 8px;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.2);
            display: block;
            margin: 0 auto;
        }

        .instructions {
            margin-top: 15px;
            font-size: 0.9rem;
            opacity: 0.9;
            font-weight: 500;
        }

        .key {
            background: rgba(255, 255, 255, 0.2);
            padding: 2px 8px;
            border-radius: 4px;
            border: 1px solid rgba(255, 255, 255, 0.4);
        }
    </style>
</head>
<body>

<div id="game-card">
    <h1>Bakugo RUNNER</h1>
    <canvas id="gameCanvas" width="600" height="150"></canvas>
    <div class="instructions">
        <span class="key">Espacio</span> / <span class="key">Flecha Arriba</span> para saltar | 
        <span class="key">Flecha Abajo</span> / <span class="key">S</span> para agacharte
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    // === CARGA DE IMÁGENES ===
    const dinoImg = new Image();
    dinoImg.src = 'bajo.png';

    const dinoJumpImg = new Image();
    dinoJumpImg.src = 'jump.png';

    const dinoDuckImg = new Image();
    dinoDuckImg.src = 'Captura_de_pantalla_2026-09-05_113851-removebg-preview.png';

    const cactusImg = new Image();
    cactusImg.src = 'Muro.png';

    const birdImg = new Image();
    birdImg.src = 'ave.png';

    const bgImg = new Image();
    bgImg.src = 'city.jpg';

    const victoryImg = new Image();
    victoryImg.src = 'win.png';

    // === ESTADO DEL JUEGO ===
    let isPlaying = false;
    let gameOver = false;
    let gameWon = false;
    let score = 0;
    let gameSpeed = 5;

    let bgX1 = 0;
    let bgX2 = canvas.width;

    const STAND_WIDTH = 40;
    const STAND_HEIGHT = 40;
    const DUCK_WIDTH = 45;
    const DUCK_HEIGHT = 22;
    const GROUND_Y = 90;

    const dino = {
        x: 50,
        y: GROUND_Y,
        width: STAND_WIDTH,
        height: STAND_HEIGHT,
        vy: 0,
        gravity: 0.6,
        jumpPower: -10,
        isJumping: false,
        isDucking: false
    };

    let obstacles = [];
    let spawnTimer = 0;

    function jump() {
        if (!isPlaying || gameOver || gameWon) {
            resetGame();
            isPlaying = true;
            return;
        }
        if (!dino.isJumping && !dino.isDucking) {
            dino.vy = dino.jumpPower;
            dino.isJumping = true;
        }
    }

    // CONTROLES
    document.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') {
            jump();
        }
        if ((e.code === 'ArrowDown' || e.code === 'KeyS') && isPlaying && !dino.isJumping) {
            dino.isDucking = true;
            dino.height = DUCK_HEIGHT;
            dino.width = DUCK_WIDTH;
            dino.y = GROUND_Y + (STAND_HEIGHT - DUCK_HEIGHT);
        }
    });

    document.addEventListener('keyup', (e) => {
        if (e.code === 'ArrowDown' || e.code === 'KeyS') {
            dino.isDucking = false;
            dino.height = STAND_HEIGHT;
            dino.width = STAND_WIDTH;
            dino.y = GROUND_Y;
        }
    });

    canvas.addEventListener('mousedown', jump);
    canvas.addEventListener('touchstart', jump);

    function resetGame() {
        isPlaying = false;
        gameOver = false;
        gameWon = false;
        score = 0;
        gameSpeed = 5;
        dino.y = GROUND_Y;
        dino.vy = 0;
        dino.isJumping = false;
        dino.isDucking = false;
        dino.width = STAND_WIDTH;
        dino.height = STAND_HEIGHT;
        obstacles = [];
        spawnTimer = 0;
        bgX1 = 0;
        bgX2 = canvas.width;
    }

    function update() {
        if (!isPlaying || gameOver || gameWon) return;

        bgX1 -= gameSpeed * 0.5;
        bgX2 -= gameSpeed * 0.5;
        if (bgX1 <= -canvas.width) bgX1 = canvas.width;
        if (bgX2 <= -canvas.width) bgX2 = canvas.width;

        dino.vy += dino.gravity;
        dino.y += dino.vy;

        const currentGround = dino.isDucking ? GROUND_Y + (STAND_HEIGHT - DUCK_HEIGHT) : GROUND_Y;

        if (dino.y >= currentGround) {
            dino.y = currentGround;
            dino.vy = 0;
            dino.isJumping = false;
        }

        spawnTimer++;
        if (spawnTimer > Math.max(45, 90 - Math.floor(score / 50))) {
            const isHigh = Math.random() > 0.5;

            if (isHigh) {
                obstacles.push({
                    x: canvas.width,
                    y: 65,
                    width: 35,
                    height: 25,
                    type: 'high'
                });
            } else {
                obstacles.push({
                    x: canvas.width,
                    y: 95,
                    width: 25,
                    height: 35,
                    type: 'low'
                });
            }
            spawnTimer = 0;
        }

        for (let i = obstacles.length - 1; i >= 0; i--) {
            let obs = obstacles[i];
            obs.x -= gameSpeed;

            if (
                dino.x < obs.x + obs.width &&
                dino.x + dino.width > obs.x &&
                dino.y < obs.y + obs.height &&
                dino.y + dino.height > obs.y
            ) {
                gameOver = true;
            }

            if (obs.x + obs.width < 0) {
                obstacles.splice(i, 1);
            }
        }

        score++;
        gameSpeed += 0.001;

        if (score >= 2000) {
            gameWon = true;
        }
    }

    function draw() {
        if (bgImg.complete && bgImg.naturalWidth !== 0) {
            ctx.drawImage(bgImg, bgX1, 0, canvas.width, canvas.height);
            ctx.drawImage(bgImg, bgX2, 0, canvas.width, canvas.height);
        } else {
            ctx.fillStyle = '#f7f7f7';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        if (!gameWon) {
            let currentDinoImg = dinoImg;
            
            if (dino.isJumping && dinoJumpImg.complete && dinoJumpImg.naturalWidth !== 0) {
                currentDinoImg = dinoJumpImg;
            } else if (dino.isDucking && dinoDuckImg.complete && dinoDuckImg.naturalWidth !== 0) {
                currentDinoImg = dinoDuckImg;
            }

            ctx.drawImage(currentDinoImg, dino.x, dino.y, dino.width, dino.height);

            obstacles.forEach(obs => {
                if (obs.type === 'high') {
                    ctx.drawImage(birdImg.complete && birdImg.naturalWidth !== 0 ? birdImg : cactusImg, obs.x, obs.y, obs.width, obs.height);
                } else {
                    ctx.drawImage(cactusImg, obs.x, obs.y, obs.width, obs.height);
                }
            });
        }

        ctx.fillStyle = '#ffffff';
        ctx.strokeStyle = '#000000';
        ctx.lineWidth = 3;
        ctx.font = 'bold 16px monospace';
        ctx.strokeText(`SCORE: ${Math.floor(score)} / 2000`, 390, 25);
        ctx.fillText(`SCORE: ${Math.floor(score)} / 2000`, 390, 25);

        if (!isPlaying) {
            drawMessage('PRESIONA ESPACIO PARA EMPEZAR', '#ff4b2b');
        } else if (gameOver) {
            drawMessage('¡GAME OVER!', '#d32f2f', 'Presiona Espacio para Reiniciar');
        } else if (gameWon) {
            if (victoryImg.complete && victoryImg.naturalWidth !== 0) {
                ctx.drawImage(victoryImg, (canvas.width / 2) - 100, 10, 200, 100);
            }
            drawMessage('¡¡¡¡ GANASTE !!!!', '#2e7d32', 'Presiona Espacio para Volver a Jugar');
        }
    }

    function drawMessage(title, color, subtitle = '') {
        ctx.fillStyle = 'rgba(0, 0, 0, 0.6)';
        const bannerY = gameWon ? 110 : 25;
        const bannerHeight = gameWon ? 38 : 100;
        
        ctx.fillRect(80, bannerY, 440, bannerHeight);

        ctx.fillStyle = color;
        ctx.font = 'bold 18px sans-serif';
        ctx.textAlign = 'center';
        
        if (gameWon) {
            ctx.fillText(`${title} - ${subtitle}`, canvas.width / 2, bannerY + 25);
        } else {
            ctx.fillText(title, canvas.width / 2, bannerY + 40);
            if (subtitle) {
                ctx.fillStyle = '#ffffff';
                ctx.font = '14px sans-serif';
                ctx.fillText(subtitle, canvas.width / 2, bannerY + 70);
            }
        }
        ctx.textAlign = 'start';
    }

    function loop() {
        update();
        draw();
        requestAnimationFrame(loop);
    }

    loop();
</script>
</body>
</html>
