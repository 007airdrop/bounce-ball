<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
    <title>Nokia Bounce (Clone)</title>
    <script src="https://cdn.jsdelivr.net/npm/@farcade/game-sdk@latest/dist/index.min.js"></script>
    <style>
      :root {
        color-scheme: dark;
      }
      html,
      body {
        margin: 0;
        padding: 0;
        height: 100%;
        width: 100%;
        background: #000;
        overflow: hidden;
        -webkit-tap-highlight-color: transparent;
        font-family: Arial, Helvetica, sans-serif;
      }
      #game-wrap {
        position: fixed;
        inset: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        background: #000;
      }
      canvas {
        width: 100%;
        height: 100%;
        touch-action: none;
      }
      .overlay {
        position: absolute;
        inset: 0;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        padding: 20px;
        color: #fff;
        text-align: center;
        pointer-events: none;
      }
      .title {
        font-size: 28px;
        letter-spacing: 1px;
        margin-bottom: 14px;
        text-shadow: 0 2px 0 rgba(0, 0, 0, 0.3);
      }
      .subtitle {
        opacity: 0.8;
        font-size: 14px;
        margin-bottom: 18px;
      }
      .btn {
        pointer-events: auto;
        margin: 8px 0;
        padding: 12px 18px;
        border-radius: 10px;
        border: 2px solid #fff;
        color: #fff;
        background: rgba(255, 255, 255, 0.08);
        font-weight: bold;
        letter-spacing: 0.5px;
      }
      .hint {
        font-size: 12px;
        opacity: 0.7;
        margin-top: 8px;
      }

      /* On-screen controls (mobile) */
      .controls {
        position: absolute;
        bottom: 16px;
        left: 0;
        right: 0;
        display: flex;
        justify-content: center;
        gap: 14px;
        pointer-events: none;
      }
      .control-btn {
        pointer-events: auto;
        width: 64px;
        height: 64px;
        border-radius: 50%;
        background: rgba(255, 255, 255, 0.08);
        border: 2px solid rgba(255, 255, 255, 0.7);
        display: flex;
        align-items: center;
        justify-content: center;
        color: #fff;
        font-weight: 900;
        user-select: none;
      }
      .control-btn:active {
        background: rgba(255, 255, 255, 0.18);
      }
    
/* Mobile touch controls and layout improvements */
html, body {
  overscroll-behavior: none;
}
body {
  touch-action: manipulation;
}
.touch-controls {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  gap: 12px;
  justify-content: space-between;
  padding: 12px calc(12px + env(safe-area-inset-right)) calc(12px + env(safe-area-inset-bottom)) calc(12px + env(safe-area-inset-left));
  pointer-events: auto;
  z-index: 10;
}
.touch-controls button {
  flex: 1;
  font-size: 22px;
  line-height: 1;
  padding: 14px 0;
  background: rgba(0, 0, 0, 0.5);
  color: #fff;
  border: 2px solid rgba(255,255,255,0.3);
  border-radius: 14px;
  box-shadow: 0 6px 16px rgba(0,0,0,0.35);
  backdrop-filter: blur(6px);
  -webkit-tap-highlight-color: transparent;
  touch-action: manipulation;
}
.touch-controls button:active {
  transform: translateY(1px);
  filter: brightness(1.15);
}
.touch-controls .jump {
  flex: 1.2;
}
/* Ensure canvas sits above background but below controls */
#canvas { position: relative; z-index: 1; }
</style>

    <!-- Config: tweak gameplay and visuals without editing code -->
    <script id="game-config" type="application/json">
      {
        "colors": {
          "background": "#001014",
          "ground": "#08333C",
          "platform": "#0AA6B5",
          "spike": "#E24D4D",
          "ball": "#FFCC33",
          "hud": "#FFFFFF",
          "hudShadow": "#000000",
          "ring": "#FFD862"
        },
        "player": {
          "radius": 12,
          "moveSpeed": 2.6,
          "bounce": 0.82,
          "airControl": 0.85,
          "maxHSpeed": 4.2
        },
        "gameplay": {
          "gravity": 0.32,
          "friction": 0.02,
          "levelScrollSpeed": 1.1,
          "ringScore": 130,
          "finishBonus": 350
        },
        "enemy": {
          "spikeSize": 17
        },
        "ui": {
          "showFPS": false,
          "enableParticles": true
        },
        "_meta": {
          "colors.background": {
            "type": "color",
            "label": "Background Color"
          },
          "colors.ground": {
            "type": "color",
            "label": "Ground Color"
          },
          "colors.platform": {
            "type": "color",
            "label": "Platform Color"
          },
          "colors.spike": {
            "type": "color",
            "label": "Spike Color"
          },
          "colors.ball": {
            "type": "color",
            "label": "Ball Color"
          },
          "colors.hud": {
            "type": "color",
            "label": "HUD Color"
          },
          "colors.ring": {
            "type": "color",
            "label": "Ring Color"
          },
          "player.radius": {
            "type": "number",
            "label": "Ball Radius",
            "min": 6,
            "max": 22,
            "step": 1
          },
          "player.moveSpeed": {
            "type": "number",
            "label": "Move Speed",
            "min": 1,
            "max": 6,
            "step": 0.1
          },
          "player.bounce": {
            "type": "number",
            "label": "Bounce",
            "min": 0.5,
            "max": 1.1,
            "step": 0.01
          },
          "player.airControl": {
            "type": "number",
            "label": "Air Control",
            "min": 0.5,
            "max": 1,
            "step": 0.01
          },
          "player.maxHSpeed": {
            "type": "number",
            "label": "Max Horizontal Speed",
            "min": 2,
            "max": 8,
            "step": 0.1
          },
          "gameplay.gravity": {
            "type": "number",
            "label": "Gravity",
            "min": 0.1,
            "max": 1,
            "step": 0.01
          },
          "gameplay.friction": {
            "type": "number",
            "label": "Friction",
            "min": 0,
            "max": 0.1,
            "step": 0.005
          },
          "gameplay.levelScrollSpeed": {
            "type": "number",
            "label": "Auto Scroll Speed",
            "min": 0,
            "max": 3,
            "step": 0.05
          },
          "gameplay.ringScore": {
            "type": "number",
            "label": "Ring Score",
            "min": 10,
            "max": 500,
            "step": 10
          },
          "gameplay.finishBonus": {
            "type": "number",
            "label": "Finish Bonus",
            "min": 50,
            "max": 1000,
            "step": 50
          },
          "enemy.spikeSize": {
            "type": "number",
            "label": "Spike Size",
            "min": 8,
            "max": 28,
            "step": 1
          },
          "ui.showFPS": {
            "type": "boolean",
            "label": "Show FPS"
          },
          "ui.enableParticles": {
            "type": "boolean",
            "label": "Enable Particles"
          }
        }
      }
    </script>

    <!-- Assets: swap art and audio easily -->
    <script id="game-assets" type="application/json">
      {
        "sounds": {
          "bounce": "",
          "ring": "",
          "hit": "",
          "bg": ""
        },
        "_meta": {
          "sounds.bounce": { "label": "Bounce Sound", "category": "Sound Effects" },
          "sounds.ring": { "label": "Ring Collect", "category": "Sound Effects" },
          "sounds.hit": { "label": "Hit/Death", "category": "Sound Effects" },
          "sounds.bg": { "label": "Background Music", "category": "Music" }
        }
      }
    </script>
  </head>
  <body>
    <div id="game-wrap">
      <canvas id="canvas" width="400" height="600"></canvas>
      <div id="overlay" class="overlay" style="display: none">
        <div class="title">Nokia Bounce</div>
        <div class="subtitle">
          Tap to bounce and steer the ball to the goal. Avoid spikes. Collect rings for points!
        </div>
        <button id="startBtn" class="btn">Tap to Start</button>
        <div class="hint">Controls: Tap left/right half to move. Keyboard: ← → or A/D. R to restart.</div>
      </div>
      <div id="controls" class="controls" style="display: none">
        <div id="btnLeft" class="control-btn">◀</div>
        <div id="btnRight" class="control-btn">▶</div>
      </div>
    </div>

    <script>
      // Globals
      let canvas, ctx;
      let W = 400,
        H = 600;
      let lastTime = 0,
        acc = 0,
        FPS = 0;
      const FIXED_DT = 1000 / 60;
      let gameStarted = false,
        gameOver = false,
        isMuted = false;
      let gameScore = 0;
let scoreTime = 0;
      let levelIndex = 0;

      // Config
      let CONFIG = {};
      function initConfig() {
        try {
          CONFIG = JSON.parse(document.getElementById("game-config").textContent);
        } catch (e) {
          CONFIG = {};
        }
        if (!CONFIG.colors) CONFIG.colors = { background: "#001014", hud: "#fff" };
        if (window.onConfigUpdate) window.onConfigUpdate(CONFIG);
        window.addEventListener("message", (event) => {
          if (event.data && event.data.type === "UPDATE_CONFIG") {
            CONFIG = event.data.config;
            if (window.onConfigUpdate) window.onConfigUpdate(CONFIG);
          }
        });
      }
      window.onConfigUpdate = (config) => {
        document.body.style.backgroundColor = config.colors.background;
      };

      // Assets
      let ASSETS = {};
      const IMG = {},
        SND = {};
      function initAssets() {
        try {
          ASSETS = JSON.parse(document.getElementById("game-assets").textContent);
        } catch (e) {
          ASSETS = {};
        }
        // sounds
        const s = ASSETS.sounds || {};
        if (s.bounce) {
          SND.bounce = new Audio(s.bounce);
        }
        if (s.ring) {
          SND.ring = new Audio(s.ring);
        }
        if (s.hit) {
          SND.hit = new Audio(s.hit);
        }
        if (s.bg) {
          SND.bg = new Audio(s.bg);
          SND.bg.loop = true;
          SND.bg.volume = 0.4;
        }
        window.addEventListener("message", (event) => {
          if (event.data?.type === "UPDATE_ASSETS") {
            ASSETS = event.data.assets;
            reloadAssets();
          }
        });
      }
      function reloadAssets() {
        const s = ASSETS.sounds || {};
        if (SND.bounce && s.bounce) SND.bounce.src = s.bounce;
        if (SND.ring && s.ring) SND.ring.src = s.ring;
        if (SND.hit && s.hit) SND.hit.src = s.hit;
        if (SND.bg && s.bg) SND.bg.src = s.bg;
      }
      function playSound(aud) {
        if (!aud || isMuted) return;
        try {
          aud.currentTime = 0;
          aud.play();
        } catch (e) {}
      }

      // Simple RNG
      const rand = (a, b) => a + Math.random() * (b - a);

      // Physics / Level
      const keys = { left: false, right: false };
      const touches = { left: false, right: false };
      const inputLeft = () => keys.left || touches.left;
      const inputRight = () => keys.right || touches.right;

      const state = {
        camY: 0,
        chisels: [],
        particles: [],
      };

      const player = {
        x: 200,
        y: 520,
        vx: 0,
        vy: 0,
        r: 12,
        onGround: false,
        alive: true,
      };

      function resetLevel() {
        const c = CONFIG;
        state.chisels = [];
        state.particles = [];
        player.x = 200;
        player.y = 100; // start near top
        player.vx = 0;
        player.vy = 0;
        player.alive = true;
        player.onGround = false;
        player.r = c.player.radius;
        state.camY = 0;
        gameOver = false;
        gameScore = 0;
        scoreTime = 0;
      }

      function collideCircleRect(cx, cy, cr, rx, ry, rw, rh) {
        // Clamp point to rect
        const nx = Math.max(rx, Math.min(cx, rx + rw));
        const ny = Math.max(ry, Math.min(cy, ry + rh));
        const dx = cx - nx;
        const dy = cy - ny;
        return dx * dx + dy * dy <= cr * cr;
      }

      function update(dt) {
        const c = CONFIG;
        if (!c.player) return;

        // Input
        if (inputLeft()) player.vx -= c.player.moveSpeed * (player.onGround ? 1 : c.player.airControl);
        if (inputRight()) player.vx += c.player.moveSpeed * (player.onGround ? 1 : c.player.airControl);

        // Clamp horizontal speed
        player.vx = Math.max(-c.player.maxHSpeed, Math.min(c.player.maxHSpeed, player.vx));

        // Gravity
        player.vy += c.gameplay.gravity;

        // Move
        player.x += player.vx;
        player.y += player.vy;

        // Friction (horizontal)
        player.vx *= 1 - c.gameplay.friction;

        // World bounds (left/right walls)
        if (player.x - player.r < 0) {
          player.x = player.r;
          player.vx *= -0.5;
        }
        if (player.x + player.r > W) {
          player.x = W - player.r;
          player.vx *= -0.5;
        }

        // Ground bounce at bottom
        player.onGround = false;
        if (player.y + player.r >= H) {
          player.y = H - player.r;
          player.vy = -Math.abs(player.vy) * c.player.bounce;
          player.onGround = true;
          playSound(SND.bounce);
        }

        // Spawn chisels from top
        if (!update.nextSpawn) update.nextSpawn = 600; // ms until next spawn
        update.nextSpawn -= dt;
        if (update.nextSpawn <= 0) {
          const w = 40 + Math.random() * 60; // width of chisel head
          const h = 24; // height of chisel head
          const x = Math.random() * (W - w) + w * 0.5;
          const speed = 1.2 + Math.random() * 1.5; // falling speed
          state.chisels.push({ x, y: -h, w, h, speed });
          // decrease interval gradually for difficulty
          const base = 700;
          const variance = 300;
          update.nextSpawn = base + Math.random() * variance;
          if (!update.spawnScale) update.spawnScale = 1;
          update.spawnScale *= 0.995;
          update.nextSpawn *= Math.max(0.6, update.spawnScale);
        }

        // Move chisels and check collision
        const keep = [];
        for (const ch of state.chisels) {
          ch.y += ch.speed;
          // collision circle-rect
          const hit = collideCircleRect(
            player.x,
            player.y,
            player.r * 0.9,
            ch.x - ch.w * 0.5,
            ch.y,
            ch.w,
            ch.h,
          );
          if (hit) {
            player.alive = false;
            gameOver = true;
            playSound(SND.hit);
            endGame();
            break;
          }
          if (ch.y < H + 40) keep.push(ch);
        }
        state.chisels = keep;

        // Score over time (10 pts/sec)
        scoreTime += dt;
        while (scoreTime >= 1000) {
          gameScore += 10;
          scoreTime -= 1000;
        }

        // Particles update
        const nowParticles = [];
        const decay = dt;
        for (const p of state.particles) {
          p.x += p.vx;
          p.y += p.vy;
          p.life -= decay;
          if (p.life > 0) nowParticles.push(p);
        }
        state.particles = nowParticles;

        // Death if goes above top (optional)
        if (player.y + player.r < 0) {
          player.alive = false;
          gameOver = true;
          playSound(SND.hit);
          endGame();
        }
      }

      function draw() {
        const c = CONFIG.colors;
        // Clear
        ctx.fillStyle = c.background;
        ctx.fillRect(0, 0, W, H);

        // Background stripes for depth
        ctx.save();
        ctx.translate(0, 0);
        ctx.fillStyle = CONFIG.colors.ground;
        for (let y = -40; y < H + 40; y += 40) {
          ctx.fillRect(0, y, W, 20);
        }
        ctx.restore();

        // Chisels (falling hazards)
        for (const ch of state.chisels) {
          // draw a chisel-like head: rectangle with a triangle tip
          ctx.fillStyle = CONFIG.colors.spike;
          ctx.strokeStyle = "rgba(0,0,0,0.25)";
          // rect body
          ctx.fillRect(ch.x - ch.w * 0.5, ch.y, ch.w, ch.h);
          // tip
          ctx.beginPath();
          ctx.moveTo(ch.x - ch.w * 0.5, ch.y + ch.h);
          ctx.lineTo(ch.x, ch.y + ch.h + ch.h * 0.6);
          ctx.lineTo(ch.x + ch.w * 0.5, ch.y + ch.h);
          ctx.closePath();
          ctx.fill();
          ctx.stroke();
        }

        // Player (ball) with highlight
        ctx.save();
        ctx.translate(player.x, player.y);
        const grd = ctx.createRadialGradient(-player.r * 0.4, -player.r * 0.4, player.r * 0.2, 0, 0, player.r);
        grd.addColorStop(0, "#ffffff");
        grd.addColorStop(0.15, CONFIG.colors.ball);
        grd.addColorStop(1, shade(CONFIG.colors.ball, -40));
        ctx.fillStyle = grd;
        ctx.beginPath();
        ctx.arc(0, 0, player.r, 0, Math.PI * 2);
        ctx.fill();
        // shadow on floor reference
        ctx.globalAlpha = 0.15;
        ctx.fillStyle = "#000";
        ctx.beginPath();
        ctx.ellipse(0, player.r, player.r * 0.9, player.r * 0.35, 0, 0, Math.PI * 2);
        ctx.fill();
        ctx.globalAlpha = 1;
        ctx.restore();

        // Particles
        for (const p of state.particles) {
          ctx.globalAlpha = Math.max(0, Math.min(1, p.life / 350));
          ctx.fillStyle = p.col || CONFIG.colors.platform;
          ctx.fillRect(p.x - 2, p.y - 2, 4, 4);
        }
        ctx.globalAlpha = 1;

        // HUD
        ctx.fillStyle = CONFIG.colors.hud;
        ctx.font = "20px Arial";
        ctx.textAlign = "left";
        ctx.textBaseline = "top";
        ctx.shadowColor = CONFIG.colors.hudShadow;
        ctx.shadowBlur = 8;
        ctx.fillText("Score: " + gameScore, 12, 10);
        ctx.shadowBlur = 0;

        if (CONFIG.ui.showFPS) {
          ctx.fillStyle = "#0f0";
          ctx.fillText("FPS: " + FPS.toFixed(0), 12, 34);
        }

        if (!gameStarted || gameOver) {
          // Dim screen when overlay active
          ctx.fillStyle = "rgba(0,0,0,0.35)";
          ctx.fillRect(0, 0, W, H);
        }
      }

      function shade(hex, amt) {
        // hex shade utility
        let c = hex.replace("#", "");
        if (c.length === 3) c = c[0] + c[0] + c[1] + c[1] + c[2] + c[2];
        const num = parseInt(c, 16);
        let r = Math.max(0, Math.min(255, (num >> 16) + amt));
        let g = Math.max(0, Math.min(255, ((num >> 8) & 0xff) + amt));
        let b = Math.max(0, Math.min(255, (num & 0xff) + amt));
        return (
          "#" + r.toString(16).padStart(2, "0") + g.toString(16).padStart(2, "0") + b.toString(16).padStart(2, "0")
        );
      }

      function loop(ts) {
        if (!lastTime) lastTime = ts;
        const delta = ts - lastTime;
        lastTime = ts;
        FPS = 1000 / Math.max(1, delta);
        acc += delta;
        while (acc >= FIXED_DT) {
          if (gameStarted && !gameOver) update(FIXED_DT);
          acc -= FIXED_DT;
        }
        draw();
        requestAnimationFrame(loop);
      }

      // Input handlers
      function setupInputs() {
        window.addEventListener("keydown", (e) => {
          if (e.key === "ArrowLeft" || e.key === "a" || e.key === "A") keys.left = true;
          if (e.key === "ArrowRight" || e.key === "d" || e.key === "D") keys.right = true;
          if (e.key === "r" || e.key === "R") {
            resetGame();
            startGame();
          }
        });
        window.addEventListener("keyup", (e) => {
          if (e.key === "ArrowLeft" || e.key === "a" || e.key === "A") keys.left = false;
          if (e.key === "ArrowRight" || e.key === "d" || e.key === "D") keys.right = false;
        });

        // Touch zones: left/right half screen
        canvas.addEventListener(
          "touchstart",
          (e) => {
            e.preventDefault();
            for (const t of e.changedTouches) {
              if (t.clientX < window.innerWidth / 2) touches.left = true;
              else touches.right = true;
            }
          },
          { passive: false },
        );
        canvas.addEventListener(
          "touchend",
          (e) => {
            e.preventDefault();
            touches.left = false;
            touches.right = false;
          },
          { passive: false },
        );

        // On-screen buttons
        const btnLeft = document.getElementById("btnLeft");
        const btnRight = document.getElementById("btnRight");
        const controls = document.getElementById("controls");
        // Show controls on touch devices
        if ("ontouchstart" in window) controls.style.display = "flex";
        const press = (side, v) => {
          if (side === "L") touches.left = v;
          else touches.right = v;
        };
        btnLeft.addEventListener("pointerdown", () => press("L", true));
        btnLeft.addEventListener("pointerup", () => press("L", false));
        btnLeft.addEventListener("pointerleave", () => press("L", false));
        btnRight.addEventListener("pointerdown", () => press("R", true));
        btnRight.addEventListener("pointerup", () => press("R", false));
        btnRight.addEventListener("pointerleave", () => press("R", false));

        // Start button
        document.getElementById("startBtn").addEventListener("click", () => {
          startGame();
        });
      }

      // Farcade SDK integration
      function setupFarcade() {
        window.FarcadeSDK.on("play_again", () => {
          resetGame();
          startGame();
        });
        window.FarcadeSDK.on("toggle_mute", (data) => {
          isMuted = data.isMuted;
          if (SND.bg) {
            if (isMuted) SND.bg.pause();
            else
              try {
                SND.bg.play();
              } catch (e) {}
          }
        });
      }

      function showStartScreen() {
        const overlay = document.getElementById("overlay");
        overlay.style.display = "flex";
      }
      function hideStartScreen() {
        document.getElementById("overlay").style.display = "none";
      }

      function startGame() {
        gameStarted = true;
        gameOver = false;
        gameScore = 0;
        hideStartScreen();
        if (SND.bg && !isMuted)
          try {
            SND.bg.play();
          } catch (e) {}
      }

      function endGame() {
        gameStarted = false; // stop player control loop; overlay will dim via draw
        if (SND.bg) SND.bg.pause();
        window.FarcadeSDK.singlePlayer.actions.gameOver({ score: gameScore });
        // After game over, show start overlay with play again
        setTimeout(() => {
          const overlay = document.getElementById("overlay");
          overlay.style.display = "flex";
          overlay.querySelector(".title").textContent = player.alive ? "Level Complete!" : "Game Over";
          overlay.querySelector("#startBtn").textContent = "Play Again";
        }, 300);
      }

      function resetGame() {
        resetLevel(levelIndex);
      }

      function init() {
        canvas = document.getElementById("canvas");
        ctx = canvas.getContext("2d");
        // Fixed logical resolution for portrait 2:3; canvas CSS scales to fit
        W = canvas.width = 400;
        H = canvas.height = 600;

        initConfig();
        initAssets();
        setupInputs();
        setupFarcade();

        resetLevel();

        // Signal Farcade that the game is ready
        window.FarcadeSDK.singlePlayer.actions.ready();

        // Update subtitle to new mode
        const sub = document.querySelector('#overlay .subtitle');
        if (sub) sub.textContent = 'Move left/right to avoid the falling chisel. Bounce on the floor and survive!';
        showStartScreen();
        requestAnimationFrame(loop);
      }

      if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', init);
      } else {
        init();
      }
    </script>
  
</body>
</html>
