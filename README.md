<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>1:1 Roblox · Mobile</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }
        body {
            background: #1a1e2c;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            width: 100vw;
            overflow: hidden;
            font-family: 'Segoe UI', Roboto, system-ui, sans-serif;
            touch-action: pan-y; 
        }
        #game-wrapper {
            position: relative;
            width: 100%;
            max-width: 480px;
            aspect-ratio: 1 / 1;
            background: #2d3349;
            border-radius: 28px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.6), 0 0 0 2px #4b526e inset;
            overflow: hidden;
            touch-action: none;
        }
        canvas {
            display: block;
            width: 100% !important;
            height: 100% !important;
            background: #8bb0d9;
            image-rendering: crisp-edges; 
            image-rendering: pixelated;
            touch-action: none;
            cursor: grab;
        }
        canvas:active {
            cursor: grabbing;
        }

        /* mobile HUD overlay – Roblox style */
        #hud {
            position: absolute;
            bottom: 12px;
            left: 0;
            right: 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 0 16px;
            pointer-events: none;
        }
        .hud-btn {
            pointer-events: auto;
            background: rgba(20, 24, 36, 0.7);
            backdrop-filter: blur(4px);
            -webkit-backdrop-filter: blur(4px);
            border: 2px solid #a0b3e6;
            border-radius: 40px;
            padding: 10px 18px;
            font-size: 1.1rem;
            font-weight: 700;
            color: white;
            text-shadow: 0 2px 3px rgba(0,0,0,0.6);
            letter-spacing: 0.5px;
            box-shadow: 0 4px 0 #0f121b, 0 6px 12px rgba(0,0,0,0.4);
            transition: all 0.05s ease;
            text-transform: uppercase;
            display: flex;
            align-items: center;
            gap: 6px;
            touch-action: manipulation;
        }
        .hud-btn:active {
            transform: translateY(4px);
            box-shadow: 0 0px 0 #0f121b, 0 6px 12px rgba(0,0,0,0.4);
        }
        #jump-btn {
            background: rgba(240, 180, 40, 0.85);
            border-color: #ffd966;
            font-size: 1.3rem;
            padding: 12px 22px;
        }
        #move-panel {
            display: flex;
            gap: 14px;
            pointer-events: none;
        }
        .move-btn {
            pointer-events: auto;
            background: rgba(30, 40, 60, 0.75);
            backdrop-filter: blur(4px);
            -webkit-backdrop-filter: blur(4px);
            border: 2px solid #7a8bc0;
            border-radius: 60px;
            width: 70px;
            height: 70px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 2.4rem;
            font-weight: 300;
            color: white;
            text-shadow: 0 2px 0 #0a0c14;
            box-shadow: 0 6px 0 #141a28, 0 8px 16px rgba(0,0,0,0.5);
            transition: all 0.04s ease;
            touch-action: manipulation;
        }
        .move-btn:active {
            transform: translateY(6px);
            box-shadow: 0 0px 0 #141a28, 0 8px 16px rgba(0,0,0,0.5);
        }
        #status-tip {
            position: absolute;
            top: 14px;
            left: 0;
            right: 0;
            text-align: center;
            color: rgba(255, 255, 240, 0.9);
            font-size: 0.9rem;
            font-weight: 600;
            text-shadow: 0 2px 6px #0a0e1a;
            letter-spacing: 1px;
            pointer-events: none;
            background: rgba(10, 14, 26, 0.3);
            backdrop-filter: blur(2px);
            -webkit-backdrop-filter: blur(2px);
            padding: 6px 12px;
            margin: 0 auto;
            width: fit-content;
            border-radius: 30px;
            border: 1px solid rgba(255,255,200,0.15);
        }
        @media (max-width: 480px) {
            .move-btn { width: 60px; height: 60px; font-size: 2rem; }
            .hud-btn { padding: 8px 14px; font-size: 0.9rem; }
            #jump-btn { font-size: 1.1rem; padding: 10px 18px; }
        }
        @media (max-width: 400px) {
            .move-btn { width: 54px; height: 54px; font-size: 1.8rem; }
        }
    </style>
</head>
<body>
<div id="game-wrapper">
    <canvas id="gameCanvas" width="600" height="600"></canvas>

    <!-- 1:1 Roblox style HUD -->
    <div id="status-tip">⚡ 1:1 · ROBLOX</div>

    <div id="hud">
        <div id="move-panel">
            <div class="move-btn" id="btn-left">◀</div>
            <div class="move-btn" id="btn-right">▶</div>
        </div>
        <div class="hud-btn" id="jump-btn">⬆ JUMP</div>
    </div>
</div>

<script>
    (function() {
        // ----- 1:1 SCALE ROBLOX GAME (mobile HTML) -----
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // --- fixed 1:1 world (600x600) ---
        const WORLD_SIZE = 600;
        canvas.width = WORLD_SIZE;
        canvas.height = WORLD_SIZE;

        // --- game state ---
        const player = {
            x: 300, y: 300,
            w: 28, h: 40,
            vx: 0, vy: 0,
            speed: 3.6,
            jumpPower: -7.2,
            grounded: false,
            facing: 1, // 1 right, -1 left
        };

        // --- platforms (1:1 scale, roblox style) ---
        const platforms = [
            // ground 
            { x: 0, y: 550, w: 600, h: 50 },
            // floating platforms
            { x: 80, y: 460, w: 110, h: 20 },
            { x: 280, y: 400, w: 100, h: 18 },
            { x: 440, y: 340, w: 100, h: 18 },
            { x: 50, y: 280, w: 90, h: 18 },
            { x: 230, y: 200, w: 100, h: 18 },
            { x: 410, y: 150, w: 80, h: 18 },
            // stairs / block
            { x: 480, y: 490, w: 70, h: 20 },
            { x: 0, y: 120, w: 70, h: 18 },
        ];

        // ---- roblox-style "part" decorations (just visual) ----
        const decos = [
            { x: 30, y: 500, w: 30, h: 30, color: '#f2a65a' },
            { x: 530, y: 500, w: 30, h: 30, color: '#5a8cf2' },
            { x: 150, y: 420, w: 24, h: 24, color: '#f25a8c' },
            { x: 370, y: 360, w: 24, h: 24, color: '#8cf25a' },
            { x: 110, y: 240, w: 20, h: 20, color: '#e6c85a' },
            { x: 490, y: 110, w: 20, h: 20, color: '#c85ae6' },
        ];

        // ---- input (touch / mouse) ----
        const keys = { left: false, right: false, jump: false };

        // ---- helpers ----
        function rectCollide(r1, r2) {
            return r1.x < r2.x + r2.w && r1.x + r1.w > r2.x &&
                   r1.y < r2.y + r2.h && r1.y + r1.h > r2.y;
        }

        // ---- physics update ----
        function update() {
            // horizontal movement
            if (keys.left) {
                player.vx = -player.speed;
                player.facing = -1;
            } else if (keys.right) {
                player.vx = player.speed;
                player.facing = 1;
            } else {
                player.vx *= 0.78; // friction
                if (Math.abs(player.vx) < 0.2) player.vx = 0;
            }

            // gravity
            player.vy += 0.28;
            if (player.vy > 10) player.vy = 10;

            // jump
            if (keys.jump && player.grounded) {
                player.vy = player.jumpPower;
                player.grounded = false;
                keys.jump = false; // single jump
            }

            // apply velocity (with collision)
            // horizontal
            const newX = player.x + player.vx;
            const playerBoxX = { x: newX, y: player.y, w: player.w, h: player.h };
            let blockedX = false;
            for (let p of platforms) {
                if (rectCollide(playerBoxX, p)) {
                    blockedX = true;
                    if (player.vx > 0) {
                        player.x = p.x - player.w;
                    } else if (player.vx < 0) {
                        player.x = p.x + p.w;
                    }
                    player.vx = 0;
                    break;
                }
            }
            if (!blockedX) {
                player.x = newX;
            }

            // vertical
            const newY = player.y + player.vy;
            const playerBoxY = { x: player.x, y: newY, w: player.w, h: player.h };
            let blockedY = false;
            player.grounded = false;
            for (let p of platforms) {
                if (rectCollide(playerBoxY, p)) {
                    blockedY = true;
                    if (player.vy > 0) { // falling down
                        player.y = p.y - player.h;
                        player.grounded = true;
                    } else if (player.vy < 0) { // jumping up
                        player.y = p.y + p.h;
                    }
                    player.vy = 0;
                    break;
                }
            }
            if (!blockedY) {
                player.y = newY;
                if (player.vy > 0) player.grounded = false;
            }

            // boundaries (world edges)
            if (player.x < 0) { player.x = 0; player.vx = 0; }
            if (player.x + player.w > WORLD_SIZE) { player.x = WORLD_SIZE - player.w; player.vx = 0; }
            if (player.y + player.h > WORLD_SIZE) { 
                player.y = WORLD_SIZE - player.h; 
                player.vy = 0; 
                player.grounded = true; 
            }
            if (player.y < 0) { player.y = 0; player.vy = 0; }

            // if fall off the map (respawn)
            if (player.y > WORLD_SIZE + 50) {
                player.x = 280; player.y = 300; player.vx = 0; player.vy = 0;
                player.grounded = false;
            }
        }

        // ---- drawing (1:1 roblox style) ----
        function draw() {
            ctx.clearRect(0, 0, WORLD_SIZE, WORLD_SIZE);
            
            // sky gradient (roblox pastel)
            const grad = ctx.createLinearGradient(0, 0, 0, WORLD_SIZE);
            grad.addColorStop(0, '#7bb8e6');
            grad.addColorStop(0.7, '#b4d9f0');
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, WORLD_SIZE, WORLD_SIZE);

            // clouds (simple)
            ctx.fillStyle = 'rgba(255,255,240,0.2)';
            for (let i=0; i<6; i++) {
                const cx = (i*97 + 30) % 600, cy = (i*43 + 20) % 300;
                ctx.beginPath();
                ctx.arc(cx, cy, 35, 0, Math.PI*2);
                ctx.arc(cx+40, cy-10, 30, 0, Math.PI*2);
                ctx.arc(cx-30, cy-5, 28, 0, Math.PI*2);
                ctx.fill();
            }

            // ---- platforms (1:1 roblox block style) ----
            for (let p of platforms) {
                // base color
                ctx.fillStyle = '#4477b3';
                ctx.shadowColor = '#1f2e4a';
                ctx.shadowBlur = 8;
                ctx.shadowOffsetY = 3;
                ctx.fillRect(p.x, p.y, p.w, p.h);
                // top edge highlight
                ctx.shadowBlur = 0;
                ctx.fillStyle = '#5f93d9';
                ctx.fillRect(p.x, p.y, p.w, 4);
                ctx.fillStyle = '#2b4a73';
                ctx.fillRect(p.x, p.y+p.h-4, p.w, 4);
                // border
                ctx.strokeStyle = '#1f2e4a';
                ctx.lineWidth = 1.5;
                ctx.strokeRect(p.x, p.y, p.w, p.h);
            }

            // ---- decorations (roblox parts) ----
            for (let d of decos) {
                ctx.shadowBlur = 6;
                ctx.shadowColor = '#0a1220';
                ctx.fillStyle = d.color;
                ctx.fillRect(d.x, d.y, d.w, d.h);
                ctx.shadowBlur = 0;
                ctx.fillStyle = 'rgba(255,255,240,0.25)';
                ctx.fillRect(d.x+2, d.y-2, d.w-4, 4);
                ctx.strokeStyle = '#1f2a3a';
                ctx.lineWidth = 1.2;
                ctx.strokeRect(d.x, d.y, d.w, d.h);
            }

            // ---- player (1:1 roblox character) ----
            const px = player.x, py = player.y, f = player.facing;
            // shadow
            ctx.shadowBlur = 18;
            ctx.shadowColor = '#0f1626';
            // torso
            ctx.fillStyle = '#3d5a9e';
            ctx.shadowOffsetY = 4;
            ctx.fillRect(px+4, py+10, 20, 18);
            // head
            ctx.fillStyle = '#f2d9b3';
            ctx.fillRect(px+4, py-2, 20, 16);
            // eyes
            ctx.fillStyle = '#1f1f2e';
            if (f === 1) {
                ctx.fillRect(px+14, py+4, 4, 4);
                ctx.fillRect(px+20, py+4, 4, 4);
            } else {
                ctx.fillRect(px+6, py+4, 4, 4);
                ctx.fillRect(px+12, py+4, 4, 4);
            }
            // legs
            ctx.fillStyle = '#2c3f6e';
            ctx.fillRect(px+4, py+28, 8, 12);
            ctx.fillRect(px+16, py+28, 8, 12);
            // arms (wave)
            ctx.fillStyle = '#3d5a9e';
            if (f === 1) {
                ctx.fillRect(px-4, py+12, 6, 14);
                ctx.fillRect(px+26, py+12, 6, 14);
            } else {
                ctx.fillRect(px-4, py+12, 6, 14);
                ctx.fillRect(px+26, py+12, 6, 14);
            }
            // roblox "R" logo on chest
            ctx.fillStyle = '#e64b4b';
            ctx.font = 'bold 12px "Segoe UI", system-ui';
            ctx.textAlign = 'center';
            ctx.shadowBlur = 6;
            ctx.fillText('R', px+14, py+25);
            // reset shadow
            ctx.shadowBlur = 0;
            ctx.shadowOffsetY = 0;

            // ---- mini grid (1:1 reference) ----
            ctx.strokeStyle = 'rgba(255,255,240,0.04)';
            ctx.lineWidth = 1;
            for (let i=0; i<=WORLD_SIZE; i+=60) {
                ctx.beginPath();
                ctx.moveTo(i, 0);
                ctx.lineTo(i, WORLD_SIZE);
                ctx.stroke();
                ctx.beginPath();
                ctx.moveTo(0, i);
                ctx.lineTo(WORLD_SIZE, i);
                ctx.stroke();
            }

            // HUD text (scale)
            ctx.fillStyle = 'rgba(0,0,0,0.4)';
            ctx.font = 'bold 13px monospace';
            ctx.fillText('⚡ 1:1', 16, 30);
            ctx.fillStyle = 'rgba(255,255,220,0.7)';
            ctx.fillText('⚡ 1:1', 14, 28);
        }

        // ---- game loop ----
        function loop() {
            update();
            draw();
            requestAnimationFrame(loop);
        }

        // ---- input handlers (mobile + desktop) ----
        function setupControls() {
            const leftBtn = document.getElementById('btn-left');
            const rightBtn = document.getElementById('btn-right');
            const jumpBtn = document.getElementById('jump-btn');

            // -- touch / mouse events --
            function attachPress(el, onStart, onEnd) {
                if (!el) return;
                const start = (e) => {
                    e.preventDefault();
                    onStart();
                };
                const end = (e) => {
                    e.preventDefault();
                    onEnd();
                };
                el.addEventListener('mousedown', start);
                el.addEventListener('mouseup', end);
                el.addEventListener('mouseleave', end);
                el.addEventListener('touchstart', start, { passive: false });
                el.addEventListener('touchend', end, { passive: false });
                el.addEventListener('touchcancel', end, { passive: false });
            }

            attachPress(leftBtn, 
                () => { keys.left = true; },
                () => { keys.left = false; }
            );
            attachPress(rightBtn,
                () => { keys.right = true; },
                () => { keys.right = false; }
            );
            attachPress(jumpBtn,
                () => { 
                    if (player.grounded) {
                        keys.jump = true; 
                    } else {
                        // allow jump in air? we use single jump anyway
                        keys.jump = true; 
                    }
                },
                () => { 
                    // keep jump as single press (reset after update)
                    // but we keep false after one jump, but we need to reset 
                    // in update we set keys.jump = false after jump.
                    // but if user holds jump, we prevent multi-jump
                    // we set a flag: but we handle in update
                    // we set keys.jump = false on release to avoid "hold jump" 
                    // but we want single jump press. So we just leave it.
                    // But we also need to let it fire again only after release+press.
                    // So we set to false on release (and in update we only jump if grounded)
                    keys.jump = false;
                }
            );
            // Keyboard fallback (desktop)
            document.addEventListener('keydown', (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a') keys.left = true;
                if (e.key === 'ArrowRight' || e.key === 'd') keys.right = true;
                if (e.key === ' ' || e.key === 'ArrowUp' || e.key === 'w') {
                    e.preventDefault();
                    if (player.grounded) {
                        keys.jump = true;
                    }
                }
            });
            document.addEventListener('keyup', (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a') keys.left = false;
                if (e.key === 'ArrowRight' || e.key === 'd') keys.right = false;
                if (e.key === ' ' || e.key === 'ArrowUp' || e.key === 'w') {
                    e.preventDefault();
                    keys.jump = false;
                }
            });
        }

        // ---- start ----
        setupControls();
        loop();

        // prevent context menu / scroll
        canvas.addEventListener('contextmenu', (e) => e.preventDefault());
        document.addEventListener('touchmove', (e) => {
            if (e.target.closest('#game-wrapper')) e.preventDefault();
        }, { passive: false });

        // resize canvas visual (preserve 1:1)
        function fitCanvas() {
            const wrapper = document.getElementById('game-wrapper');
            const rect = wrapper.getBoundingClientRect();
            // canvas already uses CSS 100% 100%, but ensure ratio
        }
        window.addEventListener('resize', fitCanvas);
        fitCanvas();
    })();
</script>
</body>
</html>
