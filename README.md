<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Bounce Tales Mini</title>
    <link rel="manifest" href="./manifest.json">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            touch-action: manipulation;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #1a237e, #311b92);
            color: white;
            overflow: hidden;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 10px;
        }
        
        #game-container {
            position: relative;
            width: 100%;
            max-width: 500px;
            height: 80vh;
            margin: 0 auto;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            background: #0a0e29;
        }
        
        #game-canvas {
            background: #0a0e29;
            display: block;
            width: 100%;
            height: 100%;
        }
        
        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background: rgba(10, 14, 41, 0.95);
            z-index: 10;
            padding: 20px;
            text-align: center;
        }
        
        h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            color: #4fc3f7;
            text-shadow: 0 2px 10px rgba(79, 195, 247, 0.5);
        }
        
        h2 {
            font-size: 1.8rem;
            margin-bottom: 20px;
            color: #4dabf7;
        }
        
        p {
            font-size: 1.1rem;
            margin-bottom: 15px;
            max-width: 80%;
            line-height: 1.5;
        }
        
        .btn {
            background: linear-gradient(to right, #4dabf7, #3a7bd5);
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 1.2rem;
            border-radius: 50px;
            cursor: pointer;
            margin: 10px;
            transition: all 0.3s;
            box-shadow: 0 4px 15px rgba(58, 123, 213, 0.4);
        }
        
        .btn:hover {
            transform: translateY(-3px);
            box-shadow: 0 6px 20px rgba(58, 123, 213, 0.6);
        }
        
        .btn:active {
            transform: translateY(1px);
        }
        
        #score-display {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 1.2rem;
            background: rgba(0, 0, 0, 0.5);
            padding: 8px 15px;
            border-radius: 20px;
            z-index: 5;
        }
        
        #level-display {
            position: absolute;
            top: 20px;
            right: 20px;
            font-size: 1.2rem;
            background: rgba(0, 0, 0, 0.5);
            padding: 8px 15px;
            border-radius: 20px;
            z-index: 5;
        }
        
        .hidden {
            display: none !important;
        }
        
        .controls-info {
            position: absolute;
            bottom: 20px;
            width: 100%;
            text-align: center;
            font-size: 0.9rem;
            color: rgba(255, 255, 255, 0.7);
            padding: 0 20px;
        }
        
        @media (max-width: 768px) {
            #game-container {
                height: 70vh;
            }
            
            h1 {
                font-size: 2rem;
            }
            
            .controls-info {
                font-size: 0.8rem;
            }
        }
        
        @media (max-height: 600px) {
            #game-container {
                height: 90vh;
            }
        }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="game-canvas"></canvas>
        
        <div id="score-display">Score: 0</div>
        <div id="level-display">Level: 1</div>
        
        <div id="start-screen" class="screen">
            <h1>Bounce Tales</h1>
            <p>Tap to bounce and steer the ball to the goal. Avoid spikes.</p>
            <p>Collect rings for points!</p>
            <button id="start-btn" class="btn">Tap to Start</button>
            <div class="controls-info">
                Controls: Tap left/right half to move. Keyboard: ←/→ or A/D. R to restart.
            </div>
        </div>
        
        <div id="game-over-screen" class="screen hidden">
            <h2>Game Over</h2>
            <p id="final-score">Your score: 0</p>
            <button id="restart-btn" class="btn">Play Again</button>
        </div>
        
        <div id="level-complete-screen" class="screen hidden">
            <h2>Level Complete!</h2>
            <p id="level-score">Score this level: 0</p>
            <button id="next-level-btn" class="btn">Next Level</button>
        </div>
        
        <div id="win-screen" class="screen hidden">
            <h2>Congratulations!</h2>
            <p>You've completed all levels!</p>
            <p id="total-score">Total Score: 0</p>
            <button id="win-restart-btn" class="btn">Play Again</button>
        </div>
    </div>

    <script>
        // Game variables
        let canvas, ctx;
        let ball = {
            x: 50,
            y: 300,
            radius: 15,
            speedX: 0,
            speedY: 0,
            color: '#4fc3f7'
        };
        
        let gravity = 0.5;
        let friction = 0.8;
        let isJumping = false;
        
        let platforms = [];
        let obstacles = [];
        let rings = [];
        let goal = {};
        
        let score = 0;
        let level = 1;
        let gameRunning = false;
        let keys = {};
        
        // DOM elements
        const startScreen = document.getElementById('start-screen');
        const gameOverScreen = document.getElementById('game-over-screen');
        const levelCompleteScreen = document.getElementById('level-complete-screen');
        const winScreen = document.getElementById('win-screen');
        const scoreDisplay = document.getElementById('score-display');
        const levelDisplay = document.getElementById('level-display');
        const finalScore = document.getElementById('final-score');
        const levelScore = document.getElementById('level-score');
        const totalScore = document.getElementById('total-score');
        
        // Initialize the game
        function init() {
            canvas = document.getElementById('game-canvas');
            ctx = canvas.getContext('2d');
            
            // Set canvas size
            canvas.width = canvas.offsetWidth;
            canvas.height = canvas.offsetHeight;
            
            // Set up event listeners
            document.getElementById('start-btn').addEventListener('click', startGame);
            document.getElementById('restart-btn').addEventListener('click', restartGame);
            document.getElementById('next-level-btn').addEventListener('click', nextLevel);
            document.getElementById('win-restart-btn').addEventListener('click', restartGame);
            
            // Touch controls for left/right sides of screen
            canvas.addEventListener('touchstart', handleTouch);
            
            // Keyboard controls
            window.addEventListener('keydown', (e) => {
                keys[e.key] = true;
                
                // R key to restart
                if (e.key === 'r' || e.key === 'R') {
                    restartGame();
                }
            });
            
            window.addEventListener('keyup', (e) => {
                keys[e.key] = false;
            });
            
            // Handle window resize
            window.addEventListener('resize', () => {
                canvas.width = canvas.offsetWidth;
                canvas.height = canvas.offsetHeight;
                if (gameRunning) setupLevel();
            });
            
            // Show start screen
            showStartScreen();
        }
        
        // Handle touch input for left/right controls
        function handleTouch(e) {
            if (!gameRunning) return;
            
            e.preventDefault();
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            const touchX = touch.clientX - rect.left;
            
            // If touch is on left half of screen, move left
            if (touchX < canvas.width / 2) {
                keys['ArrowLeft'] = true;
                setTimeout(() => { keys['ArrowLeft'] = false; }, 100);
            } 
            // If touch is on right half of screen, move right
            else {
                keys['ArrowRight'] = true;
                setTimeout(() => { keys['ArrowRight'] = false; }, 100);
            }
        }
        
        // Show start screen
        function showStartScreen() {
            startScreen.classList.remove('hidden');
            gameOverScreen.classList.add('hidden');
            levelCompleteScreen.classList.add('hidden');
            winScreen.classList.add('hidden');
            gameRunning = false;
        }
        
        // Start the game
        function startGame() {
            startScreen.classList.add('hidden');
            score = 0;
            level = 1;
            updateDisplays();
            setupLevel();
            gameRunning = true;
            gameLoop();
        }
        
        // Restart the game
        function restartGame() {
            gameOverScreen.classList.add('hidden');
            winScreen.classList.add('hidden');
            score = 0;
            level = 1;
            updateDisplays();
            setupLevel();
            gameRunning = true;
            gameLoop();
        }
        
        // Go to next level
        function nextLevel() {
            levelCompleteScreen.classList.add('hidden');
            level++;
            if (level > 3) {
                showWinScreen();
                return;
            }
            updateDisplays();
            setupLevel();
            gameRunning = true;
            gameLoop();
        }
        
        // Show win screen
        function showWinScreen() {
            winScreen.classList.remove('hidden');
            totalScore.textContent = `Total Score: ${score}`;
            gameRunning = false;
        }
        
        // Set up the current level
        function setupLevel() {
            // Reset ball
            ball.x = 50;
            ball.y = canvas.height - 100;
            ball.speedX = 0;
            ball.speedY = 0;
            
            // Clear arrays
            platforms = [];
            obstacles = [];
            rings = [];
            
            // Create floor platform
            platforms.push({x: 0, y: canvas.height - 30, width: canvas.width, height: 30});
            
            // Create level-specific elements
            if (level === 1) {
                // Level 1 - Basic platforms and obstacles
                platforms.push({x: 100, y: 400, width: 150, height: 20});
                platforms.push({x: 300, y: 350, width: 150, height: 20});
                platforms.push({x: 200, y: 250, width: 150, height: 20});
                
                // Add obstacles (spikes) coming from above
                obstacles.push({x: 150, y: 100, width: 40, height: 40, type: 'spike', speedY: 2});
                obstacles.push({x: 350, y: 50, width: 40, height: 40, type: 'spike', speedY: 1.5});
                
                // Add rings
                rings.push({x: 150, y: 370, radius: 12, collected: false});
                rings.push({x: 350, y: 320, radius: 12, collected: false});
                rings.push({x: 250, y: 220, radius: 12, collected: false});
                
                // Add goal
                goal = {x: canvas.width - 80, y: 200, width: 60, height: 60};
            } else if (level === 2) {
                // Level 2 - More challenging
                platforms.push({x: 100, y: 450, width: 100, height: 20});
                platforms.push({x: 250, y: 400, width: 100, height: 20});
                platforms.push({x: 150, y: 300, width: 100, height: 20});
                platforms.push({x: 300, y: 250, width: 100, height: 20});
                platforms.push({x: 200, y: 150, width: 100, height: 20});
                
                // Add obstacles
                obstacles.push({x: 120, y: 80, width: 40, height: 40, type: 'spike', speedY: 2});
                obstacles.push({x: 280, y: 120, width: 40, height: 40, type: 'spike', speedY: 1.8});
                obstacles.push({x: 350, y: 200, width: 40, height: 40, type: 'spike', speedY: 1.5});
                
                // Add rings
                rings.push({x: 140, y: 420, radius: 12, collected: false});
                rings.push({x: 290, y: 370, radius: 12, collected: false});
                rings.push({x: 190, y: 270, radius: 12, collected: false});
                rings.push({x: 340, y: 220, radius: 12, collected: false});
                rings.push({x: 240, y: 120, radius: 12, collected: false});
                
                // Add goal
                goal = {x: canvas.width - 80, y: 120, width: 60, height: 60};
            } else if (level === 3) {
                // Level 3 - Most challenging
                platforms.push({x: 50, y: 450, width: 80, height: 20});
                platforms.push({x: 180, y: 400, width: 80, height: 20});
                platforms.push({x: 100, y: 350, width: 80, height: 20});
                platforms.push({x: 250, y: 300, width: 80, height: 20});
                platforms.push({x: 150, y: 250, width: 80, height: 20});
                platforms.push({x: 300, y: 200, width: 80, height: 20});
                platforms.push({x: 200, y: 150, width: 80, height: 20});
                
                // Add obstacles
                obstacles.push({x: 80, y: 50, width: 40, height: 40, type: 'spike', speedY: 2.2});
                obstacles.push({x: 200, y: 80, width: 40, height: 40, type: 'spike', speedY: 1.9});
                obstacles.push({x: 320, y: 120, width: 40, height: 40, type: 'spike', speedY: 1.7});
                obstacles.push({x: 150, y: 180, width: 40, height: 40, type: 'spike', speedY: 1.5});
                
                // Add rings
                rings.push({x: 80, y: 420, radius: 12, collected: false});
                rings.push({x: 210, y: 370, radius: 12, collected: false});
                rings.push({x: 130, y: 320, radius: 12, collected: false});
                rings.push({x: 280, y: 270, radius: 12, collected: false});
                rings.push({x: 180, y: 220, radius: 12, collected: false});
                rings.push({x: 330, y: 170, radius: 12, collected: false});
                rings.push({x: 230, y: 120, radius: 12, collected: false});
                
                // Add goal
                goal = {x: canvas.width - 80, y: 100, width: 60, height: 60};
            }
        }
        
        // Update score and level displays
        function updateDisplays() {
            scoreDisplay.textContent = `Score: ${score}`;
            levelDisplay.textContent = `Level: ${level}`;
        }
        
        // Main game loop
        function gameLoop() {
            if (!gameRunning) return;
            
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw background
            drawBackground();
            
            // Update and draw game objects
            updateBall();
            updateObstacles();
            drawPlatforms();
            drawObstacles();
            drawRings();
            drawGoal();
            drawBall();
            
            // Check for collisions
            checkPlatformCollisions();
            checkObstacleCollisions();
            checkRingCollisions();
            checkGoalCollision();
            
            // Check if ball fell off screen
            if (ball.y > canvas.height) {
                gameOver();
                return;
            }
            
            // Continue game loop
            requestAnimationFrame(gameLoop);
        }
        
        // Draw background
        function drawBackground() {
            // Draw gradient background
            const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
            gradient.addColorStop(0, '#0a0e29');
            gradient.addColorStop(1, '#1a1f4b');
            ctx.fillStyle = gradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw stars in background
            ctx.fillStyle = 'white';
            for (let i = 0; i < 50; i++) {
                const x = Math.random() * canvas.width;
                const y = Math.random() * canvas.height;
                const radius = Math.random() * 1.5;
                ctx.beginPath();
                ctx.arc(x, y, radius, 0, Math.PI * 2);
                ctx.fill();
            }
        }
        
        // Update ball position and physics
        function updateBall() {
            // Apply gravity
            ball.speedY += gravity;
            
            // Apply friction
            ball.speedX *= friction;
            
            // Handle controls
            if (keys['ArrowLeft'] || keys['a'] || keys['A']) {
                ball.speedX = -5;
            }
            if (keys['ArrowRight'] || keys['d'] || keys['D']) {
                ball.speedX = 5;
            }
            
            // Update position
            ball.x += ball.speedX;
            ball.y += ball.speedY;
            
            // Boundary checks
            if (ball.x - ball.radius < 0) {
                ball.x = ball.radius;
                ball.speedX *= -0.5;
            }
            if (ball.x + ball.radius > canvas.width) {
                ball.x = canvas.width - ball.radius;
                ball.speedX *= -0.5;
            }
        }
        
        // Update obstacles (move them down)
        function updateObstacles() {
            obstacles.forEach(obstacle => {
                obstacle.y += obstacle.speedY;
                
                // Reset obstacle if it goes off screen
                if (obstacle.y > canvas.height) {
                    obstacle.y = -50;
                    obstacle.x = Math.random() * (canvas.width - obstacle.width);
                }
            });
        }
        
        // Draw the ball
        function drawBall() {
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            
            // Create gradient for ball
            const gradient = ctx.createRadialGradient(
                ball.x - 5, ball.y - 5, 1,
                ball.x, ball.y, ball.radius
            );
            gradient.addColorStop(0, '#ffffff');
            gradient.addColorStop(0.5, ball.color);
            gradient.addColorStop(1, '#1a5a92');
            
            ctx.fillStyle = gradient;
            ctx.fill();
            
            // Add highlight
            ctx.beginPath();
            ctx.arc(ball.x - 5, ball.y - 5, ball.radius / 3, 0, Math.PI * 2);
            ctx.fillStyle = 'rgba(255, 255, 255, 0.6)';
            ctx.fill();
        }
        
        // Draw platforms
        function drawPlatforms() {
            ctx.fillStyle = '#3a7bd5';
            platforms.forEach(platform => {
                ctx.fillRect(platform.x, platform.y, platform.width, platform.height);
                
                // Add platform details
                ctx.fillStyle = '#2a5b9d';
                ctx.fillRect(platform.x, platform.y, platform.width, 5);
                ctx.fillRect(platform.x, platform.y + platform.height - 5, platform.width, 5);
            });
        }
        
        // Draw obstacles
        function drawObstacles() {
            obstacles.forEach(obstacle => {
                if (obstacle.type === 'spike') {
                    // Draw spike obstacle
                    ctx.fillStyle = '#e74c3c';
                    ctx.beginPath();
                    ctx.moveTo(obstacle.x, obstacle.y + obstacle.height);
                    ctx.lineTo(obstacle.x + obstacle.width / 2, obstacle.y);
                    ctx.lineTo(obstacle.x + obstacle.width, obstacle.y + obstacle.height);
                    ctx.closePath();
                    ctx.fill();
                    
                    // Add spike details
                    ctx.fillStyle = '#c0392b';
                    ctx.beginPath();
                    ctx.moveTo(obstacle.x + 5, obstacle.y + obstacle.height - 5);
                    ctx.lineTo(obstacle.x + obstacle.width / 2, obstacle.y + 10);
                    ctx.lineTo(obstacle.x + obstacle.width - 5, obstacle.y + obstacle.height - 5);
                    ctx.closePath();
                    ctx.fill();
                }
            });
        }
        
        // Draw rings
        function drawRings() {
            rings.forEach(ring => {
                if (!ring.collected) {
                    ctx.strokeStyle = '#f1c40f';
                    ctx.lineWidth = 3;
                    ctx.beginPath();
                    ctx.arc(ring.x, ring.y, ring.radius, 0, Math.PI * 2);
                    ctx.stroke();
                    
                    // Add ring glow
                    ctx.beginPath();
                    ctx.arc(ring.x, ring.y, ring.radius + 5, 0, Math.PI * 2);
                    ctx.strokeStyle = 'rgba(241, 196, 15, 0.3)';
                    ctx.lineWidth = 5;
                    ctx.stroke();
                }
            });
        }
        
        // Draw goal
        function drawGoal() {
            ctx.fillStyle = '#2ecc71';
            ctx.fillRect(goal.x, goal.y, goal.width, goal.height);
            
            // Add goal details
            ctx.fillStyle = '#27ae60';
            ctx.fillRect(goal.x, goal.y, goal.width, 10);
            ctx.fillRect(goal.x, goal.y + goal.height - 10, goal.width, 10);
            ctx.fillRect(goal.x, goal.y, 10, goal.height);
            ctx.fillRect(goal.x + goal.width - 10, goal.y, 10, goal.height);
            
            // Draw goal text
            ctx.fillStyle = 'white';
            ctx.font = 'bold 16px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('GOAL', goal.x + goal.width/2, goal.y + goal.height/2 + 5);
        }
        
        // Check for platform collisions
        function checkPlatformCollisions() {
            let onPlatform = false;
            
            platforms.forEach(platform => {
                // Check if ball is colliding with platform from above
                if (
                    ball.x + ball.radius > platform.x &&
                    ball.x - ball.radius < platform.x + platform.width &&
                    ball.y + ball.radius > platform.y &&
                    ball.y - ball.radius < platform.y + platform.height &&
                    ball.speedY > 0
                ) {
                    // Place ball on top of platform
                    ball.y = platform.y - ball.radius;
                    ball.speedY = 0;
                    isJumping = false;
                    onPlatform = true;
                }
            });
            
            // Apply gravity if not on platform
            if (!onPlatform && ball.speedY === 0) {
                ball.speedY = 0.1; // Small value to start falling
            }
        }
        
        // Check for obstacle collisions
        function checkObstacleCollisions() {
            obstacles.forEach(obstacle => {
                if (obstacle.type === 'spike') {
                    // Simplified collision for spike (triangle)
                    const dx = Math.abs(ball.x - (obstacle.x + obstacle.width/2));
                    const dy = Math.abs(ball.y - (obstacle.y + obstacle.height/2));
                    
                    if (dx < obstacle.width/2 + ball.radius && dy < obstacle.height/2 + ball.radius) {
                        // More precise collision for triangle
                        const triangleTop = obstacle.y;
                        const triangleBottom = obstacle.y + obstacle.height;
                        const triangleLeft = obstacle.x;
                        const triangleRight = obstacle.x + obstacle.width;
                        
                        // Check if ball center is below the diagonal lines of the triangle
                        const slope = obstacle.height / (obstacle.width / 2);
                        const leftEdge = triangleTop + slope * (ball.x - triangleLeft);
                        const rightEdge = triangleTop + slope * (triangleRight - ball.x);
                        
                        if (ball.y > leftEdge && ball.y > rightEdge && ball.y < triangleBottom) {
                            gameOver();
                        }
                    }
                }
            });
        }
        
        // Check for ring collisions
        function checkRingCollisions() {
            rings.forEach(ring => {
                if (
                    !ring.collected &&
                    Math.sqrt((ball.x - ring.x) ** 2 + (ball.y - ring.y) ** 2) < ball.radius + ring.radius
                ) {
                    ring.collected = true;
                    score += 50;
                    updateDisplays();
                }
            });
        }
        
        // Check for goal collision
        function checkGoalCollision() {
            if (
                ball.x + ball.radius > goal.x &&
                ball.x - ball.radius < goal.x + goal.width &&
                ball.y + ball.radius > goal.y &&
                ball.y - ball.radius < goal.y + goal.height
            ) {
                levelComplete();
            }
        }
        
        // Handle level completion
        function levelComplete() {
            gameRunning = false;
            levelScore.textContent = `Score this level: ${score}`;
            levelCompleteScreen.classList.remove('hidden');
        }
        
        // Handle game over
        function gameOver() {
            gameRunning = false;
            finalScore.textContent = `Your score: ${score}`;
            gameOverScreen.classList.remove('hidden');
        }
        
        // Initialize the game when page loads
        window.onload = init;
    </script>
</body>
</html>
