<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Pixel Snake v2.5</title>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <style>
        :root {
            /* 升级配色：深紫色背景 + 荧光绿/粉色 */
            --bg-color: #0f0c29; 
            --console-body: #1c1c1c;
            --screen-bg: #1a1a2e;
            --grid-line: rgba(78, 205, 196, 0.05);
            --snake-head: #00ff88; /* 荧光绿 */
            --snake-body: #00bd68;
            --food-color: #ff0077; /* 霓虹粉 */
            --btn-color: #30336b;
            --text-glow: 0 0 10px rgba(0, 255, 136, 0.5);
        }
        * { box-sizing: border-box; touch-action: none; }
        body {
            background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
            display: flex; justify-content: center; align-items: center;
            height: 100vh; margin: 0; font-family: 'VT323', monospace;
            overflow: hidden;
        }
        .handheld-container {
            background: var(--console-body); padding: 25px; border-radius: 20px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5), inset 0 0 10px rgba(255,255,255,0.1);
            display: flex; flex-direction: column; align-items: center; 
            width: 420px; /* 界面变大 */
            border: 2px solid #444;
        }
        .header {
            width: 100%; display: flex; justify-content: space-between;
            background-color: var(--screen-bg); padding: 10px 15px;
            border: 4px solid #000; border-bottom: none; font-size: 22px; color: var(--snake-head);
            text-shadow: var(--text-glow);
        }
        #gameCanvas {
            background-color: var(--screen-bg);
            background-image: linear-gradient(var(--grid-line) 1px, transparent 1px),
                              linear-gradient(90deg, var(--grid-line) 1px, transparent 1px);
            background-size: 20px 20px;
            border: 4px solid #000; image-rendering: pixelated;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.8);
        }
        .controls {
            margin-top: 25px; width: 100%; display: grid;
            grid-template-columns: repeat(3, 1fr); gap: 12px;
        }
        .btn {
            background-color: var(--btn-color); border: 2px solid #000;
            box-shadow: 0 4px #000; color: #fff;
            font-size: 28px; display: flex; justify-content: center;
            align-items: center; cursor: pointer; height: 60px; user-select: none;
            border-radius: 10px;
        }
        .btn:active { transform: translateY(2px); box-shadow: 0 2px #000; background: #3c40c6; }
        #btn-up { grid-column: 2; }
        #btn-left { grid-column: 1; grid-row: 2; }
        #btn-reset { grid-column: 2; grid-row: 2; font-size: 18px; color: #ff0077; font-weight: bold; }
        #btn-right { grid-column: 3; grid-row: 2; }
        #btn-down { grid-column: 2; grid-row: 3; }
    </style>
</head>
<body>

    <div class="handheld-container">
        <div class="header">
            <span>NEON SNAKE</span>
            <span>HI: <span id="hiScoreVal">0</span></span>
            <span>SCORE: <span id="scoreVal">0</span></span>
        </div>
        <canvas id="gameCanvas" width="360" height="360"></canvas>
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

        // 核心参数调整
        let grid = 20; // 网格变大
        let count = 0; 
        let speed = 12; // 调慢速度（数值越大越慢，之前是8）
        let score = 0, gameState = 0; 
        let hiScore = localStorage.getItem("snakeHiScore") || 0;
        hiScoreVal.innerHTML = hiScore;

        let snake = { x: grid*5, y: grid*5, dx: grid, dy: 0, cells: [], maxCells: 4 };
        let food = { x: grid*10, y: grid*10 };

        function getRandomInt(min, max) { return Math.floor(Math.random() *
