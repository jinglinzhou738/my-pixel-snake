<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Retro Pixel Snake v2.2</title>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-color: #2c3e50;
            --screen-bg: #95a5a6;
            --grid-line: rgba(0,0,0,0.05);
            --snake-head: #1e3718;
            --snake-body: #2c5e1c;
            --food-color: #a93226;
            --btn-color: #bdc3c7;
        }
        * { box-sizing: border-box; touch-action: none; }
        body {
            background-color: var(--bg-color);
            display: flex; justify-content: center; align-items: center;
            height: 100vh; margin: 0; font-family: 'VT323', monospace;
            overflow: hidden;
        }
        .handheld-container {
            background: #34495e; padding: 20px; border-radius: 15px 15px 50px 15px;
            box-shadow: 10px 10px 0px rgba(0,0,0,0.3);
            display: flex; flex-direction: column; align-items: center; width: 350px;
        }
        .header {
            width: 100%; display: flex; justify-content: space-between;
            background-color: var(--screen-bg); padding: 5px 10px;
            border: 4px solid #2c3e50; border-bottom: none; font-size: 20px;
        }
        #gameCanvas {
            background-color: var(--screen-bg);
            background-image: linear-gradient(var(--grid-line) 1px, transparent 1px),
                              linear-gradient(90deg, var(--grid-line) 1px, transparent 1px);
            background-size: 20px 20px;
            border: 4px solid #2c3e50; image-rendering: pixelated;
            transition: border-color 0.2s;
        }
        .controls {
            margin-top: 20px; width: 100%; display: grid;
            grid-template-columns: repeat(3, 1fr); gap: 8px;
        }
        .btn {
            background-color: var(--btn-color); border: 4px solid #2c3e50;
            box-shadow: 3px 3px 0px rgba(0,0,0,0.2); color: #2c3e50;
            font-size: 24px; display: flex; justify-content: center;
            align-items: center; cursor: pointer; height: 55px; user-select: none;
        }
        .btn:active { transform: translate(2px, 2px); box-shadow: 1px 1px 0px rgba(0,0,0,0.2); }
        #btn-up { grid-column: 2; }
        #btn-left { grid-column: 1; grid-row: 2; }
        #btn-reset { grid-column: 2; grid-row: 2; border-radius: 50%; font-size: 16px; font-weight: bold; background: #ecf0f1;}
        #btn-right { grid-column: 3; grid-row: 2; }
        #btn-down { grid-column: 2; grid-row: 3; }
    </style>
</head>
<body>

    <div class="handheld-container">
        <div class="header">
            <span>SNAKE v2.2</span>
            <span>HI: <span id="hiScoreVal">0</span></span>
            <span>SC: <span id="scoreVal">0</span></span>
        </div>
        <canvas id="gameCanvas" width="300" height="300"></canvas>
        <div class="controls">
            <div class="btn" id="btn-up">↑</div>
            <div class="btn" id="btn-left">←</div>
            <div class="btn" id="btn-reset">START</div>
            <div class="btn" id="btn-right">→</div>
            <div class="btn" id="btn-down">↓</div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const scoreVal = document.getElementById("scoreVal");
        const hiScoreVal = document.getElementById("hiScoreVal");

        let grid = 15, count = 0, score = 0, gameState = 0; // 0:准备, 1:运行, 2:结束
        let hiScore = localStorage.getItem("snakeHiScore") || 0;
        hiScoreVal.innerHTML = hiScore;

        let snake = { x: 150, y: 150, dx: grid, dy: 0, cells: [], maxCells: 4 };
        let food = { x: 60, y: 60 };

        function getRandomInt(min, max) { return Math.floor(Math.random() * (max - min)) + min; }

        function startGame() {
            if (score > hiScore) { 
                hiScore = score; 
                localStorage.setItem("snakeHiScore", hiScore); 
                hiScoreVal.innerHTML = hiScore; 
            }
            snake = { x: 150, y: 150, dx: grid, dy: 0, cells: [], maxCells: 4 };
            score = 0; 
            scoreVal.innerHTML = score; 
            gameState = 1;
            canvas.style.borderColor = "#2c3e50";
        }

        function drawOverlay(title, sub) {
            ctx.fillStyle = "rgba(149, 165, 166, 0.8)";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = "#2c3e50";
            ctx.font = "40px 'VT323'";
            ctx.textAlign = "center";
            ctx.fillText(title, canvas.width/2, canvas.height/2 - 10);
            ctx.font = "20px 'VT323'";
            ctx.fillText(sub, canvas.width/2, canvas.height/2 + 25);
        }

        function loop() {
            requestAnimationFrame(loop);
            if (++count < 8) return;
            count = 0;
            ctx.clearRect(0,0,canvas.width,canvas.height);

            if (gameState === 0) {
                drawOverlay("READY?", "PRESS START BUTTON");
                return;
            }

            if (gameState === 2) {
                drawOverlay("GAME OVER", "SCORE: " + score + " | RESTART?");
                return;
            }

            // 移动逻辑
            snake.x += snake.dx;
            snake.y += snake.dy;

            // 穿墙
            if (snake.x < 0) snake.x = canvas.width - grid; 
            else if (snake.x >= canvas.width) snake.x = 0;
            if (snake.y < 0) snake.y = canvas.height - grid; 
            else if (snake.y >= canvas.height) snake.y = 0;

            snake.cells.unshift({x: snake.x, y: snake.y});
            if (snake.cells.length > snake.maxCells) snake.cells.pop();

            // 画食物
            ctx.fillStyle = "#a93226";
            ctx.fillRect(food.x+2, food.y+2, grid-4, grid-4);

            // 画蛇
            snake.cells.forEach((cell, index) => {
                ctx.fillStyle = (index === 0) ? "#1e3718" : "#2c5e1c";
                ctx.fillRect(cell.x, cell.y, grid-1, grid-1);

                // 吃到食物
                if (cell.x === food.x && cell.y === food.y) {
                    snake.maxCells++;
                    score += 10;
                    scoreVal.innerHTML = score;
                    food.x = getRandomInt(0, 20) * grid;
                    food.y = getRandomInt(0, 20) * grid;
                }

                // 撞到自己
                for (let i = index + 1; i < snake.cells.length; i++) {
                    if (cell.x === snake.cells[i].x && cell.y === snake.cells[i].y) {
                        gameState = 2;
                        canvas.style.borderColor = "#e74c3c"; // 撞击瞬间红框反馈
                    }
                }
            });
        }

        function handleInput(dir) {
            if (gameState !== 1) return;
            if (dir === 'up' && snake.dy === 0) { snake.dy = -grid; snake.dx = 0; }
            if (dir === 'down' && snake.dy === 0) { snake.dy = grid; snake.dx = 0; }
            if (dir === 'left' && snake.dx === 0) { snake.dx = -grid; snake.dy = 0; }
            if (dir === 'right' && snake.dx === 0) { snake.dx = grid; snake.dy = 0; }
        }

        // 键盘支持
        document.addEventListener('keydown', e => {
            if (gameState !== 1) {
                if(e.which === 13 || e.which === 32) startGame();
            }
            const keys = {38:'up', 40:'down', 37:'left', 39:'right'};
            if (keys[e.which]) handleInput(keys[e.which]);
        });

        // 按钮支持
        const btns = { 'btn-up':'up', 'btn-down':'down', 'btn-left':'left', 'btn-right':'right' };
        Object.keys(btns).forEach(id => {
            const el = document.getElementById(id);
            el.onmousedown = (e) => { e.preventDefault(); handleInput(btns[id]); };
            el.ontouchstart = (e) => { e.preventDefault(); handleInput(btns[id]); };
        });
        document.getElementById('btn-reset').onclick = startGame;

        requestAnimationFrame(loop);
    </script>
</body>
</html>
