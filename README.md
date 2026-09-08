!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dynamic Tic Tac Toe</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            min-height: 100vh;
            font-family: Arial, sans-serif;
            background: #1e1e24;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .game {
            text-align: center;
            width: 95%;
            position: relative;
        }

        h1 {
            margin-bottom: 15px;
        }

        .settings {
            margin-bottom: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-wrap: wrap;
            gap: 10px;
        }

        .settings label {
            font-size: 16px;
        }

        select {
            padding: 8px 12px;
            font-size: 16px;
            border-radius: 6px;
            border: none;
            background: #fff;
            cursor: pointer;
        }

        #status {
            font-size: 22px;
            font-weight: bold;
            margin: 15px;
            min-height: 28px;
        }

        /* Container wrapping board and strike line */
        .board-wrapper {
            position: relative;
            display: inline-block;
            margin: auto;
        }

        .board {
            display: grid;
            gap: 6px;
            justify-content: center;
            margin: auto;
        }

        .cell {
            width: 70px;
            height: 70px;
            background: #2b2d42;
            border: 2px solid #4a4e69;
            border-radius: 8px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 35px;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s, transform 0.1s;
        }

        .cell:hover {
            background: #3d405b;
        }

        .x {
            color: #00d2d3;
            text-shadow: 0 0 10px rgba(0, 210, 211, 0.4);
        }

        .o {
            color: #ff6b6b;
            text-shadow: 0 0 10px rgba(255, 107, 107, 0.4);
        }

        /* Highlight winner cells */
        .cell.win-cell {
            background: #383e56;
            transform: scale(1.03);
            border-color: #ffd166;
        }

        /* SVG Strike Line Layer */
        #strike-svg {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 10;
        }

        #win-line {
            stroke: #ffd166;
            stroke-width: 8;
            stroke-linecap: round;
            filter: drop-shadow(0 0 8px #ffd166);
            stroke-dasharray: 1000;
            stroke-dashoffset: 1000;
            animation: drawLine 0.4s ease-out forwards;
        }

        @keyframes drawLine {
            to {
                stroke-dashoffset: 0;
            }
        }

        button {
            margin-top: 15px;
            padding: 10px 22px;
            font-size: 17px;
            font-weight: bold;
            border: none;
            border-radius: 6px;
            background: #00d2d3;
            color: #1e1e24;
            cursor: pointer;
            transition: 0.2s;
        }

        button:hover {
            background: #01a3a4;
            color: #fff;
        }

        /* Winner Popup Modal */
        .modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: rgba(0, 0, 0, 0.7);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 100;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
        }

        .modal.show {
            opacity: 1;
            pointer-events: auto;
        }

        .modal-content {
            background: #2b2d42;
            padding: 30px 40px;
            border-radius: 12px;
            border: 2px solid #ffd166;
            text-align: center;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
        }

        .modal-content h2 {
            margin-top: 0;
            font-size: 30px;
        }

        @media (max-width: 500px) {
            .cell {
                width: 50px;
                height: 50px;
                font-size: 25px;
            }
            #win-line {
                stroke-width: 6;
            }
        }
    </style>
</head>

<body>

<div class="game">

    <h1>🎮 Dynamic Tic Tac Toe</h1>

    <div class="settings">
        <!-- Mode Option Added -->
        <label>
            Mode:
            <select id="gameMode">
                <option value="pvp">👥 vs Friend</option>
                <option value="pve">🤖 vs Computer</option>
            </select>
        </label>

        <label>
            Rows:
            <select id="rows">
                <option value="3">3</option>
                <option value="4">4</option>
                <option value="5">5</option>
                <option value="6">6</option>
                <option value="7">7</option>
            </select>
        </label>

        <label>
            Columns:
            <select id="columns">
                <option value="3">3</option>
                <option value="4">4</option>
                <option value="5">5</option>
                <option value="6">6</option>
                <option value="7">7</option>
            </select>
        </label>

        <button onclick="startGame()">Start Game</button>
    </div>

    <div id="status">Select mode, size, and press Start Game</div>

    <!-- Board Wrapper with SVG overlay -->
    <div class="board-wrapper">
        <div id="board" class="board"></div>
        <svg id="strike-svg">
            <line id="win-line" x1="0" y1="0" x2="0" y2="0" style="display:none;" />
        </svg>
    </div>

    <br>
    <button onclick="restartGame()">Restart Game</button>

</div>

<!-- Win/Draw Popup Modal -->
<div id="winnerModal" class="modal">
    <div class="modal-content">
        <h2 id="modalText">🎉 Player X Wins!</h2>
        <button onclick="closeModalAndRestart()">Play Again</button>
    </div>
</div>

<script>
    let rows = 3;
    let columns = 3;
    let currentPlayer = "X";
    let gameActive = false;
    let board = [];
    let gameMode = "pvp"; // "pvp" (Friend) or "pve" (Computer)

    function startGame() {
        gameMode = document.getElementById("gameMode").value;
        rows = parseInt(document.getElementById("rows").value);
        columns = parseInt(document.getElementById("columns").value);

        currentPlayer = "X";
        gameActive = true;

        clearStrikeLine();
        createBoard();

        updateStatusText();
    }

    function createBoard() {
        const boardElement = document.getElementById("board");
        boardElement.innerHTML = "";
        board = [];

        boardElement.style.gridTemplateColumns = `repeat(${columns}, 70px)`;

        if (window.innerWidth <= 500) {
            boardElement.style.gridTemplateColumns = `repeat(${columns}, 50px)`;
        }

        for (let i = 0; i < rows * columns; i++) {
            board.push("");
            const cell = document.createElement("div");
            cell.classList.add("cell");
            cell.dataset.index = i;
            cell.addEventListener("click", handleCellClick);
            boardElement.appendChild(cell);
        }
    }

    function handleCellClick(event) {
        if (!gameActive) return;

        // Prevent clicking during computer's turn
        if (gameMode === "pve" && currentPlayer === "O") return;

        const cell = event.target;
        const index = parseInt(cell.dataset.index);

        if (board[index] !== "") return;

        makeMove(index, cell);

        // If playing against computer and game is still active, trigger computer move
        if (gameActive && gameMode === "pve" && currentPlayer === "O") {
            setTimeout(computerMove, 400);
        }
    }

    function makeMove(index, cell) {
        board[index] = currentPlayer;
        cell.textContent = currentPlayer;
        cell.classList.add(currentPlayer === "X" ? "x" : "o");

        const result = checkWinner();
        if (result) return;

        // Switch turns
        currentPlayer = currentPlayer === "X" ? "O" : "X";
        updateStatusText();
    }

    // AI logic: picks winning move, blocks opponent, or chooses random spot
    function computerMove() {
        if (!gameActive) return;

        let emptyIndices = [];
        for (let i = 0; i < board.length; i++) {
            if (board[i] === "") emptyIndices.push(i);
        }

        if (emptyIndices.length === 0) return;

        // 1. Can computer win on this turn?
        let move = findWinningSpot("O");

        // 2. Can player win on next turn? Block them!
        if (move === null) {
            move = findWinningSpot("X");
        }

        // 3. Otherwise pick random open cell
        if (move === null) {
            const randomIndex = Math.floor(Math.random() * emptyIndices.length);
            move = emptyIndices[randomIndex];
        }

        const cells = document.querySelectorAll(".cell");
        makeMove(move, cells[move]);
    }

    // Simulated helper to check if a spot secures a line
    function findWinningSpot(player) {
        for (let i = 0; i < board.length; i++) {
            if (board[i] === "") {
                board[i] = player;
                const win = simulateWin(player);
                board[i] = ""; // backtrack
                if (win) return i;
            }
        }
        return null;
    }

    function simulateWin(player) {
        // Rows
        for (let r = 0; r < rows; r++) {
            let win = true;
            for (let c = 0; c < columns; c++) {
                if (board[r * columns + c] !== player) { win = false; break; }
            }
            if (win) return true;
        }

        // Columns
        for (let c = 0; c < columns; c++) {
            let win = true;
            for (let r = 0; r < rows; r++) {
                if (board[r * columns + c] !== player) { win = false; break; }
            }
            if (win) return true;
        }

        // Diagonals
        if (rows === columns) {
            let win1 = true;
            for (let i = 0; i < rows; i++) {
                if (board[i * columns + i] !== player) { win1 = false; break; }
            }
            if (win1) return true;

            let win2 = true;
            for (let i = 0; i < rows; i++) {
                if (board[i * columns + (columns - 1 - i)] !== player) { win2 = false; break; }
            }
            if (win2) return true;
        }

        return false;
    }

    function checkWinner() {
        // 1. Check Rows
        for (let r = 0; r < rows; r++) {
            let first = board[r * columns];
            if (first === "") continue;

            let win = true;
            let indices = [r * columns];

            for (let c = 1; c < columns; c++) {
                let idx = r * columns + c;
                indices.push(idx);
                if (board[idx] !== first) {
                    win = false;
                    break;
                }
            }

            if (win) {
                endGame(first, indices);
                return true;
            }
        }

        // 2. Check Columns
        for (let c = 0; c < columns; c++) {
            let first = board[c];
            if (first === "") continue;

            let win = true;
            let indices = [c];

            for (let r = 1; r < rows; r++) {
                let idx = r * columns + c;
                indices.push(idx);
                if (board[idx] !== first) {
                    win = false;
                    break;
                }
            }

            if (win) {
                endGame(first, indices);
                return true;
            }
        }

        // 3. Check Diagonals (square boards only)
        if (rows === columns) {
            let first = board[0];
            if (first !== "") {
                let win = true;
                let indices = [0];
                for (let i = 1; i < rows; i++) {
                    let idx = i * columns + i;
                    indices.push(idx);
                    if (board[idx] !== first) {
                        win = false;
                        break;
                    }
                }
                if (win) {
                    endGame(first, indices);
                    return true;
                }
            }

            first = board[columns - 1];
            if (first !== "") {
                let win = true;
                let indices = [columns - 1];
                for (let i = 1; i < rows; i++) {
                    let idx = i * columns + (columns - 1 - i);
                    indices.push(idx);
                    if (board[idx] !== first) {
                        win = false;
                        break;
                    }
                }
                if (win) {
                    endGame(first, indices);
                    return true;
                }
            }
        }

        // 4. Check Draw
        if (!board.includes("")) {
            document.getElementById("status").textContent = "🤝 Game Draw!";
            gameActive = false;
            showModal("🤝 It's a Draw!");
            return true;
        }

        return false;
    }

    function updateStatusText() {
        if (gameMode === "pve") {
            document.getElementById("status").textContent =
                currentPlayer === "X" ? "Your Turn (X)" : "🤖 Computer is thinking...";
        } else {
            document.getElementById("status").textContent = `Player ${currentPlayer}'s Turn`;
        }
    }

    function endGame(winner, winningIndices) {
        gameActive = false;
        
        let label = winner;
        if (gameMode === "pve") {
            label = winner === "X" ? "You" : "Computer";
        } else {
            label = `Player ${winner}`;
        }

        document.getElementById("status").textContent = `🎉 ${label} Won!`;

        drawStrikeLine(winningIndices);

        const cells = document.querySelectorAll(".cell");
        winningIndices.forEach(idx => cells[idx].classList.add("win-cell"));

        setTimeout(() => {
            showModal(`🎉 ${label} Won!`);
        }, 500);
    }

    function drawStrikeLine(indices) {
        const boardWrapper = document.querySelector(".board-wrapper");
        const cells = document.querySelectorAll(".cell");
        const firstCell = cells[indices[0]];
        const lastCell = cells[indices[indices.length - 1]];

        const wrapperRect = boardWrapper.getBoundingClientRect();
        const startRect = firstCell.getBoundingClientRect();
        const endRect = lastCell.getBoundingClientRect();

        const x1 = (startRect.left + startRect.width / 2) - wrapperRect.left;
        const y1 = (startRect.top + startRect.height / 2) - wrapperRect.top;
        const x2 = (endRect.left + endRect.width / 2) - wrapperRect.left;
        const y2 = (endRect.top + endRect.height / 2) - wrapperRect.top;

        const line = document.getElementById("win-line");
        line.setAttribute("x1", x1);
        line.setAttribute("y1", y1);
        line.setAttribute("x2", x2);
        line.setAttribute("y2", y2);

        line.style.display = "block";
        line.style.animation = "none";
        line.offsetHeight;
        line.style.animation = "drawLine 0.4s ease-out forwards";
    }

    function clearStrikeLine() {
        const line = document.getElementById("win-line");
        line.style.display = "none";
    }

    function showModal(text) {
        document.getElementById("modalText").textContent = text;
        document.getElementById("winnerModal").classList.add("show");
    }

    function closeModalAndRestart() {
        document.getElementById("winnerModal").classList.remove("show");
        restartGame();
    }

    function restartGame() {
        gameMode = document.getElementById("gameMode").value;
        clearStrikeLine();
        currentPlayer = "X";
        gameActive = true;
        createBoard();
        updateStatusText();
    }
</script>

</body>
</html>
