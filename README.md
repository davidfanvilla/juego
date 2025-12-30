<!DOCTYPE html>
<html lang="en">
<head>
    <!-- Character Encoding -->
    <meta charset="UTF-8">
    
    <!-- Viewport Configuration for Mobile Responsiveness -->
    <meta name="viewport" 
          content="width=device-width, 
                   initial-scale=1.0, 
                   maximum-scale=1.0, 
                   user-scalable=no, 
                   viewport-fit=cover">
    
    <!-- Mobile Browser Colors -->
    <meta name="theme-color" content="#1a1a1a">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="Responsive Game">
    
    <!-- Device Orientation Lock -->
    <meta name="screen-orientation" content="landscape">
    
    <!-- Fullscreen Capabilities -->
    <meta name="fullscreen" content="yes">
    
    <title>Responsive Game</title>
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #1a1a1a;
        }
        
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Arial', sans-serif;
            touch-action: none;
        }
        
        #gameContainer {
            display: flex;
            justify-content: center;
            align-items: center;
            width: 100vw;
            height: 100vh;
            background: linear-gradient(135deg, #1a1a1a, #2d2d2d);
            position: relative;
        }
        
        #gameCanvas {
            display: block;
            background-color: #0a0a0a;
            cursor: crosshair;
            touch-action: none;
            image-rendering: pixelated;
            image-rendering: crisp-edges;
            -webkit-tap-highlight-color: transparent;
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            user-select: none;
        }
        
        #statsContainer {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.7);
            color: #00ff00;
            padding: 15px 20px;
            border-radius: 8px;
            font-family: 'Courier New', monospace;
            font-size: clamp(12px, 2vw, 16px);
            z-index: 10;
            backdrop-filter: blur(10px);
        }
        
        #controlsInfo {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: rgba(0, 0, 0, 0.7);
            color: #ffffff;
            padding: 15px 20px;
            border-radius: 8px;
            font-size: clamp(10px, 1.5vw, 14px);
            z-index: 10;
            backdrop-filter: blur(10px);
            text-align: right;
        }
        
        .stat-line {
            margin: 5px 0;
        }
        
        .stat-label {
            color: #ffaa00;
            font-weight: bold;
        }
        
        /* Landscape Mode */
        @media (orientation: landscape) {
            #gameContainer {
                width: 100vw;
                height: 100vh;
            }
        }
        
        /* Portrait Mode */
        @media (orientation: portrait) {
            #gameContainer {
                width: 100vw;
                height: 100vh;
            }
        }
        
        /* Small Phones */
        @media (max-width: 480px) {
            #statsContainer {
                font-size: 11px;
                padding: 10px 15px;
            }
            
            #controlsInfo {
                font-size: 10px;
                padding: 10px 15px;
            }
        }
        
        /* Tablets */
        @media (min-width: 768px) and (max-width: 1024px) {
            #statsContainer {
                font-size: 14px;
            }
            
            #controlsInfo {
                font-size: 13px;
            }
        }
        
        /* Desktop Large Screens */
        @media (min-width: 1920px) {
            #statsContainer {
                font-size: 18px;
                padding: 20px 25px;
            }
            
            #controlsInfo {
                font-size: 16px;
                padding: 20px 25px;
            }
        }
        
        /* Fullscreen Styles */
        #gameCanvas:fullscreen {
            width: 100% !important;
            height: 100% !important;
        }
        
        /* High DPI Screens */
        @media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
            #gameCanvas {
                image-rendering: auto;
            }
        }
        
        /* Touch-optimized */
        @media (hover: none) and (pointer: coarse) {
            #gameCanvas {
                cursor: pointer;
            }
        }
        
        /* Safe area for notched devices */
        @supports (padding: max(0px)) {
            #statsContainer {
                padding-left: max(20px, env(safe-area-inset-left));
                padding-top: max(20px, env(safe-area-inset-top));
            }
            
            #controlsInfo {
                padding-right: max(20px, env(safe-area-inset-right));
                padding-bottom: max(20px, env(safe-area-inset-bottom));
            }
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas"></canvas>
        
        <div id="statsContainer">
            <div class="stat-line">
                <span class="stat-label">FPS:</span> <span id="fps">0</span>
            </div>
            <div class="stat-line">
                <span class="stat-label">Resolution:</span> <span id="resolution">0x0</span>
            </div>
            <div class="stat-line">
                <span class="stat-label">DPI:</span> <span id="dpi">1x</span>
            </div>
            <div class="stat-line">
                <span class="stat-label">Device:</span> <span id="device">Detecting...</span>
            </div>
            <div class="stat-line">
                <span class="stat-label">Orientation:</span> <span id="orientation">-</span>
            </div>
        </div>
        
        <div id="controlsInfo">
            <div><strong>Controls:</strong></div>
            <div>Mouse: Click to interact</div>
            <div>Touch: Tap to interact</div>
            <div>Mobile: Tilt to control</div>
            <div>Keyboard: Arrow keys to move</div>
            <div>F11: Toggle fullscreen</div>
        </div>
    </div>

    <script>
        // ============================================
        // Responsive Game Engine with DPI Support
        // ============================================
        
        class ResponsiveGameEngine {
            constructor() {
                this.canvas = document.getElementById('gameCanvas');
                this.ctx = this.canvas.getContext('2d', { 
                    alpha: false,
                    antialias: false 
                });
                
                // Device information
                this.dpi = window.devicePixelRatio || 1;
                this.lastTime = Date.now();
                this.frameCount = 0;
                this.fps = 0;
                
                // Input handling
                this.mouse = { x: 0, y: 0, pressed: false };
                this.keys = {};
                this.touches = [];
                
                // Game state
                this.gameObjects = [];
                this.particles = [];
                
                // Initialize
                this.init();
            }
            
            init() {
                // Set up event listeners
                this.setupEventListeners();
                
                // Initialize responsive canvas
                this.resizeCanvas();
                
                // Create demo game objects
                this.createDemoObjects();
                
                // Start game loop
                this.gameLoop();
                
                // Update UI
                this.updateUI();
                
                // Handle resize and orientation changes
                window.addEventListener('resize', () => this.handleResize());
                window.addEventListener('orientationchange', () => this.handleOrientationChange());
            }
            
            setupEventListeners() {
                // Mouse events
                document.addEventListener('mousemove', (e) => this.handleMouseMove(e));
                document.addEventListener('mousedown', (e) => this.handleMouseDown(e));
                document.addEventListener('mouseup', (e) => this.handleMouseUp(e));
                document.addEventListener('contextmenu', (e) => e.preventDefault());
                
                // Touch events
                this.canvas.addEventListener('touchstart', (e) => this.handleTouchStart(e));
                this.canvas.addEventListener('touchmove', (e) => this.handleTouchMove(e));
                this.canvas.addEventListener('touchend', (e) => this.handleTouchEnd(e));
                this.canvas.addEventListener('touchcancel', (e) => this.handleTouchCancel(e));
                
                // Keyboard events
                document.addEventListener('keydown', (e) => this.handleKeyDown(e));
                document.addEventListener('keyup', (e) => this.handleKeyUp(e));
                
                // Fullscreen
                document.addEventListener('keydown', (e) => {
                    if (e.key === 'F11') {
                        e.preventDefault();
                        this.toggleFullscreen();
                    }
                });
                
                // Accelerometer for mobile
                if (window.DeviceOrientationEvent) {
                    window.addEventListener('deviceorientation', (e) => this.handleDeviceOrientation(e));
                }
                
                // Prevent scrolling
                document.addEventListener('touchmove', (e) => {
                    if (e.target === this.canvas || e.target.closest('#gameContainer')) {
                        e.preventDefault();
                    }
                }, { passive: false });
            }
            
            resizeCanvas() {
                // Get container dimensions
                const container = document.getElementById('gameContainer');
                const width = container.clientWidth;
                const height = container.clientHeight;
                
                // Account for device pixel ratio
                this.canvas.width = width * this.dpi;
                this.canvas.height = height * this.dpi;
                
                // Set CSS display size
                this.canvas.style.width = width + 'px';
                this.canvas.style.height = height + 'px';
                
                // Scale context to device pixel ratio
                this.ctx.scale(this.dpi, this.dpi);
                
                // Store actual dimensions
                this.displayWidth = width;
                this.displayHeight = height;
                this.actualWidth = this.canvas.width;
                this.actualHeight = this.canvas.height;
            }
            
            handleResize() {
                this.resizeCanvas();
                this.updateUI();
            }
            
            handleOrientationChange() {
                setTimeout(() => {
                    this.resizeCanvas();
                    this.updateUI();
                }, 100);
            }
            
            handleMouseMove(e) {
                const rect = this.canvas.getBoundingClientRect();
                this.mouse.x = (e.clientX - rect.left) / (rect.width / this.displayWidth);
                this.mouse.y = (e.clientY - rect.top) / (rect.height / this.displayHeight);
            }
            
            handleMouseDown(e) {
                this.mouse.pressed = true;
                this.handleMouseMove(e);
            }
            
            handleMouseUp(e) {
                this.mouse.pressed = false;
            }
            
            handleTouchStart(e) {
                e.preventDefault();
                this.touches = [];
                for (let touch of e.touches) {
                    const rect = this.canvas.getBoundingClientRect();
                    const x = (touch.clientX - rect.left) / (rect.width / this.displayWidth);
                    const y = (touch.clientY - rect.top) / (rect.height / this.displayHeight);
                    this.touches.push({ x, y, id: touch.identifier });
                }
                this.mouse.pressed = this.touches.length > 0;
                if (this.touches.length > 0) {
                    this.mouse.x = this.touches[0].x;
                    this.mouse.y = this.touches[0].y;
                }
            }
            
            handleTouchMove(e) {
                e.preventDefault();
                this.handleTouchStart(e);
            }
            
            handleTouchEnd(e) {
                e.preventDefault();
                this.touches = [];
                this.mouse.pressed = false;
            }
            
            handleTouchCancel(e) {
                this.handleTouchEnd(e);
            }
            
            handleKeyDown(e) {
                this.keys[e.key.toLowerCase()] = true;
            }
            
            handleKeyUp(e) {
                this.keys[e.key.toLowerCase()] = false;
            }
            
            handleDeviceOrientation(e) {
                // Store device orientation for use in game logic
                this.deviceOrientation = {
                    alpha: e.alpha, // Z axis rotation (0-360)
                    beta: e.beta,   // X axis rotation (-180 to 180)
                    gamma: e.gamma  // Y axis rotation (-90 to 90)
                };
            }
            
            toggleFullscreen() {
                if (!document.fullscreenElement) {
                    this.canvas.requestFullscreen().catch(err => {
                        console.log(`Error attempting to enable fullscreen: ${err.message}`);
                    });
                } else {
                    document.exitFullscreen();
                }
            }
            
            createDemoObjects() {
                // Create some demo game objects
                for (let i = 0; i < 5; i++) {
                    this.gameObjects.push({
                        x: Math.random() * this.displayWidth,
                        y: Math.random() * this.displayHeight,
                        size: 20 + Math.random() * 30,
                        vx: (Math.random() - 0.5) * 2,
                        vy: (Math.random() - 0.5) * 2,
                        color: `hsl(${Math.random() * 360}, 70%, 50%)`,
                        angle: 0
                    });
                }
            }
            
            update(deltaTime) {
                // Update game objects
                for (let obj of this.gameObjects) {
                    obj.x += obj.vx;
                    obj.y += obj.vy;
                    obj.angle += 2;
                    
                    // Bounce off walls
                    if (obj.x - obj.size < 0 || obj.x + obj.size > this.displayWidth) {
                        obj.vx *= -1;
                        obj.x = Math.max(obj.size, Math.min(this.displayWidth - obj.size, obj.x));
                    }
                    if (obj.y - obj.size < 0 || obj.y + obj.size > this.displayHeight) {
                        obj.vy *= -1;
                        obj.y = Math.max(obj.size, Math.min(this.displayHeight - obj.size, obj.y));
                    }
                }
                
                // Particle updates
                this.particles = this.particles.filter(p => p.life > 0);
                for (let p of this.particles) {
                    p.x += p.vx;
                    p.y += p.vy;
                    p.vy += 0.1; // gravity
                    p.life -= 1;
                }
                
                // Create particles on click/touch
                if (this.mouse.pressed) {
                    for (let i = 0; i < 3; i++) {
                        this.particles.push({
                            x: this.mouse.x,
                            y: this.mouse.y,
                            vx: (Math.random() - 0.5) * 4,
                            vy: (Math.random() - 0.5) * 4,
                            life: 30,
                            size: 3 + Math.random() * 2,
                            color: `hsl(${Math.random() * 360}, 100%, 50%)`
                        });
                    }
                }
                
                // Handle keyboard input
                let moveX = 0, moveY = 0;
                if (this.keys['arrowup'] || this.keys['w']) moveY -= 2;
                if (this.keys['arrowdown'] || this.keys['s']) moveY += 2;
                if (this.keys['arrowleft'] || this.keys['a']) moveX -= 2;
                if (this.keys['arrowright'] || this.keys['d']) moveX += 2;
                
                // Move all objects based on input
                if (moveX !== 0 || moveY !== 0) {
                    for (let obj of this.gameObjects) {
                        obj.x += moveX;
                        obj.y += moveY;
                    }
                }
            }
            
            draw() {
                // Clear canvas
                this.ctx.fillStyle = '#0a0a0a';
                this.ctx.fillRect(0, 0, this.displayWidth, this.displayHeight);
                
                // Draw grid (for reference)
                this.drawGrid();
                
                // Draw game objects
                for (let obj of this.gameObjects) {
                    this.drawGameObject(obj);
                }
                
                // Draw particles
                for (let p of this.particles) {
                    const alpha = p.life / 30;
                    this.ctx.fillStyle = p.color;
                    this.ctx.globalAlpha = alpha;
                    this.ctx.beginPath();
                    this.ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                    this.ctx.fill();
                    this.ctx.globalAlpha = 1;
                }
                
                // Draw mouse cursor
                this.drawCursor();
                
                // Draw touch indicators
                for (let touch of this.touches) {
                    this.ctx.strokeStyle = 'rgba(0, 255, 0, 0.7)';
                    this.ctx.lineWidth = 2;
                    this.ctx.beginPath();
                    this.ctx.arc(touch.x, touch.y, 30, 0, Math.PI * 2);
                    this.ctx.stroke();
                }
            }
            
            drawGameObject(obj) {
                this.ctx.save();
                this.ctx.translate(obj.x, obj.y);
                this.ctx.rotate((obj.angle * Math.PI) / 180);
                
                // Draw object
                this.ctx.fillStyle = obj.color;
                this.ctx.beginPath();
                this.ctx.arc(0, 0, obj.size, 0, Math.PI * 2);
                this.ctx.fill();
                
                // Draw outline
                this.ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
                this.ctx.lineWidth = 2;
                this.ctx.stroke();
                
                this.ctx.restore();
            }
            
            drawGrid() {
                this.ctx.strokeStyle = 'rgba(255, 255, 255, 0.05)';
                this.ctx.lineWidth = 1;
                const gridSize = 50;
                
                for (let x = 0; x < this.displayWidth; x += gridSize) {
                    this.ctx.beginPath();
                    this.ctx.moveTo(x, 0);
                    this.ctx.lineTo(x, this.displayHeight);
                    this.ctx.stroke();
                }
                
                for (let y = 0; y < this.displayHeight; y += gridSize) {
                    this.ctx.beginPath();
                    this.ctx.moveTo(0, y);
                    this.ctx.lineTo(this.displayWidth, y);
                    this.ctx.stroke();
                }
            }
            
            drawCursor() {
                if (!this.mouse.pressed) return;
                
                this.ctx.strokeStyle = 'rgba(0, 255, 0, 0.8)';
                this.ctx.lineWidth = 2;
                this.ctx.beginPath();
                this.ctx.arc(this.mouse.x, this.mouse.y, 15, 0, Math.PI * 2);
                this.ctx.stroke();
            }
            
            gameLoop = () => {
                const now = Date.now();
                const deltaTime = now - this.lastTime;
                this.lastTime = now;
                
                // Update and draw
                this.update(deltaTime);
                this.draw();
                
                // Update FPS
                this.frameCount++;
                if (now - this.lastFpsTime >= 1000) {
                    this.fps = this.frameCount;
                    this.frameCount = 0;
                    this.lastFpsTime = now;
                    this.updateUI();
                }
                
                requestAnimationFrame(this.gameLoop);
            }
            
            updateUI() {
                // Update stats
                document.getElementById('fps').textContent = this.fps;
                document.getElementById('resolution').textContent = 
                    `${this.displayWidth}x${this.displayHeight}`;
                document.getElementById('dpi').textContent = 
                    `${this.dpi.toFixed(1)}x`;
                document.getElementById('orientation').textContent = 
                    screen.orientation?.type || window.innerWidth > window.innerHeight ? 'landscape' : 'portrait';
                
                // Detect device type
                const width = window.innerWidth;
                let device = 'Desktop';
                if (width <= 480) device = 'Phone';
                else if (width <= 1024) device = 'Tablet';
                document.getElementById('device').textContent = device;
            }
        }
        
        // Initialize game engine when DOM is ready
        document.addEventListener('DOMContentLoaded', () => {
            window.gameEngine = new ResponsiveGameEngine();
        });
    </script>
</body>
</html>
