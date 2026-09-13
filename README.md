<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MS101 Adventure - Math Edition (English)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #1a1a24;
            font-family: 'Inter', sans-serif;
            touch-action: none;
        }

        #gameCanvas {
            display: block;
            width: 100%;
            height: 100%;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            box-sizing: border-box;
        }

        .hud-text {
            color: white;
            text-shadow: 1px 1px 3px rgba(0,0,0,0.8);
        }

        #dpad {
            position: absolute;
            bottom: 30px;
            left: 30px;
            width: 150px;
            height: 150px;
            pointer-events: auto;
            display: none;
        }

        @media (max-width: 768px) {
            #dpad { display: block; }
            #instructions { display: none; }
        }

        .dpad-btn {
            position: absolute;
            width: 50px;
            height: 50px;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-weight: bold;
            user-select: none;
            backdrop-filter: blur(4px);
            border: 2px solid rgba(255, 255, 255, 0.4);
            transition: background 0.1s;
        }
        .dpad-btn:active {
            background: rgba(255, 255, 255, 0.5);
        }

        #up-btn { top: 0; left: 50px; }
        #down-btn { bottom: 0; left: 50px; }
        #left-btn { top: 50px; left: 0; }
        #right-btn { top: 50px; right: 0; }
        
        .vignette {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle, rgba(0,0,0,0) 40%, rgba(0,0,0,0.8) 100%);
            pointer-events: none;
        }

        #action-btn-container {
            position: absolute;
            bottom: 30px;
            right: 30px;
            pointer-events: auto;
            display: none;
        }
        @media (max-width: 768px) {
            #action-btn-container { display: block; }
        }
        .action-btn {
            width: 70px;
            height: 70px;
            background: rgba(241, 196, 15, 0.3);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #f1c40f;
            font-size: 24px;
            font-weight: bold;
            user-select: none;
            backdrop-filter: blur(4px);
            border: 2px solid rgba(241, 196, 15, 0.6);
            transition: background 0.1s;
        }
        .action-btn:active { background: rgba(241, 196, 15, 0.6); }
    </style>
</head>
<body>

    <canvas id="gameCanvas"></canvas>
    
    <div class="vignette"></div>

    <div id="ui-layer">
        <div>
            <h1 class="text-3xl font-extrabold hud-text tracking-wide">MS101 ADVENTURE - MATH EDITION</h1>
            <p id="instructions" class="text-sm font-semibold hud-text mt-2 opacity-80">
                Use W, A, S, D or arrow keys to move.<br>
                Find and open Mystery Boxes (Press 'E' or use Action button)! Avoid 3 mistakes!
            </p>
        </div>
        <div class="flex flex-col items-end gap-2">
            <div class="text-xl font-bold text-yellow-400 hud-text bg-black/40 px-3 py-1 rounded-full border border-yellow-500/30">
                Score: <span id="score-display">0</span>
            </div>
            <div class="text-md font-bold text-red-400 hud-text bg-black/40 px-3 py-1 rounded-full border border-red-500/30">
                Mistakes: <span id="mistakes-display">0</span>/3
            </div>
            <div class="text-right text-xs hud-text opacity-50 font-bold self-end">
                MS101 Math Edition v2.0
            </div>
        </div>
    </div>

    <!-- Mobile Controls -->
    <div id="dpad">
        <div id="up-btn" class="dpad-btn">▲</div>
        <div id="left-btn" class="dpad-btn">◀</div>
        <div id="right-btn" class="dpad-btn">▶</div>
        <div id="down-btn" class="dpad-btn">▼</div>
    </div>

    <!-- Mobile Action Button -->
    <div id="action-btn-container">
        <div id="action-btn" class="action-btn">🎁</div>
    </div>

    <!-- Question Modal -->
    <div id="question-modal" class="hidden absolute top-0 left-0 w-full h-full bg-black/80 flex items-center justify-center z-50 pointer-events-auto">
        <div class="bg-slate-800 p-6 rounded-xl border-4 border-yellow-500 text-white max-w-md w-full mx-4 shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-2xl font-bold text-yellow-400 flex items-center gap-2">
                    <span>🎁</span> MS101 Math Challenge
                </h2>
                <div id="q-timer" class="text-3xl font-black text-red-400 drop-shadow-md">10s</div>
            </div>
            <p id="q-text" class="mb-6 text-lg font-semibold">Question here?</p>
            <div id="q-options" class="flex flex-col gap-3"></div>
            <p id="q-feedback" class="mt-4 font-bold text-center text-xl hidden"></p>
        </div>
    </div>

    <!-- Game Over Modal -->
    <div id="gameover-modal" class="hidden absolute top-0 left-0 w-full h-full bg-black/90 flex items-center justify-center z-50 pointer-events-auto">
        <div class="bg-slate-900 p-8 rounded-2xl border-4 border-red-500 text-white max-w-md w-full mx-4 text-center shadow-2xl">
            <h2 class="text-4xl font-black text-red-500 mb-2">GAME OVER</h2>
            <p class="text-slate-300 mb-4 text-lg">You reached 3 mistakes! Better luck next time.</p>
            <div class="text-2xl font-bold text-yellow-400 mb-6">Final Score: <span id="final-score">0</span></div>
            <button onclick="restartGame()" class="w-full py-3 bg-red-600 hover:bg-red-500 font-bold rounded-xl text-lg transition-colors cursor-pointer shadow-lg">Play Again (Restart)</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d', { alpha: false });

        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        const TILE_SIZE = 64;
        
        const mapData = [
            [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
            [1,0,0,0,1,1,0,0,0,0,0,1,1,0,0,0,0,0,0,1,1,0,0,0,1],
            [1,0,3,3,3,0,0,0,0,0,0,0,1,0,0,5,5,5,0,0,1,0,0,0,1],
            [1,0,3,0,3,3,3,0,0,1,0,0,0,0,0,5,3,5,0,0,0,0,0,0,1],
            [1,0,3,0,0,0,3,0,0,1,1,0,0,0,0,5,3,5,0,0,0,0,1,1,1],
            [1,1,1,0,0,0,3,3,0,0,0,0,0,2,2,6,3,6,2,2,2,0,1,1,1],
            [1,0,0,0,1,0,0,3,3,3,3,6,6,2,2,6,3,6,2,2,2,0,0,0,1],
            [1,0,1,1,1,0,0,0,0,0,3,3,6,2,2,6,4,6,2,2,2,0,0,0,1],
            [1,0,0,0,1,1,0,0,1,0,0,3,6,2,2,6,4,6,2,2,2,0,1,0,1],
            [1,1,0,0,0,0,0,1,1,0,0,3,6,2,2,2,4,2,2,2,2,0,1,0,1],
            [1,1,1,0,0,0,0,0,0,0,0,3,3,4,4,4,4,2,2,2,6,6,1,0,1],
            [1,0,0,0,1,1,0,0,0,0,0,0,6,2,2,2,2,2,2,2,2,6,1,0,1],
            [1,0,1,1,1,1,1,1,0,0,1,0,6,2,2,2,2,2,2,2,2,6,0,0,1],
            [1,0,0,0,0,0,0,1,0,0,1,0,0,6,6,6,6,2,2,2,6,0,0,0,1],
            [1,1,1,1,1,1,0,1,1,0,1,1,0,0,0,0,6,6,6,6,0,0,1,1,1],
            [1,1,1,1,1,1,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,1,1,1],
            [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
        ];

        const MAP_ROWS = mapData.length;
        const MAP_COLS = mapData[0].length;
        const MAP_WIDTH = MAP_COLS * TILE_SIZE;
        const MAP_HEIGHT = MAP_ROWS * TILE_SIZE;

        const solidTiles = [1, 2, 5];

        let score = 0;
        let mistakes = 0;
        let isPaused = false;
        let activeChest = null;
        let nearChest = null;
        let timerInterval = null;
        let timeLeft = 10;

        const mathQuestionPool = [
            { q: "What is 15 + 28?", options: ["41", "43", "45", "53"], ans: 1 },
            { q: "What is 8 x 7?", options: ["54", "56", "64", "72"], ans: 1 },
            { q: "If you have 50 apples and give away 18, how many are left?", options: ["30", "32", "34", "42"], ans: 1 },
            { q: "What is the square root of 144?", options: ["10", "11", "12", "14"], ans: 2 },
            { q: "What is 1/2 of 100?", options: ["25", "50", "75", "100"], ans: 1 },
            { q: "What is 100 - 45 + 15?", options: ["60", "70", "80", "90"], ans: 1 },
            { q: "How many total degrees are in a circle?", options: ["90°", "180°", "270°", "360°"], ans: 3 },
            { q: "If a box has 12 eggs, how many eggs are in 5 boxes?", options: ["50", "60", "65", "70"], ans: 1 },
            { q: "What is 9 x 9 - 15?", options: ["66", "72", "76", "86"], ans: 0 },
            { q: "What is the perimeter of a square with 10 cm sides?", options: ["20 cm", "30 cm", "40 cm", "50 cm"], ans: 2 },
            { q: "What is 3/4 of 80?", options: ["50", "60", "70", "75"], ans: 1 },
            { q: "What is 125 + 75 - 50?", options: ["100", "150", "200", "250"], ans: 1 },
            { q: "If a tray has 24 eggs, how many are in 2 trays?", options: ["42", "46", "48", "50"], ans: 2 },
            { q: "How many minutes are in 2 and a half hours?", options: ["120", "130", "150", "180"], ans: 2 },
            { q: "What is 9 x 8 / 2?", options: ["32", "36", "40", "42"], ans: 1 },
            { q: "What is the area of a rectangle with length 8cm and width 5cm?", options: ["26 cm²", "30 cm²", "40 cm²", "45 cm²"], ans: 2 },
            { q: "If shoes cost $500 with a 20% discount, how much do they cost?", options: ["$350", "$400", "$450", "$480"], ans: 1 },
            { q: "What is the next number in the pattern: 2, 4, 8, 16, ?", options: ["24", "30", "32", "36"], ans: 2 },
            { q: "How many seconds are in an hour?", options: ["3000", "3600", "7200", "6000"], ans: 1 },
            { q: "What is 45 divided by 5 plus 10?", options: ["15", "17", "19", "21"], ans: 2 },
            { q: "What is 11 x 11?", options: ["111", "121", "131", "141"], ans: 1 },
            { q: "How many total sides does a Hexagon have?", options: ["5", "6", "7", "8"], ans: 1 }
        ];

        let chests = [
            { id: 1, col: 7, row: 2, solved: false, qIndex: 0 },
            { id: 2, col: 8, row: 12, solved: false, qIndex: 1 },
            { id: 3, col: 20, row: 4, solved: false, qIndex: 2 },
            { id: 4, col: 2, row: 13, solved: false, qIndex: 3 },
            { id: 5, col: 23, row: 2, solved: false, qIndex: 4 },
            { id: 6, col: 21, row: 14, solved: false, qIndex: 5 },
            { id: 7, col: 12, row: 7, solved: false, qIndex: 6 },
            { id: 8, col: 13, row: 2, solved: false, qIndex: 7 },
            { id: 9, col: 2, row: 5, solved: false, qIndex: 8 },
            { id: 10, col: 22, row: 9, solved: false, qIndex: 9 },
            { id: 11, col: 3, row: 2, solved: false, qIndex: 10 },
            { id: 12, col: 18, row: 12, solved: false, qIndex: 11 },
            { id: 13, col: 11, row: 4, solved: false, qIndex: 12 },
            { id: 14, col: 5, row: 14, solved: false, qIndex: 13 },
            { id: 15, col: 16, row: 2, solved: false, qIndex: 14 },
            { id: 16, col: 17, row: 15, solved: false, qIndex: 15 },
            { id: 17, col: 8, row: 6, solved: false, qIndex: 16 },
            { id: 18, col: 14, row: 10, solved: false, qIndex: 17 },
            { id: 19, col: 3, row: 10, solved: false, qIndex: 18 },
            { id: 20, col: 23, row: 12, solved: false, qIndex: 19 }
        ];

        const player = {
            x: 150,
            y: 200,
            radius: 14,
            speed: 4,
            color: '#ff4757',
            vx: 0,
            vy: 0,
            isMoving: false,
            bobbing: 0,
            facing: 'down'
        };

        const keys = {
            w: false, a: false, s: false, d: false,
            ArrowUp: false, ArrowLeft: false, ArrowDown: false, ArrowRight: false,
            e: false, Action: false
        };

        window.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            if (keys.hasOwnProperty(key)) keys[key] = true;
            if (keys.hasOwnProperty(e.key)) keys[e.key] = true;
        });
        window.addEventListener('keyup', (e) => {
            const key = e.key.toLowerCase();
            if (keys.hasOwnProperty(key)) keys[key] = false;
            if (keys.hasOwnProperty(e.key)) keys[e.key] = false;
        });

        function bindDpad(id, keyBinding) {
            const btn = document.getElementById(id);
            const press = (e) => { e.preventDefault(); keys[keyBinding] = true; };
            const release = (e) => { e.preventDefault(); keys[keyBinding] = false; };
            
            btn.addEventListener('touchstart', press, {passive: false});
            btn.addEventListener('mousedown', press);
            btn.addEventListener('touchend', release, {passive: false});
            btn.addEventListener('mouseup', release);
            btn.addEventListener('mouseleave', release);
        }
        bindDpad('up-btn', 'w');
        bindDpad('down-btn', 's');
        bindDpad('left-btn', 'a');
        bindDpad('right-btn', 'd');
        bindDpad('action-btn', 'Action');

        const camera = { x: 0, y: 0 };

        function checkCollision(newX, newY) {
            if (newX - player.radius < 0 || newX + player.radius > MAP_WIDTH ||
                newY - player.radius < 0 || newY + player.radius > MAP_HEIGHT) {
                return true;
            }

            const offset = player.radius * 0.8;
            const corners = [
                {x: newX - offset, y: newY - offset},
                {x: newX + offset, y: newY - offset},
                {x: newX - offset, y: newY + offset},
                {x: newX + offset, y: newY + offset}
            ];

            for (let c of corners) {
                const tileX = Math.floor(c.x / TILE_SIZE);
                const tileY = Math.floor(c.y / TILE_SIZE);
                
                if (tileY >= 0 && tileY < MAP_ROWS && tileX >= 0 && tileX < MAP_COLS) {
                    if (solidTiles.includes(mapData[tileY][tileX])) return true;
                }
            }
            return false;
        }

        const modal = document.getElementById('question-modal');
        const gameOverModal = document.getElementById('gameover-modal');
        const qText = document.getElementById('q-text');
        const qOptions = document.getElementById('q-options');
        const qFeedback = document.getElementById('q-feedback');
        const scoreDisplay = document.getElementById('score-display');
        const mistakesDisplay = document.getElementById('mistakes-display');
        const finalScoreDisplay = document.getElementById('final-score');
        const qTimer = document.getElementById('q-timer');

        function openQuestionModal(chest) {
            isPaused = true;
            activeChest = chest;
            
            const currentQ = mathQuestionPool[chest.qIndex];
            qText.innerText = currentQ.q;
            qOptions.innerHTML = '';
            qFeedback.classList.add('hidden');
            
            const labels = ['A', 'B', 'C', 'D'];
            currentQ.options.forEach((opt, index) => {
                const btn = document.createElement('button');
                btn.className = 'w-full text-left px-4 py-3 bg-slate-700 hover:bg-slate-600 border border-slate-500 rounded-lg transition-colors duration-200 font-semibold cursor-pointer';
                btn.innerText = `${labels[index]}. ${opt}`;
                btn.onclick = () => handleAnswer(index);
                qOptions.appendChild(btn);
            });
            
            modal.classList.remove('hidden');

            clearInterval(timerInterval);
            timeLeft = 10;
            qTimer.innerText = timeLeft + 's';
            qTimer.classList.remove('text-red-600', 'animate-pulse');
            qTimer.classList.add('text-red-400');

            timerInterval = setInterval(() => {
                timeLeft--;
                qTimer.innerText = timeLeft + 's';
                
                if (timeLeft <= 3) {
                    qTimer.classList.add('text-red-600', 'animate-pulse');
                }
                
                if (timeLeft <= 0) {
                    clearInterval(timerInterval);
                    handleTimeout();
                }
            }, 1000);
        }

        function handleTimeout() {
            const buttons = qOptions.querySelectorAll('button');
            buttons.forEach(b => b.disabled = true);
            
            const currentQ = mathQuestionPool[activeChest.qIndex];
            qFeedback.classList.remove('hidden');
            qFeedback.innerText = "Out of time! The question has been changed.";
            qFeedback.className = "mt-4 font-bold text-center text-xl text-red-500";
            
            buttons[currentQ.ans].classList.replace('bg-slate-700', 'bg-green-600');
            
            processWrongAnswer();
        }

        function handleAnswer(index) {
            clearInterval(timerInterval);
            const buttons = qOptions.querySelectorAll('button');
            buttons.forEach(b => b.disabled = true);
            
            const currentQ = mathQuestionPool[activeChest.qIndex];
            qFeedback.classList.remove('hidden');
            
            if (index === currentQ.ans) {
                qFeedback.innerText = "Correct! +10 Points!";
                qFeedback.className = "mt-4 font-bold text-center text-xl text-green-400";
                score += 10;
                scoreDisplay.innerText = score;
                activeChest.solved = true;
                buttons[index].classList.replace('bg-slate-700', 'bg-green-600');
                closeModalAfterDelay();
            } else {
                qFeedback.innerText = "Wrong answer! The question for this box has changed.";
                qFeedback.className = "mt-4 font-bold text-center text-xl text-red-400";
                buttons[index].classList.replace('bg-slate-700', 'bg-red-600');
                buttons[currentQ.ans].classList.replace('bg-slate-700', 'bg-green-600');
                
                activeChest.qIndex = Math.floor(Math.random() * mathQuestionPool.length);
                processWrongAnswer();
            }
        }

        function processWrongAnswer() {
            mistakes++;
            mistakesDisplay.innerText = mistakes;
            
            if (mistakes >= 3) {
                setTimeout(() => {
                    modal.classList.add('hidden');
                    finalScoreDisplay.innerText = score;
                    gameOverModal.classList.remove('hidden');
                }, 1500);
            } else {
                closeModalAfterDelay();
            }
        }

        function closeModalAfterDelay() {
            setTimeout(() => {
                modal.classList.add('hidden');
                setTimeout(() => {
                    isPaused = false;
                    activeChest = null;
                }, 200);
            }, 1800);
        }

        function restartGame() {
            score = 0;
            mistakes = 0;
            scoreDisplay.innerText = score;
            mistakesDisplay.innerText = mistakes;
            player.x = 150;
            player.y = 200;
            
            chests.forEach(c => {
                c.solved = false;
                c.qIndex = Math.floor(Math.random() * mathQuestionPool.length);
            });

            gameOverModal.classList.add('hidden');
            isPaused = false;
        }

        function updatePlayer() {
            if (isPaused) return;

            nearChest = null;
            let canInteract = false;
            for (let chest of chests) {
                if (!chest.solved) {
                    const chestX = chest.col * TILE_SIZE + TILE_SIZE/2;
                    const chestY = chest.row * TILE_SIZE + TILE_SIZE/2;
                    const dist = Math.hypot(player.x - chestX, player.y - chestY);
                    
                    if (dist < TILE_SIZE * 1.2) {
                        nearChest = chest;
                        if (keys.e || keys.Action) {
                            canInteract = true;
                        }
                    }
                }
            }

            if (canInteract && nearChest) {
                openQuestionModal(nearChest);
                keys.e = false; 
                keys.Action = false;
                return;
            }

            let dx = 0;
            let dy = 0;
            if (keys.w || keys.ArrowUp) { dy -= 1; player.facing = 'up'; }
            if (keys.s || keys.ArrowDown) { dy += 1; player.facing = 'down'; }
            if (keys.a || keys.ArrowLeft) { dx -= 1; player.facing = 'left'; }
            if (keys.d || keys.ArrowRight) { dx += 1; player.facing = 'right'; }

            if (dx !== 0 && dy !== 0) {
                const length = Math.sqrt(dx * dx + dy * dy);
                dx /= length;
                dy /= length;
            }

            player.vx = dx * player.speed;
            player.vy = dy * player.speed;
            player.isMoving = (dx !== 0 || dy !== 0);

            if (player.vx !== 0 && !checkCollision(player.x + player.vx, player.y)) {
                player.x += player.vx;
            }
            if (player.vy !== 0 && !checkCollision(player.x, player.y + player.vy)) {
                player.y += player.vy;
            }

            if (player.isMoving) {
                player.bobbing += 0.2;
            } else {
                player.bobbing = 0;
            }
        }

        function updateCamera() {
            camera.x = player.x - canvas.width / 2;
            camera.y = player.y - canvas.height / 2;
            camera.x = Math.max(0, Math.min(camera.x, MAP_WIDTH - canvas.width));
            camera.y = Math.max(0, Math.min(camera.y, MAP_HEIGHT - canvas.height));
        }

        function drawTree(x, y) {
            ctx.save();
            ctx.translate(x, y);
            ctx.fillStyle = 'rgba(0,0,0,0.3)';
            ctx.beginPath();
            ctx.ellipse(TILE_SIZE/2, TILE_SIZE - 10, 16, 8, 0, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = '#6d4c41';
            ctx.fillRect(TILE_SIZE/2 - 6, TILE_SIZE/2, 12, TILE_SIZE/2 - 10);
            
            ctx.fillStyle = '#2e7d32';
            ctx.beginPath();
            ctx.arc(TILE_SIZE/2, TILE_SIZE/2 - 5, 22, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.fillStyle = '#388e3c';
            ctx.beginPath();
            ctx.arc(TILE_SIZE/2, TILE_SIZE/2 - 15, 18, 0, Math.PI * 2);
            ctx.fill();
            ctx.restore();
        }

        function drawHouseWall(x, y) {
            ctx.fillStyle = '#78909c';
            ctx.fillRect(x, y, TILE_SIZE, TILE_SIZE);
            ctx.fillStyle = '#546e7a';
            ctx.fillRect(x, y + 15, TILE_SIZE, 2);
            ctx.fillRect(x, y + 35, TILE_SIZE, 2);
            ctx.fillRect(x, y + 55, TILE_SIZE, 2);
            ctx.fillRect(x + 15, y, 2, 15);
            ctx.fillRect(x + 45, y, 2, 15);
        }

        function drawChest(x, y, solved) {
            ctx.save();
            ctx.translate(x, y);
            ctx.fillStyle = 'rgba(0,0,0,0.3)';
            ctx.beginPath();
            ctx.ellipse(TILE_SIZE/2, TILE_SIZE - 10, 14, 6, 0, 0, Math.PI * 2);
            ctx.fill();

            if (!solved) {
                ctx.fillStyle = '#f1c40f';
                ctx.fillRect(TILE_SIZE/2 - 16, TILE_SIZE/2 - 5, 32, 24);
                ctx.fillStyle = '#d35400';
                ctx.fillRect(TILE_SIZE/2 - 16, TILE_SIZE/2 + 5, 32, 3);
                ctx.fillStyle = '#bdc3c7';
                ctx.beginPath();
                ctx.arc(TILE_SIZE/2, TILE_SIZE/2 + 5, 4, 0, Math.PI*2);
                ctx.fill();
            } else {
                ctx.fillStyle = '#7f8c8d';
                ctx.fillRect(TILE_SIZE/2 - 16, TILE_SIZE/2 + 5, 32, 14);
                ctx.fillStyle = '#95a5a6';
                ctx.fillRect(TILE_SIZE/2 - 16, TILE_SIZE/2 - 15, 32, 12);
                ctx.fillStyle = '#2c3e50';
                ctx.fillRect(TILE_SIZE/2 - 14, TILE_SIZE/2 + 5, 28, 6);
            }
            ctx.restore();
        }

        let time = 0; 

        function drawMap() {
            const startCol = Math.floor(camera.x / TILE_SIZE);
            const endCol = Math.ceil((camera.x + canvas.width) / TILE_SIZE);
            const startRow = Math.floor(camera.y / TILE_SIZE);
            const endRow = Math.ceil((camera.y + canvas.height) / TILE_SIZE);

            for (let r = startRow; r <= endRow; r++) {
                for (let c = startCol; c <= endCol; c++) {
                    if (r >= 0 && r < MAP_ROWS && c >= 0 && c < MAP_COLS) {
                        const tile = mapData[r][c];
                        const x = c * TILE_SIZE;
                        const y = r * TILE_SIZE;

                        ctx.fillStyle = ((r+c)%2 === 0) ? '#8bc34a' : '#7cb342'; 
                        ctx.fillRect(x, y, TILE_SIZE, TILE_SIZE);

                        if (tile === 2) {
                            ctx.fillStyle = '#29b6f6';
                            ctx.fillRect(x, y, TILE_SIZE, TILE_SIZE);
                            ctx.fillStyle = 'rgba(255, 255, 255, 0.3)';
                            const waveOffset = Math.sin(time + c + r) * 5;
                            ctx.fillRect(x + 10 + waveOffset, y + 20, 20, 3);
                        } else if (tile === 3) {
                            ctx.fillStyle = '#d4e157';
                            ctx.fillRect(x + 10, y + 10, TILE_SIZE - 20, TILE_SIZE - 20);
                        } else if (tile === 6) {
                            ctx.fillStyle = '#ffe082';
                            ctx.fillRect(x, y, TILE_SIZE, TILE_SIZE);
                        } else if (tile === 4) {
                            ctx.fillStyle = '#29b6f6';
                            ctx.fillRect(x, y, TILE_SIZE, TILE_SIZE);
                            ctx.fillStyle = '#8d6e63';
                            ctx.fillRect(x, y + 10, TILE_SIZE, TILE_SIZE - 20);
                        }
                    }
                }
            }

            for (let r = startRow; r <= endRow; r++) {
                for (let c = startCol; c <= endCol; c++) {
                    if (r >= 0 && r < MAP_ROWS && c >= 0 && c < MAP_COLS) {
                        const tile = mapData[r][c];
                        const x = c * TILE_SIZE;
                        const y = r * TILE_SIZE;
                        if (tile === 1) drawTree(x, y);
                        else if (tile === 5) drawHouseWall(x, y);
                    }
                }
            }

            for (let chest of chests) {
                drawChest(chest.col * TILE_SIZE, chest.row * TILE_SIZE, chest.solved);
            }
        }

        function drawPlayer() {
            ctx.save();
            const drawX = player.x;
            const drawY = player.y + (Math.sin(player.bobbing) * 3);
            ctx.translate(drawX, drawY);

            ctx.fillStyle = 'rgba(0,0,0,0.4)';
            ctx.beginPath();
            ctx.ellipse(0, 15, 12, 5, 0, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = '#c0392b';
            ctx.beginPath();
            ctx.moveTo(-10, -5);
            ctx.lineTo(10, -5);
            ctx.lineTo(14, 12);
            ctx.lineTo(-14, 12);
            ctx.fill();

            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.arc(0, 0, player.radius, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = '#f1c40f';
            ctx.beginPath();
            ctx.arc(0, -10, 10, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.fillStyle = '#ffeaa7';
            ctx.beginPath();
            ctx.arc(0, -8, 7, 0, Math.PI * 2);
            ctx.fill();

            if (nearChest && !nearChest.solved) {
                const indY = -35 + Math.sin(time * 5) * 5;
                ctx.fillStyle = 'rgba(255, 255, 255, 0.9)';
                ctx.beginPath();
                ctx.arc(-10, indY - 15, 5, Math.PI, Math.PI*1.5);
                ctx.arc(10, indY - 15, 5, Math.PI*1.5, Math.PI*2);
                ctx.arc(10, indY - 5, 5, 0, Math.PI*0.5);
                ctx.arc(-10, indY - 5, 5, Math.PI*0.5, Math.PI);
                ctx.fill();

                ctx.fillStyle = '#d35400';
                ctx.font = 'bold 12px Inter';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                const isMobile = window.innerWidth <= 768;
                ctx.fillText(isMobile ? '!' : 'E', 0, indY - 10);
            }
            ctx.restore();
        }

        function gameLoop() {
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            updatePlayer();
            updateCamera();
            time += 0.05;

            ctx.save();
            ctx.translate(-Math.floor(camera.x), -Math.floor(camera.y));
            drawMap();
            drawPlayer();
            ctx.restore();

            requestAnimationFrame(gameLoop);
        }

        window.onload = function() {
            gameLoop();
        };
    </script>
</body>
</html>
