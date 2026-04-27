<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Snake Fix v2.7</title>
    <style>
        body { background: #0f0c29; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; font-family: sans-serif; color: #fff; overflow: hidden; }
        .handheld { background: #1c1c1c; padding: 15px; border-radius: 15px; display: flex; flex-direction: column; align-items: center; border: 2px solid #333; }
        #gameCanvas { background: #1a1a2e; border: 3px solid #000; width: 300px; height: 300px; display: block; }
        .controls { margin-top: 15px; display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; width: 100%; }
        .btn { background: #30336b; color: white; border: none; height: 50px; border-radius: 8px; font-size: 20px; cursor: pointer; display: flex; justify-content: center; align-items: center; -webkit-tap-highlight-color: transparent; }
        .btn:active { background: #4834d4; }
        #btn-reset { color: #ff0077; font-weight: bold; font-size: 16px; }
    </style>
</head>
<body>
    <div class="handheld">
        <div style="width:100%; display:flex; justify-content:space-between; margin-bottom:5px; font-size:14px;">
            <span>SCORE: <b id="sc">0</b></span>
            <span>BEST: <b id="hi">0</b></span>
        </div>
        <canvas id="gameCanvas" width="300" height="300"></canvas>
        <div class="controls">
            <div></div><button class="btn" id="u">▲</button><div></div>
            <button class="btn" id="l">◀</button>
            <button class="btn" id="btn-reset">START</button>
            <button class="btn" id="r">▶</button>
            <div></div><button class="btn" id="d">▼</button><div></div>
        </div>
    </div>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scEl = document.getElementById("sc");
    const hiEl = document.getElementById("hi");

    let grid = 20, count = 0, score = 0, running = false;
    let hiScore = localStorage.getItem("snakeHi") || 0;
    hiEl.innerText = hiScore;

    let snake = { x: 140, y: 140, dx: 20, dy: 0, cells: [], max: 4 };
    let food = { x: 40, y: 40 };

    function reset() {
        if (score > hiScore) { hiScore = score; localStorage.setItem("snakeHi", hiScore); hiEl.innerText = hiScore; }
        snake = { x: 140, y: 140, dx: 20, dy: 0, cells: [], max: 4 };
        score = 0; scEl.innerText = "0"; running = true;
    }

    function loop() {
        requestAnimationFrame(loop);
        if (!running || ++count < 15) return; // 速度调慢
        count = 0;
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        snake.x += snake.dx; snake.y += snake.dy;
        if (snake.x < 0) snake.x = 280; else if (snake.x >= 300) snake.x = 0;
        if (snake.y < 0) snake.y = 280; else if (snake.y >= 300) snake.y = 0;

        snake.cells.unshift({x: snake.x, y: snake.y});
        if (snake.cells.length > snake.max) snake.cells.pop();

        ctx.fillStyle = "#ff0077"; ctx.fillRect(food.x+2, food.y+2, 16, 16);
        ctx.fillStyle = "#00ff88";
        snake.cells.forEach((c, i) => {
            ctx.fillRect(c.x+1, c.y+1, 18, 18);
            if (c.x === food.x && c.y === food.y) {
                snake.max += 2; score += 10; scEl.innerText = score;
                food.x = Math.floor(Math.random()*15)*20;
                food.y = Math.floor(Math.random()*15)*20;
            }
            for (let j = i+1; j < snake.cells.length; j++) {
                if (c.x === snake.cells[j].x && c.y === snake.cells[j].y) running = false;
            }
        });
        if (!running) {
            ctx.fillStyle = "rgba(0,0,0,0.6)"; ctx.fillRect(0,0,300,300);
            ctx.fillStyle = "#fff"; ctx.textAlign = "center"; ctx.fillText("GAME OVER / CLICK START", 150, 150);
        }
    }

    const move = (d) => {
        if (d === 'u' && snake.dy === 0) { snake.dy = -20; snake.dx = 0; }
        if (d === 'd' && snake.dy === 0) { snake.dy = 20; snake.dx = 0; }
        if (d === 'l' && snake.dx === 0) { snake.dx = -20; snake.dy = 0; }
        if (d === 'r' && snake.dx === 0) { snake.dx = 20; snake.dy = 0; }
    };

    // 绑定点击和触摸
    const btns = { 'u':'u', 'd':'d', 'l':'l', 'r':'r' };
    Object.keys(btns).forEach(id => {
        const el = document.getElementById(id);
        const action = (e) => { e.preventDefault(); move(btns[id]); };
        el.onmousedown = action; el.ontouchstart = action;
    });
    
    const startBtn = document.getElementById('btn-reset');
    startBtn.onmousedown = startBtn.ontouchstart = (e) => { e.preventDefault(); reset(); };

    document.addEventListener('keydown', e => {
        if (e.keyCode === 32) reset();
        const k = {38:'u', 40:'d', 37:'l', 39:'r'};
        if (k[e.keyCode]) move(k[e.keyCode]);
    });

    requestAnimationFrame(loop);
</script>
</body>
</html>
