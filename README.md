# Crazy-flappybird-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Flappy Bird</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
            touch-action: manipulation;
            overflow: hidden;
        }
        
        #game-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            height: 600px;
            overflow: hidden;
            border: 2px solid #000;
        }
        
        #background {
            position: absolute;
            width: 100%;
            height: 100%;
            background-image: url('https://iili.io/2mORBt9.md.jpg');
            background-size: cover;
            z-index: 1;
        }
        
        #bird {
            position: absolute;
            width: 40px;
            height: 40px;
            background-image: url('https://iili.io/2mONmdl.md.png');
            background-size: contain;
            background-repeat: no-repeat;
            z-index: 10;
        }
        
        .pipe {
            position: absolute;
            width: 60px;
            background-image: url('https://i.postimg.cc/pdBBVgZs/Pipe.png');
            background-size: 100% 100%;
            background-repeat: no-repeat;
            z-index: 5;
        }
        
        .pipe-top {
            top: 0;
            transform: rotate(180deg);
        }
        
        .pipe-bottom {
            bottom: 0;
        }
        
        #score-container {
            position: absolute;
            top: 20px;
            width: 100%;
            text-align: center;
            font-size: 24px;
            font-weight: bold;
            color: white;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            z-index: 20;
        }
        
        #game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            display: none;
            z-index: 30;
        }
        
        #start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 40;
        }
        
        .btn {
            margin-top: 15px;
            padding: 10px 20px;
            background-color: #5cb85c;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        
        .btn:hover {
            background-color: #4cae4c;
        }
        
        #ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 20px;
            background-color: #8b4513;
            z-index: 15;
        }
        
        h1 {
            color: white;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div id="background"></div>
        <div id="bird"></div>
        <div id="score-container">Score: <span id="score">0</span></div>
        <div id="game-over">
            <h2>Game Over!</h2>
            <p>Your score: <span id="final-score">0</span></p>
            <button id="restart-btn" class="btn">Play Again</button>
        </div>
        <div id="start-screen">
            <h1>Flappy Bird</h1>
            <button id="start-btn" class="btn">Start Game</button>
        </div>
        <div id="ground"></div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const gameContainer = document.getElementById('game-container');
            const bird = document.getElementById('bird');
            const scoreElement = document.getElementById('score');
            const finalScoreElement = document.getElementById('final-score');
            const gameOverElement = document.getElementById('game-over');
            const restartBtn = document.getElementById('restart-btn');
            const startScreen = document.getElementById('start-screen');
            const startBtn = document.getElementById('start-btn');
            const ground = document.getElementById('ground');
            
            // Game variables
            let gameWidth = gameContainer.offsetWidth;
            let gameHeight = gameContainer.offsetHeight;
            let birdLeft = 50;
            let birdTop = 250;
            let gravity = 0.5;
            let velocity = 0;
            let jumpForce = -10;
            let gameIsOver = false;
            let gameIsRunning = false;
            let score = 0;
            let pipes = [];
            let gameLoopId;
            let pipeGenerationInterval;
            let lastPipeTime = 0;
            let pipeGap = 150;
            let pipeFrequency = 2000;
            
            // Initialize game
            function initGame() {
                // Reset game state
                gameIsOver = false;
                gameIsRunning = true;
                score = 0;
                scoreElement.textContent = score;
                birdTop = gameHeight / 2 - 20;
                velocity = 0;
                
                // Clear existing pipes
                pipes.forEach(pipe => {
                    if (pipe.element && pipe.element.parentNode) {
                        gameContainer.removeChild(pipe.element);
                    }
                });
                pipes = [];
                
                // Reset bird position
                bird.style.left = birdLeft + 'px';
                bird.style.top = birdTop + 'px';
                
                // Hide screens
                gameOverElement.style.display = 'none';
                startScreen.style.display = 'none';
                
                // Start game loop
                lastPipeTime = performance.now();
                gameLoopId = requestAnimationFrame(gameLoop);
                pipeGenerationInterval = setInterval(generatePipes, pipeFrequency);
            }
            
            // Game loop
            function gameLoop(timestamp) {
                if (!gameIsRunning) {
                    cancelAnimationFrame(gameLoopId);
                    clearInterval(pipeGenerationInterval);
                    return;
                }
                
                // Apply gravity
                velocity += gravity;
                birdTop += velocity;
                bird.style.top = birdTop + 'px';
                
                // Check for collisions with ground or ceiling
                if (birdTop >= gameHeight - ground.offsetHeight - bird.offsetHeight) {
                    birdTop = gameHeight - ground.offsetHeight - bird.offsetHeight;
                    endGame();
                }
                
                if (birdTop <= 0) {
                    birdTop = 0;
                    endGame();
                }
                
                // Move pipes and check for collisions
                movePipes();
                checkCollisions();
                
                // Continue the game loop
                gameLoopId = requestAnimationFrame(gameLoop);
            }
            
            // Generate pipes
            function generatePipes() {
                if (!gameIsRunning || gameIsOver) return;
                
                const minTopHeight = 50;
                const maxTopHeight = gameHeight - pipeGap - minTopHeight - ground.offsetHeight;
                const topHeight = Math.floor(Math.random() * (maxTopHeight - minTopHeight + 1)) + minTopHeight;
                
                // Create top pipe
                const topPipe = document.createElement('div');
                topPipe.className = 'pipe pipe-top';
                topPipe.style.height = topHeight + 'px';
                topPipe.style.left = gameWidth + 'px';
                gameContainer.appendChild(topPipe);
                
                // Create bottom pipe
                const bottomPipe = document.createElement('div');
                bottomPipe.className = 'pipe pipe-bottom';
                bottomPipe.style.height = (gameHeight - topHeight - pipeGap - ground.offsetHeight) + 'px';
                bottomPipe.style.left = gameWidth + 'px';
                gameContainer.appendChild(bottomPipe);
                
                // Add pipes to array
                pipes.push({
                    element: topPipe,
                    x: gameWidth,
                    height: topHeight,
                    passed: false,
                    counted: false
                });
                
                pipes.push({
                    element: bottomPipe,
                    x: gameWidth,
                    height: gameHeight - topHeight - pipeGap - ground.offsetHeight,
                    passed: false,
                    counted: false
                });
            }
            
            // Move pipes
            function movePipes() {
                for (let i = pipes.length - 1; i >= 0; i--) {
                    const pipe = pipes[i];
                    pipe.x -= 2;
                    pipe.element.style.left = pipe.x + 'px';
                    
                    // Remove pipes that are off screen
                    if (pipe.x < -pipe.element.offsetWidth) {
                        gameContainer.removeChild(pipe.element);
                        pipes.splice(i, 1);
                    }
                    
                    // Check if bird passed a pipe pair
                    if (!pipe.counted && pipe.x + pipe.element.offsetWidth < birdLeft) {
                        pipe.counted = true;
                        // Only count for top pipe to avoid double counting
                        if (pipe.element.classList.contains('pipe-top')) {
                            score++;
                            scoreElement.textContent = score;
                            // Increase difficulty slightly as score increases
                            if (score % 5 === 0 && pipeFrequency > 1200) {
                                pipeFrequency -= 100;
                                clearInterval(pipeGenerationInterval);
                                pipeGenerationInterval = setInterval(generatePipes, pipeFrequency);
                            }
                        }
                    }
                }
            }
            
            // Check for collisions
            function checkCollisions() {
                const birdRect = {
                    left: birdLeft,
                    right: birdLeft + bird.offsetWidth,
                    top: birdTop,
                    bottom: birdTop + bird.offsetHeight
                };
                
                for (const pipe of pipes) {
                    const pipeRect = {
                        left: pipe.x,
                        right: pipe.x + pipe.element.offsetWidth,
                        top: pipe.element.classList.contains('pipe-top') ? 0 : gameHeight - pipe.height - ground.offsetHeight,
                        bottom: pipe.element.classList.contains('pipe-top') ? pipe.height : gameHeight - ground.offsetHeight
                    };
                    
                    if (
                        birdRect.right > pipeRect.left &&
                        birdRect.left < pipeRect.right &&
                        birdRect.bottom > pipeRect.top &&
                        birdRect.top < pipeRect.bottom
                    ) {
                        endGame();
                        break;
                    }
                }
            }
            
            // End game
            function endGame() {
                gameIsOver = true;
                gameIsRunning = false;
                finalScoreElement.textContent = score;
                gameOverElement.style.display = 'block';
            }
            
            // Jump function
            function jump() {
                if (!gameIsRunning) return;
                velocity = jumpForce;
            }
            
            // Event listeners
            document.addEventListener('keydown', (e) => {
                if ((e.code === 'Space' || e.code === 'ArrowUp') && gameIsRunning) {
                    e.preventDefault();
                    jump();
                }
                
                // Start game with space if not running
                if (e.code === 'Space' && !gameIsRunning && gameIsOver) {
                    initGame();
                }
            });
            
            // Touch support for mobile
            gameContainer.addEventListener('touchstart', (e) => {
                if (gameIsRunning) {
                    e.preventDefault();
                    jump();
                } else if (!gameIsRunning && gameIsOver) {
                    initGame();
                }
            });
            
            // Click support for desktop
            gameContainer.addEventListener('click', () => {
                if (!gameIsRunning && !gameIsOver) {
                    // This is handled by the start button
                    return;
                }
                
                if (gameIsRunning) {
                    jump();
                }
            });
            
            restartBtn.addEventListener('click', initGame);
            startBtn.addEventListener('click', initGame);
            
            // Handle window resize
            window.addEventListener('resize', () => {
                gameWidth = gameContainer.offsetWidth;
                gameHeight = gameContainer.offsetHeight;
            });
            
            // Initial setup
            bird.style.left = birdLeft + 'px';
            bird.style.top = birdTop + 'px';
        });
    </script>
</body>
</html>
