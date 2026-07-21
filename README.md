Here is the file contents, still, I recommend you download the file becouse it already has the html extension, and you don't need to use the Jostick and jump button on pc becouse it works with w, a, s, d, and space:<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>3D Mobile Obby - With Ending Screen</title>
  <style>
    * {
      box-sizing: border-box;
      user-select: none;
      -webkit-user-select: none;
      touch-action: none;
    }
    body, html {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: #000;
      font-family: sans-serif;
    }
    #canvas-container {
      width: 100%;
      height: 100%;
    }
    /* Touch UI Controls */
    #joystick-zone {
      position: absolute;
      bottom: 40px;
      left: 40px;
      width: 120px;
      height: 120px;
      background: rgba(255, 255, 255, 0.15);
      border: 2px solid rgba(255, 255, 255, 0.3);
      border-radius: 50%;
      z-index: 10;
    }
    #joystick-knob {
      position: absolute;
      top: 50%;
      left: 50%;
      width: 50px;
      height: 50px;
      background: rgba(255, 255, 255, 0.6);
      border-radius: 50%;
      transform: translate(-50%, -50%);
      pointer-events: none;
    }
    #jump-btn {
      position: absolute;
      bottom: 50px;
      right: 40px;
      width: 80px;
      height: 80px;
      background: rgba(255, 255, 255, 0.25);
      border: 2px solid rgba(255, 255, 255, 0.5);
      border-radius: 50%;
      color: white;
      font-weight: bold;
      font-size: 16px;
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 10;
    }
    #jump-btn:active {
      background: rgba(255, 255, 255, 0.5);
    }
    #camera-touch-zone {
      position: absolute;
      top: 0;
      right: 0;
      width: 60%;
      height: 100%;
      z-index: 5;
    }
    #hud {
      position: absolute;
      top: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 5px;
      z-index: 10;
      pointer-events: none;
    }
    #stage-banner {
      color: white;
      font-size: 22px;
      font-weight: bold;
      text-shadow: 2px 2px 4px rgba(0,0,0,0.8);
    }
    #timer-display {
      color: #facc15;
      font-size: 16px;
      font-weight: bold;
      text-shadow: 1px 1px 3px rgba(0,0,0,0.8);
    }

    /* --- ENDING / VICTORY SCREEN --- */
    #win-screen {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.75);
      backdrop-filter: blur(8px);
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      z-index: 100;
    }
    .win-card {
      background: #1e293b;
      border: 3px solid #facc15;
      border-radius: 20px;
      padding: 30px 40px;
      text-align: center;
      color: white;
      box-shadow: 0 10px 30px rgba(0,0,0,0.5);
      max-width: 85%;
    }
    .win-card h1 {
      margin: 0 0 10px 0;
      font-size: 32px;
      color: #facc15;
      text-shadow: 0 0 10px rgba(250, 204, 21, 0.5);
    }
    .win-card p {
      margin: 8px 0;
      font-size: 18px;
      color: #cbd5e1;
    }
    #win-time {
      font-size: 22px;
      font-weight: bold;
      color: #22c55e;
      margin: 15px 0;
    }
    #restart-btn {
      margin-top: 15px;
      padding: 14px 28px;
      font-size: 18px;
      font-weight: bold;
      color: #0f172a;
      background: #facc15;
      border: none;
      border-radius: 12px;
      cursor: pointer;
      touch-action: manipulation;
    }
    #restart-btn:active {
      transform: scale(0.95);
      background: #eab308;
    }
  </style>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

  <div id="canvas-container"></div>
  
  <div id="hud">
    <div id="stage-banner">STAGE 1</div>
    <div id="timer-display">TIME: 00:00</div>
  </div>
  
  <div id="joystick-zone">
    <div id="joystick-knob"></div>
  </div>
  <div id="camera-touch-zone"></div>
  <div id="jump-btn">JUMP</div>

  <!-- VICTORY OVERLAY -->
  <div id="win-screen">
    <div class="win-card">
      <h1>🏆 YOU WIN! 🏆</h1>
      <p>You completed all 5 stages!</p>
      <div id="win-time">Final Time: 00:00</div>
      <button id="restart-btn">PLAY AGAIN</button>
    </div>
  </div>

  <script>
    // --- 1. SCENE SETUP ---
    const container = document.getElementById('canvas-container');
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x87ceeb);
    scene.fog = new THREE.Fog(0x87ceeb, 30, 120);

    const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.rotation.reorder('YXZ'); // Fixes camera pitch leaking into turning angle

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.shadowMap.enabled = true;
    container.appendChild(renderer.domElement);

    // --- 2. LIGHTING ---
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
    scene.add(ambientLight);

    const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
    dirLight.position.set(30, 50, 30);
    dirLight.castShadow = true;
    dirLight.shadow.mapSize.width = 1024;
    dirLight.shadow.mapSize.height = 1024;
    scene.add(dirLight);

    // --- 3. GAME STATE & TIMER ---
    let currentStage = 1;
    let spawnPos = new THREE.Vector3(0, 3, 0);
    let isGameWon = false;
    let startTime = Date.now();

    function formatTime(ms) {
      const totalSeconds = Math.floor(ms / 1000);
      const mins = Math.floor(totalSeconds / 60).toString().padStart(2, '0');
      const secs = (totalSeconds % 60).toString().padStart(2, '0');
      return `${mins}:${secs}`;
    }

    // --- 4. LEVEL & STAGE BUILDER ---
    const platforms = [];
    const checkpoints = [];

    function createPlatform(x, y, z, width, depth, color = 0x22c55e, isCheckpoint = false, stageNum = 1) {
      const geo = new THREE.BoxGeometry(width, 1, depth);
      const mat = new THREE.MeshStandardMaterial({ color });
      const mesh = new THREE.Mesh(geo, mat);
      mesh.position.set(x, y, z);
      mesh.receiveShadow = true;
      mesh.castShadow = true;
      scene.add(mesh);

      mesh.geometry.computeBoundingBox();
      platforms.push(mesh);

      if (isCheckpoint) {
        mesh.userData = { isCheckpoint: true, stageNum: stageNum };
        checkpoints.push(mesh);
      }
      return mesh;
    }

    // STAGE 1: The Warmup (Green)
    createPlatform(0, 1, 0, 6, 6, 0x22c55e, true, 1);
    createPlatform(0, 1.5, -7, 5, 5, 0x22c55e);
    createPlatform(0, 2.0, -13, 5, 5, 0x22c55e);

    // STAGE 2: Stepping Stones (Cyan)
    createPlatform(0, 2.5, -20, 6, 6, 0x06b6d4, true, 2);
    createPlatform(-3, 3.0, -26, 3.5, 3.5, 0x06b6d4);
    createPlatform(3, 3.5, -32, 3.5, 3.5, 0x06b6d4);
    createPlatform(0, 4.0, -38, 3.5, 3.5, 0x06b6d4);

    // STAGE 3: The Spiral Climb (Purple)
    createPlatform(0, 4.5, -45, 6, 6, 0x8b5cf6, true, 3);
    createPlatform(5, 5.5, -45, 3.5, 3.5, 0x8b5cf6);
    createPlatform(5, 6.5, -50, 3.5, 3.5, 0x8b5cf6);
    createPlatform(0, 7.5, -50, 3.5, 3.5, 0x8b5cf6);
    createPlatform(-5, 8.5, -50, 3.5, 3.5, 0x8b5cf6);
    createPlatform(-5, 9.5, -45, 3.5, 3.5, 0x8b5cf6);

    // STAGE 4: Narrow Beams (Orange)
    createPlatform(0, 10.5, -45, 6, 6, 0xf97316, true, 4);
    createPlatform(0, 11.0, -53, 2, 6, 0xf97316);
    createPlatform(0, 11.5, -61, 2, 6, 0xf97316);
    createPlatform(0, 12.0, -69, 2, 6, 0xf97316);

    // STAGE 5: The Summit & Victory Pad (Gold)
    createPlatform(0, 12.5, -77, 6, 6, 0xeab308, true, 5);
    createPlatform(0, 13.5, -84, 3.5, 3.5, 0xeab308);
    createPlatform(0, 14.5, -91, 3.5, 3.5, 0xeab308);
    
    // FINAL WIN PLATFORM
    const winPlatform = createPlatform(0, 15.5, -100, 10, 10, 0xfacc15);
    winPlatform.userData = { isWin: true };

    // Hazard Lava Floor
    const lavaGeo = new THREE.PlaneGeometry(300, 300);
    const lavaMat = new THREE.MeshBasicMaterial({ color: 0xef4444 });
    const lava = new THREE.Mesh(lavaGeo, lavaMat);
    lava.rotation.x = -Math.PI / 2;
    lava.position.y = -5;
    scene.add(lava);

    // --- 5. PLAYER & PHYSICS ---
    const playerGeo = new THREE.BoxGeometry(1, 1.8, 1);
    const playerMat = new THREE.MeshStandardMaterial({ color: 0x3b82f6 });
    const player = new THREE.Mesh(playerGeo, playerMat);
    player.castShadow = true;
    scene.add(player);

    let velocityY = 0;
    const gravity = -25;
    const jumpForce = 13;
    const moveSpeed = 8;
    let isGrounded = false;

    function respawnPlayer() {
      player.position.copy(spawnPos);
      velocityY = 0;
    }
    respawnPlayer();

    // Trigger Win Function
    function triggerWin() {
      if (isGameWon) return;
      isGameWon = true;

      const elapsed = Date.now() - startTime;
      const timeStr = formatTime(elapsed);

      document.getElementById('win-time').innerText = `Final Time: ${timeStr}`;
      document.getElementById('win-screen').style.display = 'flex';
    }

    // Restart Game Listener
    document.getElementById('restart-btn').addEventListener('click', () => {
      currentStage = 1;
      spawnPos.set(0, 3, 0);
      respawnPlayer();

      document.getElementById('stage-banner').innerText = `STAGE 1`;
      document.getElementById('win-screen').style.display = 'none';

      isGameWon = false;
      startTime = Date.now();
    });

    // --- 6. TOUCH CONTROLS ---
    const joystickData = { x: 0, y: 0 };
    let cameraYaw = 0;
    let cameraPitch = -0.2;

    const joystickZone = document.getElementById('joystick-zone');
    const joystickKnob = document.getElementById('joystick-knob');
    let joystickTouchId = null;
    let joystickCenter = { x: 0, y: 0 };
    const maxRadius = 45;

    joystickZone.addEventListener('touchstart', (e) => {
      if (isGameWon || joystickTouchId !== null) return;
      const touch = e.changedTouches[0];
      joystickTouchId = touch.identifier;
      const rect = joystickZone.getBoundingClientRect();
      joystickCenter = { x: rect.left + rect.width / 2, y: rect.top + rect.height / 2 };
      updateJoystick(touch);
    }, { passive: false });

    window.addEventListener('touchmove', (e) => {
      if (isGameWon) return;
      for (let i = 0; i < e.changedTouches.length; i++) {
        if (e.changedTouches[i].identifier === joystickTouchId) updateJoystick(e.changedTouches[i]);
      }
    }, { passive: false });

    function resetJoystick() {
      joystickTouchId = null;
      joystickData.x = 0;
      joystickData.y = 0;
      joystickKnob.style.transform = `translate(-50%, -50%)`;
    }

    window.addEventListener('touchend', (e) => {
      for (let i = 0; i < e.changedTouches.length; i++) {
        if (e.changedTouches[i].identifier === joystickTouchId) resetJoystick();
      }
    });

    function updateJoystick(touch) {
      const dx = touch.clientX - joystickCenter.x;
      const dy = touch.clientY - joystickCenter.y;
      
      const dist = Math.hypot(dx, dy);
      const angle = Math.atan2(dy, dx);
      const intensity = Math.min(dist, maxRadius) / maxRadius;

      joystickData.x = Math.cos(angle) * intensity;
      joystickData.y = Math.sin(angle) * intensity;

      const knobX = Math.cos(angle) * (intensity * maxRadius);
      const knobY = Math.sin(angle) * (intensity * maxRadius);
      joystickKnob.style.transform = `translate(calc(-50% + ${knobX}px), calc(-50% + ${knobY}px))`;
    }

    // Camera Swipe
    const cameraZone = document.getElementById('camera-touch-zone');
    let cameraTouchId = null;
    let lastTouchX = 0, lastTouchY = 0;

    cameraZone.addEventListener('touchstart', (e) => {
      if (isGameWon || cameraTouchId !== null) return;
      const touch = e.changedTouches[0];
      cameraTouchId = touch.identifier;
      lastTouchX = touch.clientX;
      lastTouchY = touch.clientY;
    });

    window.addEventListener('touchmove', (e) => {
      if (isGameWon) return;
      for (let i = 0; i < e.changedTouches.length; i++) {
        const touch = e.changedTouches[i];
        if (touch.identifier === cameraTouchId) {
          const deltaX = touch.clientX - lastTouchX;
          const deltaY = touch.clientY - lastTouchY;

          cameraYaw -= deltaX * 0.005;
          cameraPitch -= deltaY * 0.005;
          cameraPitch = Math.max(-1.1, Math.min(0.3, cameraPitch));

          lastTouchX = touch.clientX;
          lastTouchY = touch.clientY;
        }
      }
    });

    const resetCameraTouch = (e) => {
      for (let i = 0; i < e.changedTouches.length; i++) {
        if (e.changedTouches[i].identifier === cameraTouchId) cameraTouchId = null;
      }
    };
    window.addEventListener('touchend', resetCameraTouch);

    // Jump
    document.getElementById('jump-btn').addEventListener('touchstart', (e) => {
      e.preventDefault();
      if (!isGameWon && isGrounded) {
        velocityY = jumpForce;
        isGrounded = false;
      }
    });

    // --- 7. GAME LOOP ---
    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame(animate);
      const deltaTime = Math.min(clock.getDelta(), 0.1);

      // Update timer UI during active play
      if (!isGameWon) {
        document.getElementById('timer-display').innerText = `TIME: ${formatTime(Date.now() - startTime)}`;
      }

      // Movement Calculations (Only process if game isn't won)
      if (!isGameWon) {
        const forward = new THREE.Vector3();
        camera.getWorldDirection(forward);
        forward.y = 0;
        forward.normalize();

        const right = new THREE.Vector3();
        right.crossVectors(forward, new THREE.Vector3(0, 1, 0)).normalize();

        const moveDirection = new THREE.Vector3();
        if (Math.abs(joystickData.x) > 0.05 || Math.abs(joystickData.y) > 0.05) {
          moveDirection.addScaledVector(forward, -joystickData.y);
          moveDirection.addScaledVector(right, joystickData.x);
          if (moveDirection.lengthSq() > 0) moveDirection.normalize();
        }

        player.position.x += moveDirection.x * moveSpeed * deltaTime;
        player.position.z += moveDirection.z * moveSpeed * deltaTime;

        if (moveDirection.lengthSq() > 0) {
          player.rotation.y = Math.atan2(moveDirection.x, moveDirection.z);
        }

        // Physics & Vertical Movement
        velocityY += gravity * deltaTime;
        player.position.y += velocityY * deltaTime;
      }

      isGrounded = false;
      const playerBox = new THREE.Box3().setFromObject(player);

      for (let i = 0; i < platforms.length; i++) {
        const plat = platforms[i];
        const platBox = new THREE.Box3().setFromObject(plat);

        if (playerBox.intersectsBox(platBox)) {
          if (velocityY <= 0 && (player.position.y - playerGeo.parameters.height / 2) >= (plat.position.y + 0.2)) {
            player.position.y = plat.position.y + 0.5 + playerGeo.parameters.height / 2;
            velocityY = 0;
            isGrounded = true;

            // Check if landed on final victory platform
            if (plat.userData.isWin) {
              triggerWin();
            }
            // Checkpoint Logic
            else if (plat.userData.isCheckpoint && plat.userData.stageNum > currentStage) {
              currentStage = plat.userData.stageNum;
              spawnPos.set(plat.position.x, plat.position.y + 2, plat.position.z);
              document.getElementById('stage-banner').innerText = `STAGE ${currentStage}`;
            }
            break;
          }
        }
      }

      // Reset on Lava Fall
      if (player.position.y < lava.position.y + 1) {
        respawnPlayer();
      }

      // Camera Position
      const cameraDistance = 8;
      const cameraOffset = new THREE.Vector3(0, 0, cameraDistance);
      const euler = new THREE.Euler(cameraPitch, cameraYaw, 0, 'YXZ');
      cameraOffset.applyEuler(euler);

      camera.position.copy(player.position).add(cameraOffset);
      camera.lookAt(player.position.x, player.position.y + 1, player.position.z);

      renderer.render(scene, camera);
    }

    animate();

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
Sorry, the code loaded in wrong. Please use the file. 
