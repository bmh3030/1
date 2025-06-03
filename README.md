<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>2048小游戏</title>
    <style>
        body {
            background: #faf8ef;
            font-family: "微软雅黑", Arial, sans-serif;
            text-align: center;
        }
        #game-container {
            width: 400px;
            margin: 40px auto;
            background: #bbada0;
            border-radius: 6px;
            padding: 20px;
            box-shadow: 0 0 10px #aaa;
        }
        #score {
            font-size: 24px;
            margin-bottom: 10px;
            color: #776e65;
        }
        #board {
            width: 400px;
            height: 400px;
            background: #bbada0;
            border-radius: 6px;
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            grid-template-rows: repeat(4, 1fr);
            gap: 10px;
        }
        .cell {
            width: 90px;
            height: 90px;
            background: #cdc1b4;
            border-radius: 3px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 32px;
            font-weight: bold;
            color: #776e65;
            transition: background 0.2s, color 0.2s;
        }
        .cell-2    { background: #eee4da; color: #776e65; }
        .cell-4    { background: #ede0c8; color: #776e65; }
        .cell-8    { background: #f2b179; color: #f9f6f2; }
        .cell-16   { background: #f59563; color: #f9f6f2; }
        .cell-32   { background: #f67c5f; color: #f9f6f2; }
        .cell-64   { background: #f65e3b; color: #f9f6f2; }
        .cell-128  { background: #edcf72; color: #f9f6f2; }
        .cell-256  { background: #edcc61; color: #f9f6f2; }
        .cell-512  { background: #edc850; color: #f9f6f2; }
        .cell-1024 { background: #edc53f; color: #f9f6f2; }
        .cell-2048 { background: #edc22e; color: #f9f6f2; }
        #game-over {
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(238,228,218,0.7);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 48px;
            color: #776e65;
            z-index: 10;
            border-radius: 6px;
            display: none;
        }
        #restart-btn {
            margin-top: 15px;
            padding: 8px 24px;
            font-size: 18px;
            background: #8f7a66;
            color: #fff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        #restart-btn:hover {
            background: #a39489;
        }
        #container-wrap {
            position: relative;
            display: inline-block;
        }
    </style>
</head>
<body>
    <h1>2048小游戏</h1>
    <div id="game-container">
        <div id="score">分数: <span id="score-value">0</span></div>
        <div id="container-wrap">
            <div id="board"></div>
            <div id="game-over">游戏结束！</div>
        </div>
        <button id="restart-btn">重新开始</button>
    </div>
    <script>
        const size = 4;
        let board = [];
        let score = 0;
        let gameOver = false;

        function initBoard() {
            board = [];
            for (let i = 0; i < size; i++) {
                board.push(Array(size).fill(0));
            }
            score = 0;
            gameOver = false;
            document.getElementById('game-over').style.display = 'none';
            addRandomTile();
            addRandomTile();
            updateBoard();
            updateScore();
        }

        function addRandomTile() {
            let empty = [];
            for (let i = 0; i < size; i++) {
                for (let j = 0; j < size; j++) {
                    if (board[i][j] === 0) empty.push([i, j]);
                }
            }
            if (empty.length === 0) return;
            let [x, y] = empty[Math.floor(Math.random() * empty.length)];
            board[x][y] = Math.random() < 0.9 ? 2 : 4;
        }

        function updateBoard() {
            const boardDiv = document.getElementById('board');
            boardDiv.innerHTML = '';
            for (let i = 0; i < size; i++) {
                for (let j = 0; j < size; j++) {
                    let cell = document.createElement('div');
                    cell.className = 'cell';
                    if (board[i][j] !== 0) {
                        cell.classList.add('cell-' + board[i][j]);
                        cell.textContent = board[i][j];
                    } else {
                        cell.textContent = '';
                    }
                    boardDiv.appendChild(cell);
                }
            }
        }

        function updateScore() {
            document.getElementById('score-value').textContent = score;
        }

        function move(direction) {
            if (gameOver) return;
            let moved = false;
            let merged = Array(size).fill().map(() => Array(size).fill(false));
            let before = JSON.stringify(board);

            function moveLine(line) {
                let arr = line.filter(x => x !== 0);
                for (let i = 0; i < arr.length - 1; i++) {
                    if (arr[i] === arr[i + 1]) {
                        arr[i] *= 2;
                        score += arr[i];
                        arr[i + 1] = 0;
                    }
                }
                arr = arr.filter(x => x !== 0);
                while (arr.length < size) arr.push(0);
                return arr;
            }

            if (direction === 'left') {
                for (let i = 0; i < size; i++) {
                    let newLine = moveLine(board[i]);
                    if (board[i].toString() !== newLine.toString()) moved = true;
                    board[i] = newLine;
                }
            } else if (direction === 'right') {
                for (let i = 0; i < size; i++) {
                    let reversed = board[i].slice().reverse();
                    let newLine = moveLine(reversed).reverse();
                    if (board[i].toString() !== newLine.toString()) moved = true;
                    board[i] = newLine;
                }
            } else if (direction === 'up') {
                for (let j = 0; j < size; j++) {
                    let col = [];
                    for (let i = 0; i < size; i++) col.push(board[i][j]);
                    let newCol = moveLine(col);
                    for (let i = 0; i < size; i++) {
                        if (board[i][j] !== newCol[i]) moved = true;
                        board[i][j] = newCol[i];
                    }
                }
            } else if (direction === 'down') {
                for (let j = 0; j < size; j++) {
                    let col = [];
                    for (let i = 0; i < size; i++) col.push(board[i][j]);
                    let newCol = moveLine(col.reverse()).reverse();
                    for (let i = 0; i < size; i++) {
                        if (board[i][j] !== newCol[i]) moved = true;
                        board[i][j] = newCol[i];
                    }
                }
            }

            if (moved) {
                addRandomTile();
                updateBoard();
                updateScore();
                if (isGameOver()) {
                    document.getElementById('game-over').style.display = 'flex';
                    gameOver = true;
                }
            }
        }

        function isGameOver() {
            for (let i = 0; i < size; i++) {
                for (let j = 0; j < size; j++) {
                    if (board[i][j] === 0) return false;
                    if (i < size - 1 && board[i][j] === board[i + 1][j]) return false;
                    if (j < size - 1 && board[i][j] === board[i][j + 1]) return false;
                }
            }
            return true;
        }

        document.addEventListener('keydown', function(e) {
            if (gameOver) return;
            switch (e.key) {
                case 'ArrowLeft':
                case 'a':
                case 'A':
                    move('left');
                    break;
                case 'ArrowRight':
                case 'd':
                case 'D':
                    move('right');
                    break;
                case 'ArrowUp':
                case 'w':
                case 'W':
                    move('up');
                    break;
                case 'ArrowDown':
                case 's':
                case 'S':
                    move('down');
                    break;
            }
        });

        document.getElementById('restart-btn').onclick = function() {
            initBoard();
        };

        // 触摸支持
        let startX, startY;
        document.getElementById('board').addEventListener('touchstart', function(e) {
            if (e.touches.length === 1) {
                startX = e.touches[0].clientX;
                startY = e.touches[0].clientY;
            }
        });
        document.getElementById('board').addEventListener('touchend', function(e) {
            if (gameOver) return;
            if (e.changedTouches.length === 1) {
                let dx = e.changedTouches[0].clientX - startX;
                let dy = e.changedTouches[0].clientY - startY;
                if (Math.abs(dx) > Math.abs(dy)) {
                    if (dx > 30) move('right');
                    else if (dx < -30) move('left');
                } else {
                    if (dy > 30) move('down');
                    else if (dy < -30) move('up');
                }
            }
        });

        initBoard();
    </script>
</body>
</html>
