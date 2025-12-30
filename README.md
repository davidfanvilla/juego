<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Hand Maze: Ultra Precision</title>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.4/p5.min.js"></script>
  <script src="https://unpkg.com/ml5@1/dist/ml5.min.js"></script>

  <style>
    body {
      background-color: #0f172a;
      margin: 0; padding: 0;
      overflow: hidden; touch-action: none;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      display: flex; justify-content: center; align-items: center;
      height: 100vh;
    }

    #overlay {
      position: absolute; top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(15, 23, 42, 1);
      display: flex; flex-direction: column;
      justify-content: center; align-items: center;
      z-index: 100; color: white; text-align: center;
      transition: opacity 0.5s;
      white-space: pre-line;
      padding: 0 18px;
      box-sizing: border-box;
    }

    .btn-start {
      margin-top: 30px; padding: 15px 40px;
      font-size: 1.2rem; font-weight: bold; color: #fff;
      background: linear-gradient(135deg, #3b82f6, #2563eb);
      border: none; border-radius: 50px; cursor: pointer;
      box-shadow: 0 0 25px rgba(59, 130, 246, 0.5);
      animation: pulse 1.5s infinite;
    }

    #loading-text { margin-top: 20px; color: #94a3b8; font-size: 0.9rem; font-family: monospace; }

    @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }
  </style>
</head>

<body>
  <div id="overlay">
    <h1 style="color: #60a5fa; font-size: 2.5rem; margin:0;">🎱 Hand Maze Pro</h1>
    <p>Precisión Móvil Activada</p>
    <button id="btn-init" class="btn-start" onclick="initSystem()">INICIAR SISTEMA</button>
    <div id="loading-text"></div>
  </div>

  <script>
    let handPose, hands = [];
    let systemReady = false;

    // ✅ Video robusto: p5 capture para dibujar + HTMLVideoElement para ml5
    let capture = null;     // p5.MediaElement
    let videoEl = null;     // HTMLVideoElement (capture.elt)
    let camW = 640, camH = 480;
    let videoReady = false;

    // --- JUEGO ---
    let level = 1;
    let maxLevels = 20;
    let state = 'MENU';
    let ball = { x: 50, y: 50, vx: 0, vy: 0, r: 15 };
    let goal = { x: 300, y: 300, r: 30 };
    let holes = [], walls = [];

    // Físicas ajustadas para móviles
    let friction = 0.96;
    let gravity = { x: 0, y: 0 };
    let animFrame = 0;

    // --- CONTROL SUAVIZADO ---
    let handCtrl = {
      active: false,
      rawX: 0, rawY: 0,
      smoothX: 0, smoothY: 0
    };

    // --- CALIBRACIÓN COMERCIAL ---
    let calib = { cx: null, cy: null };

    function preload() {
      handPose = ml5.handPose({ flipped: false });
    }

    function setup() {
      createCanvas(windowWidth, windowHeight);
      pixelDensity(1);
      resizeGameElements();
    }

    function windowResized() {
      resizeCanvas(windowWidth, windowHeight);
      resizeGameElements();
      if (systemReady) loadLevel(level);
    }

    function resizeGameElements() {
      let minDim = min(windowWidth, windowHeight);
      ball.r = max(12, minDim * 0.025);
      goal.r = max(25, minDim * 0.055);
    }

    // ==========================================
    // INIT SISTEMA (FIX undefined width)
    // ==========================================
    function initSystem() {
      const btn = document.getElementById('btn-init');
      const loading = document.getElementById('loading-text');

      btn.style.display = 'none';

      // iPhone / Safari: requiere HTTPS o localhost
      const isLocalhost = (location.hostname === 'localhost' || location.hostname === '127.0.0.1');
      const isHttps = (location.protocol === 'https:');
      if (!isHttps && !isLocalhost) {
        loading.innerText =
          "ERROR: Necesitas abrir esto en HTTPS o localhost.\n" +
          "En iPhone, file:// y muchas apps (IG/WhatsApp) bloquean cámara.\n\n" +
          "Solución rápida:\n- Sube a GitHub Pages / Netlify (HTTPS)\n- O corre un servidor local (localhost)";
        btn.style.display = 'block';
        btn.innerText = "REINTENTAR";
        return;
      }

      loading.innerText = "Accediendo a cámara de alto rendimiento...";

      // Constraints optimizados (tu mismo enfoque)
      const constraints = {
        audio: false,
        video: {
          facingMode: "user",
          width: { ideal: 640 },
          height: { ideal: 480 },
          frameRate: { ideal: 30 }
        }
      };

      try {
        // Creamos el capture con p5 (para tu PIP y compatibilidad)
        capture = createCapture(constraints);
        capture.hide();

        // iOS: playsinline obligatorio
        capture.elt.setAttribute('playsinline', '');
        capture.elt.setAttribute('webkit-playsinline', '');

        // ✅ Espera a metadata real (aquí se arregla el undefined width)
        capture.elt.onloadedmetadata = () => {
          camW = capture.elt.videoWidth || 640;
          camH = capture.elt.videoHeight || 480;

          // videoEl para ml5
          videoEl = capture.elt;
          videoReady = true;

          loading.innerText = "Iniciando IA...";

          handPose.detectStart(videoEl, results => {
            hands = results;

            // Activamos el sistema cuando ya recibimos resultados (1ª vez)
            if (!systemReady) {
              systemReady = true;
              document.getElementById('overlay').style.opacity = '0';
              setTimeout(() => document.getElementById('overlay').style.display = 'none', 500);

              calib.cx = null; calib.cy = null;
              loadLevel(level);
            }
          });
        };

        // Si por alguna razón no dispara metadata, damos feedback
        capture.elt.onerror = () => {
          loading.innerText = "ERROR: No se pudo iniciar el video.\nRevisa permisos de cámara y recarga.";
          btn.style.display = 'block';
          btn.innerText = "REINTENTAR";
        };

      } catch (e) {
        loading.innerText = `ERROR CÁMARA: ${e.name}\n${e.message || ''}`;
        btn.style.display = 'block';
        btn.innerText = "REINTENTAR";
      }
    }

    function draw() {
      if (!systemReady) return;
      animFrame += 0.05;

      background(30, 41, 59);
      drawGrid();

      processHandInput();

      if (state === 'PLAY') {
        updatePhysics();
        checkCollisions();
      }

      drawObstacles();
      drawGoal();
      drawBall();
      drawSign();
      drawHUD();

      if (state === 'MENU') drawMessage("NIVEL " + level, "Haz PINZA 👌 para calibrar");
      else if (state === 'GAMEOVER') drawMessage("¡CAÍSTE!", "Suelta la pinza");
      else if (state === 'WIN') drawMessage("¡CAMPEÓN!", "Juego Completado");

      drawCameraFeed();
    }

    // ==========================================
    // INPUT (LERP) + CALIBRACIÓN
    // ==========================================
    function processHandInput() {
      if (!videoReady) return; // ✅ evita undefined width por timing

      let currentlyActive = false;

      if (hands.length > 0 && hands[0] && hands[0].keypoints) {
        let h = hands[0];
        let t = h.keypoints[4];
        let i = h.keypoints[8];

        if (t && i) {
          let distPinch = dist(t.x, t.y, i.x, i.y);

          // espejo lógico
          let rawX = camW - ((t.x + i.x) / 2);
          let rawY = (t.y + i.y) / 2;

          handCtrl.rawX = rawX;
          handCtrl.rawY = rawY;

          if (distPinch < 45) {
            currentlyActive = true;

            if (state === 'MENU') {
              state = 'PLAY';
              handCtrl.smoothX = rawX;
              handCtrl.smoothY = rawY;

              // ✅ calibración comercial
              calib.cx = rawX;
              calib.cy = rawY;
            }
          } else {
            if (state === 'GAMEOVER') resetLevel();
          }
        }
      }

      handCtrl.active = currentlyActive;

      if (handCtrl.active) {
        let smoothingFactor = 0.15;
        handCtrl.smoothX = lerp(handCtrl.smoothX, handCtrl.rawX, smoothingFactor);
        handCtrl.smoothY = lerp(handCtrl.smoothY, handCtrl.rawY, smoothingFactor);
      }
    }

    function updatePhysics() {
      if (!videoReady) return;

      if (handCtrl.active) {
        let cx = camW / 2;
        let cy = camH / 2;

        let baseX = (calib.cx !== null) ? calib.cx : cx;
        let baseY = (calib.cy !== null) ? calib.cy : cy;

        let tiltX = (handCtrl.smoothX - baseX) / cx;
        let tiltY = (handCtrl.smoothY - baseY) / cy;

        let sensitivity = 1.6;

        gravity.x = tiltX * sensitivity;
        gravity.y = tiltY * sensitivity;
      } else {
        gravity.x = lerp(gravity.x, 0, 0.1);
        gravity.y = lerp(gravity.y, 0, 0.1);
      }

      ball.vx += gravity.x; ball.vy += gravity.y;
      ball.vx *= friction; ball.vy *= friction;
      ball.x += ball.vx; ball.y += ball.vy;

      if (ball.x < ball.r) { ball.x = ball.r; ball.vx *= -0.5; }
      if (ball.x > width - ball.r) { ball.x = width - ball.r; ball.vx *= -0.5; }
      if (ball.y < ball.r) { ball.y = ball.r; ball.vy *= -0.5; }
      if (ball.y > height - ball.r) { ball.y = height - ball.r; ball.vy *= -0.5; }
    }

    // ==========================================
    // CÁMARA ESPEJO (ARRIBA DERECHA)
    // ==========================================
    function drawCameraFeed() {
      if (!capture || !videoReady) return; // ✅ guard

      let pipW = width * 0.25;
      if (pipW < 120) pipW = 120;
      if (pipW > 200) pipW = 200;
      let pipH = pipW * 0.75;

      let padding = 15;
      let x = width - pipW - padding;
      let y = padding;

      push();
      fill(0); stroke(255, 50); strokeWeight(2);
      rect(x, y, pipW, pipH, 8);

      translate(x, y);

      push();
      translate(pipW, 0);
      scale(-1, 1);

      if (handCtrl.active) tint(255, 255); else tint(255, 120);
      image(capture, 0, 0, pipW, pipH); // ✅ aquí usamos p5 capture
      noTint();

      if (hands.length > 0 && hands[0] && hands[0].keypoints) {
        let h = hands[0];
        let sx = pipW / camW;
        let sy = pipH / camH;

        stroke(handCtrl.active ? '#60a5fa' : '#ef4444');
        strokeWeight(2);
        drawSkeletonLines(h, sx, sy);

        if (handCtrl.active) {
          let t = h.keypoints[4];
          let i = h.keypoints[8];
          if (t && i) {
            let cxp = ((t.x + i.x) / 2) * sx;
            let cyp = ((t.y + i.y) / 2) * sy;

            fill(96, 165, 250); noStroke();
            circle(cxp, cyp, 15);
            noFill(); stroke(255); strokeWeight(1);
            circle(cxp, cyp, 20);
          }
        }
      }

      pop();

      fill(255); noStroke(); textAlign(LEFT, TOP); textSize(10);
      text("CÁMARA", 5, 5);
      pop();
    }

    // ==========================================
    // NIVELES / COLISIONES
    // ==========================================
    function loadLevel(n) {
      holes = []; walls = [];
      ball.vx = 0; ball.vy = 0;
      ball.x = width * 0.1; ball.y = height * 0.1;
      goal.x = width * 0.9; goal.y = height * 0.9;

      let count = n * 1.5;
      let safeDist = min(width, height) * 0.20;

      if (n > 1) {
        for (let i = 0; i < count; i++) {
          let hr = ball.r * 1.4;
          let hx, hy, tries = 0;
          do {
            hx = random(50, width - 50); hy = random(50, height - 50);
            tries++; if (tries > 100) break;
          } while (dist(hx, hy, ball.x, ball.y) < safeDist || dist(hx, hy, goal.x, goal.y) < safeDist);
          holes.push({ x: hx, y: hy, r: hr });
        }
      }

      if (n > 2) {
        for (let i = 0; i < count / 1.5; i++) {
          let wW = random(40, 120);
          let wH = random(40, 120);
          let wx = random(50, width - 50 - wW);
          let wy = random(50, height - 50 - wH);
          walls.push({ x: wx, y: wy, w: wW, h: wH });
        }
      }
    }

    function checkCollisions() {
      if (dist(ball.x, ball.y, goal.x, goal.y) < goal.r) {
        level++;
        if (level > maxLevels) state = 'WIN';
        else loadLevel(level);
      }

      for (let h of holes) {
        if (dist(ball.x, ball.y, h.x, h.y) < h.r) state = 'GAMEOVER';
      }

      // ✅ Muros sin atorones: empuja fuera
      for (let w of walls) {
        if (ball.x > w.x - ball.r && ball.x < w.x + w.w + ball.r &&
            ball.y > w.y - ball.r && ball.y < w.y + w.h + ball.r) {

          let left = abs(ball.x - (w.x - ball.r));
          let right = abs((w.x + w.w + ball.r) - ball.x);
          let top = abs(ball.y - (w.y - ball.r));
          let bottom = abs((w.y + w.h + ball.r) - ball.y);

          let m = min(left, right, top, bottom);

          if (m === left) { ball.x = w.x - ball.r; ball.vx *= -0.8; }
          else if (m === right) { ball.x = w.x + w.w + ball.r; ball.vx *= -0.8; }
          else if (m === top) { ball.y = w.y - ball.r; ball.vy *= -0.8; }
          else { ball.y = w.y + w.h + ball.r; ball.vy *= -0.8; }
        }
      }
    }

    function resetLevel() {
      ball.x = width * 0.1; ball.y = height * 0.1;
      ball.vx = 0; ball.vy = 0;
      state = 'MENU';
    }

    // ==========================================
    // DIBUJO
    // ==========================================
    function drawGoal() {
      if (goal.x === 0) { goal.x = width * 0.9; goal.y = height * 0.9; }
      let wave = sin(animFrame * 3) * 5;
      noFill(); stroke(96, 165, 250, 100); strokeWeight(2);
      circle(goal.x, goal.y, (goal.r * 2) + wave + 10);
      fill(59, 130, 246); noStroke(); circle(goal.x, goal.y, goal.r * 2);
      fill(30, 64, 175); circle(goal.x, goal.y, goal.r * 1.5);
      fill(255, 255, 255, 100);
      let arrowY = map(sin(animFrame * 5), -1, 1, -5, 5);
      triangle(goal.x - 5, goal.y - 5 + arrowY, goal.x + 5, goal.y - 5 + arrowY, goal.x, goal.y + 5 + arrowY);
    }

    function drawSign() {
      let signX = goal.x;
      let signY = goal.y - goal.r - 40 + (sin(animFrame * 4) * 5);
      push(); translate(signX, signY);
      rectMode(CENTER); fill(255); stroke(59, 130, 246); strokeWeight(3);
      rect(0, 0, 160, 36, 8);
      noStroke(); fill(255); triangle(-10, 18, 10, 18, 0, 28);
      fill(30, 58, 138); textAlign(CENTER, CENTER); textSize(12); textStyle(BOLD);
      text("¡DEPOSÍTALA AQUÍ!", 0, 0);
      pop();
    }

    function drawBall() {
      fill(240); noStroke(); circle(ball.x, ball.y, ball.r * 2);
      fill(200); circle(ball.x - 2, ball.y + 2, ball.r * 2);
      fill(255); circle(ball.x - 3, ball.y - 3, ball.r * 0.8);
    }

    function drawObstacles() {
      fill(15); noStroke(); for (let h of holes) circle(h.x, h.y, h.r * 2);
      fill(100, 116, 139); stroke(255, 255, 255, 20); strokeWeight(1);
      for (let w of walls) rect(w.x, w.y, w.w, w.h, 4);
    }

    function drawGrid() {
      stroke(255, 10);
      for (let i = 0; i < width; i += 50) line(i, 0, i, height);
      for (let i = 0; i < height; i += 50) line(0, i, width, i);
    }

    function drawHUD() {
      fill(255); noStroke(); textSize(20); textAlign(LEFT, TOP);
      text(`NIVEL ${level}`, 20, 20);
    }

    function drawMessage(t, s) {
      fill(0, 0, 0, 200); noStroke();
      rect(0, 0, width, height);
      fill(255); textAlign(CENTER);
      textSize(min(width, height) * 0.1);
      text(t, width / 2, height / 2 - 20);
      textSize(16); fill(200);
      text(s, width / 2, height / 2 + 30);
    }

    function drawSkeletonLines(h, sx, sy) {
      let p = [
        [0,1],[1,2],[2,3],[3,4],
        [0,5],[5,6],[6,7],[7,8],
        [5,9],[9,13],[13,17],[0,17],
        [13,14],[14,15],[15,16],
        [17,18],[18,19],[19,20]
      ];
      for (let i of p) {
        let a = h.keypoints[i[0]];
        let b = h.keypoints[i[1]];
        if (a && b) line(a.x * sx, a.y * sy, b.x * sx, b.y * sy);
      }
    }
  </script>
</body>
</html>
