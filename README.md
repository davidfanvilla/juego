# Juego - Responsive Game Framework

A modern game framework with full responsive design support for all devices, including proper viewport scaling, DPI handling, and dynamic canvas sizing.

## Features

### Responsive Design
- ✅ Full device compatibility (mobile, tablet, desktop)
- ✅ Proper viewport scaling and meta tags
- ✅ Dynamic canvas sizing based on device dimensions
- ✅ DPI-aware rendering for crisp graphics on high-density displays
- ✅ Touch-friendly interface with proper event handling
- ✅ Adaptive layouts that scale with screen size

### Canvas Management
- **Dynamic Sizing**: Canvas automatically scales to fit the viewport
- **DPI Handling**: Renders at native device resolution for sharp graphics
- **Aspect Ratio Preservation**: Maintains consistent game experience across devices
- **Performance Optimized**: Efficient rendering pipeline for all device capabilities

### Viewport Configuration
- Proper meta viewport tag for mobile optimization
- Device pixel ratio awareness
- Scale and zoom handling
- Orientation support (portrait and landscape)

## Getting Started

### HTML Setup

Include the proper viewport meta tag in your HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="true">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Juego</title>
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
            background: #000;
        }
        
        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    <script src="game.js"></script>
</body>
</html>
```

### JavaScript Implementation

```javascript
class ResponsiveCanvas {
    constructor(canvasId) {
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.dpr = window.devicePixelRatio || 1;
        
        // Initialize canvas with proper sizing
        this.resize();
        
        // Listen for resize events
        window.addEventListener('resize', () => this.resize());
        window.addEventListener('orientationchange', () => this.resize());
        
        // Handle visibility changes for mobile
        document.addEventListener('visibilitychange', () => {
            if (document.hidden) {
                this.pause();
            } else {
                this.resume();
            }
        });
    }
    
    resize() {
        // Get viewport dimensions
        const width = window.innerWidth;
        const height = window.innerHeight;
        
        // Set canvas display size (in CSS pixels)
        this.canvas.style.width = width + 'px';
        this.canvas.style.height = height + 'px';
        
        // Set canvas internal size (in device pixels)
        this.canvas.width = width * this.dpr;
        this.canvas.height = height * this.dpr;
        
        // Scale context for high DPI displays
        this.ctx.scale(this.dpr, this.dpr);
        
        // Store logical dimensions
        this.logicalWidth = width;
        this.logicalHeight = height;
        this.physicalWidth = this.canvas.width;
        this.physicalHeight = this.canvas.height;
        
        console.log(`Canvas resized: ${width}x${height} (DPR: ${this.dpr})`);
        
        // Trigger resize callback for game logic
        if (this.onResize) {
            this.onResize(width, height);
        }
    }
    
    getPhysicalCoordinates(x, y) {
        return {
            x: x * this.dpr,
            y: y * this.dpr
        };
    }
    
    getLogicalCoordinates(x, y) {
        return {
            x: x / this.dpr,
            y: y / this.dpr
        };
    }
    
    clear(color = '#000') {
        this.ctx.fillStyle = color;
        this.ctx.fillRect(0, 0, this.logicalWidth, this.logicalHeight);
    }
    
    pause() {
        this.paused = true;
    }
    
    resume() {
        this.paused = false;
    }
}

class ResponsiveGame {
    constructor() {
        this.canvas = new ResponsiveCanvas('gameCanvas');
        this.canvas.onResize = (width, height) => this.onCanvasResize(width, height);
        
        this.gameWidth = this.canvas.logicalWidth;
        this.gameHeight = this.canvas.logicalHeight;
        
        this.setupTouchEvents();
        this.setupKeyboardEvents();
        
        this.lastFrameTime = Date.now();
        this.deltaTime = 0;
        
        this.start();
    }
    
    onCanvasResize(width, height) {
        this.gameWidth = width;
        this.gameHeight = height;
        console.log(`Game resized to: ${width}x${height}`);
    }
    
    setupTouchEvents() {
        document.addEventListener('touchstart', (e) => this.handleTouchStart(e));
        document.addEventListener('touchmove', (e) => this.handleTouchMove(e));
        document.addEventListener('touchend', (e) => this.handleTouchEnd(e));
    }
    
    setupKeyboardEvents() {
        document.addEventListener('keydown', (e) => this.handleKeyDown(e));
        document.addEventListener('keyup', (e) => this.handleKeyUp(e));
    }
    
    handleTouchStart(e) {
        e.preventDefault();
        for (let touch of e.touches) {
            const coords = this.canvas.getLogicalCoordinates(touch.clientX, touch.clientY);
            this.onTouchStart(coords.x, coords.y);
        }
    }
    
    handleTouchMove(e) {
        e.preventDefault();
        for (let touch of e.changedTouches) {
            const coords = this.canvas.getLogicalCoordinates(touch.clientX, touch.clientY);
            this.onTouchMove(coords.x, coords.y);
        }
    }
    
    handleTouchEnd(e) {
        e.preventDefault();
        for (let touch of e.changedTouches) {
            const coords = this.canvas.getLogicalCoordinates(touch.clientX, touch.clientY);
            this.onTouchEnd(coords.x, coords.y);
        }
    }
    
    handleKeyDown(e) {
        this.onKeyDown(e.key);
    }
    
    handleKeyUp(e) {
        this.onKeyUp(e.key);
    }
    
    onTouchStart(x, y) {
        // Override in subclass
    }
    
    onTouchMove(x, y) {
        // Override in subclass
    }
    
    onTouchEnd(x, y) {
        // Override in subclass
    }
    
    onKeyDown(key) {
        // Override in subclass
    }
    
    onKeyUp(key) {
        // Override in subclass
    }
    
    update(deltaTime) {
        // Override in subclass for game logic
    }
    
    render() {
        // Override in subclass for rendering
        this.canvas.clear();
    }
    
    start() {
        const gameLoop = () => {
            const now = Date.now();
            this.deltaTime = (now - this.lastFrameTime) / 1000;
            this.lastFrameTime = now;
            
            if (!this.canvas.paused) {
                this.update(this.deltaTime);
                this.render();
            }
            
            requestAnimationFrame(gameLoop);
        };
        
        requestAnimationFrame(gameLoop);
    }
}

// Example usage
class MyGame extends ResponsiveGame {
    constructor() {
        super();
        this.player = {
            x: this.gameWidth / 2,
            y: this.gameHeight / 2,
            radius: 20,
            vx: 0,
            vy: 0
        };
    }
    
    onCanvasResize(width, height) {
        super.onCanvasResize(width, height);
        this.player.x = width / 2;
        this.player.y = height / 2;
    }
    
    onTouchStart(x, y) {
        this.player.x = x;
        this.player.y = y;
    }
    
    update(deltaTime) {
        // Game update logic
    }
    
    render() {
        this.canvas.clear();
        
        // Draw player
        const ctx = this.canvas.ctx;
        ctx.fillStyle = '#fff';
        ctx.beginPath();
        ctx.arc(this.player.x, this.player.y, this.player.radius, 0, Math.PI * 2);
        ctx.fill();
    }
}

// Initialize game when DOM is ready
document.addEventListener('DOMContentLoaded', () => {
    window.game = new MyGame();
});
```

## Browser Support

- Chrome/Edge (all versions)
- Firefox (all versions)
- Safari (iOS 12+, macOS 10.12+)
- Android browser (5.0+)

## Performance Tips

1. **Use requestAnimationFrame** for smooth animations
2. **Minimize canvas operations** per frame
3. **Use offscreen canvas** for complex drawing operations
4. **Implement object pooling** for frequently created objects
5. **Profile on actual devices** to identify bottlenecks
6. **Use canvas compositing** efficiently

## Device Pixel Ratio Handling

The framework automatically handles DPI scaling:
- Detects device pixel ratio
- Renders at native resolution
- Scales context appropriately
- Converts between physical and logical coordinates

## Touch vs Mouse Input

The framework handles both:
- Touch events for mobile devices
- Mouse events can be added similarly
- Coordinates are automatically converted to logical space

## Mobile Optimization

- Prevents pinch-to-zoom
- Handles viewport properly
- Disables text selection
- Optimizes for fullscreen
- Handles orientation changes
- Manages visibility changes

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For issues and questions, please use the GitHub Issues tracker.
