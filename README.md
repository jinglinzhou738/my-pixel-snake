<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Snake v2.6</title>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-color: #0f0c29; 
            --screen-bg: #1a1a2e;
            --snake-head: #00ff88;
            --snake-body: #00bd68;
            --food-color: #ff0077;
            --btn-color: #30336b;
        }
        * { box-sizing: border-box; touch-action: none; user-select: none; }
        body {
            background: #0f0c29; display: flex; justify-content: center; align-items: center;
            height: 100vh; margin: 0; font-family: 'VT323', monospace; color: #fff;
        }
        .handheld-container {
            background: #1c1c1c; padding: 20px; border-radius: 20px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            display: flex; flex-direction: column; align-items: center; width: 380px;
        }
        .header {
            width: 100%; display: flex; justify-content: space-between;
            background: var(--screen-bg); padding: 8px 12px;
            border: 4px solid #000; border-bottom: none; color: var(--snake-head);
        }
        #gameCanvas {
            background-color: var(--screen-bg);
            border: 4px solid #000; image-rendering: pixelated;
            display: block;
        }
        .controls {
            margin-top: 20px; width: 100%; display: grid;
            grid-template-columns: repeat(3, 1fr); gap: 10px;
        }
        .btn {
            background: var(--btn-color); border: 2px solid #000;
            box-shadow: 0 4px #000; color: #fff; font-size: 24px;
            display: flex; justify-content: center; align-items: center;
            height: 60px; cursor: pointer; border-radius: 10px;
        }
        .btn:active { transform: translateY(2px); box-shadow: 0 2px #000; }
        #btn-up { grid-column: 2; }
        #btn-left { grid-column: 1; grid-row: 2; }
        #btn-reset { grid-column: 2; grid-row: 2; color: #ff0077; font-weight: bold; font-size: 18px; }
        #btn-right { grid-column: 3; grid-row: 2; }
        #btn-down { grid-column: 2; grid-row: 3; }
    </style>
</head>
<body>

    <div class="handheld-container">
        <div class="header">
            <span>NEON SNAKE</span>
            <span>HI: <span id="hiScoreVal">0</span></span>
            <span>SC: <span id="scoreVal">0</span></span>
        </div>
        <canvas id="gameCanvas" width="340" height="340"></canvas>
        <div class="controls">
            <div class="btn" id="btn-up">▲</div>
            <div class="btn" id="btn-left">◀</div>
            <div class="btn" id="btn-reset">START</div>
            <div class="btn" id="btn-right">▶</div>
            <div class="btn" id="btn-down">▼</div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const scoreVal = document.getElementById("scoreVal");
        const hiScoreVal = document.getElementById("hiScoreVal");

        let grid = 20, count = 0, score = 0, gameState = 0;
        let speed = 15; // 速度调慢（数值越大越慢）
        let hiScore = localStorage.getItem("snakeHiScore") || 0;
        hiScoreVal.innerHTML = hiScore;

        let snake = { x: 160, y: 160, dx: grid, dy: 0, cells: [], maxCells: 4 };
        let food = { x: 60, y: 60 };

        function startGame() {
            if (score > hiScore) { hiScore = score; localStorage.setItem("snakeHiScore", hiScore); hiScoreVal.innerHTML = hiScore; }
            snake = { x: 160, y: 160, dx: grid, dy: 0, cells: [], maxCells: 4 };
            score = 0; scoreVal.innerHTML = score; gameState = 1;
        }

        function loop() {
            requestAnimationFrame(loop);
            if (++count < speed) return;
            count = 0;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            if (gameState === 0) {
                drawText("NEON SNAKE", "CLICK START", "#00ff88");
                return;
            }
            if (gameState === 2) {
                drawText("GAME OVER", "SCORE: " + score, "#ff0077");
                return;
            }

            snake.x += snake.dx; snake.y += snake.dy;
            if (snake.x < 0) snake.x = canvas.width - grid; else if (snake.x >= canvas.width) snake.x = 0;
            if (snake.y < 0) snake.y = canvas.height - grid; else if (snake.y >= canvas.height) snake.y = 0;

            snake.cells.unshift({x: snake.x, y: snake.y});
            if (snake.cells.length > snake.maxCells) snake.cells.pop();

            // 食物
            ctx.fillStyle = "#ff0077";
            ctx.fillRect(food.x + 2, food.y + 2, grid - 4, grid - 4);

            // 蛇
            snake.cells.forEach((cell, index) => {
                ctx.fillStyle = (index === 0) ? "#00ff88" : "#00bd68";
                ctx.fillRect(cell.x + 1, cell.y + 1, grid - 2, grid - 2);

                if (cell.x === food.x && cell.y === food.y) {
                    snake.maxCells += 2; // 变长反馈
                    score += 10; scoreVal.innerHTML = score;
                    food.x = Math.floor(Math.random() * (canvas.width / grid)) * grid;
                    food.y = Math.floor(Math.random() * (canvas.height / grid)) * grid;
                }

                for (let i = index + 1; i < snake.cells.length; i++) {
                    if (cell.x === snake.cells[i].x && cell.y === snake.cells[i].y) gameState = 2;
                }
            });
        }

        function drawText(t1, t2, c) {
            ctx.fillStyle = "rgba(0,0,0,0.5)"; ctx.fillRect(0,0,canvas.width,canvas.height);
            ctx.fillStyle = c; ctx.font = "40px 'VT323'"; ctx.textAlign = "center";
            ctx.fillText(t1, canvas.width/2, canvas.height/2);
            ctx.font = "20px 'VT323'"; ctx.fillStyle = "#fff"; ctx.fillText(t2, canvas.width/2, canvas.height/2 + 30);
        }

        function move(d) {
            if (gameState !== 1) return;
            if (d === 'u' && snake.dy === 0) { snake.dy = -grid; snake.dx = 0; }
            if (d === 'd' && snake.dy === 0) { snake.dy = grid; snake.dx = 0; }
            if (d === 'l' && snake.dx === 0) { snake.dx = -grid; snake.dy = 0; }
            if (d === 'r' && snake.dx === 0) { snake.dx = grid; snake.dy = 0; }
        }

        // 绑定事件：同时支持鼠标和触摸
        const bind = (id, d) => {
            const el = document.getElementById(id);
            el.onmousedown = (e) => { e.preventDefault(); if(id==='btn-reset') startGame(); else move(d); };
            el.ontouchstart = (e) => { e.preventDefault(); if(id==='btn-reset') startGame(); else move(d); };
        };

        bind('btn-up', 'u'); bind('btn-down', 'd'); bind('btn-left', 'l'); bind('btn-right', 'r');
        bind('btn-reset', '');

        document.addEventListener('keydown', e => {
            const k = {38:'u', 40:'d', 37:'l', 39:'r', 13: 's', 32: 's'};
            if (k[e.which] === 's') startGame();
            else move(k[e.which]);
        });

        request
