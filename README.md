# car-world-game
لعبة سيارات جميلة من تصميمي 💖
<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Car World</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      background: #111;
      overflow: hidden;
      font-family: Arial, sans-serif;
    }
    canvas {
      display: block;
      margin: auto;
      background: linear-gradient(to top, #444, #222);
      border: 2px solid white;
      box-shadow: 0 0 30px rgba(255,255,255,0.5);
      transition: background 0.5s;
    }
    #menu, #gameOverMenu, #winMenu, #startScreen {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.9);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: white;
      z-index: 10;
      animation: fadeIn 1s ease;
    }
    button, select {
      padding: 10px;
      margin: 10px;
      font-size: 18px;
      cursor: pointer;
      transition: transform 0.2s;
    }
    button:hover, select:hover {
      transform: scale(1.1);
    }
    #score {
      position: absolute;
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
      color: white;
      font-size: 24px;
      font-weight: bold;
      z-index: 5;
    }
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }
  </style>
</head>
<body>
  <audio id="gameMusic" loop autoplay>
    <source src="https://cdn.pixabay.com/audio/2023/02/21/audio_9adf670370.mp3" type="audio/mpeg">
    متصفحك لا يدعم تشغيل الصوت.
  </audio>
  <div id="score">النقاط: 0 | المرحلة: 1</div>
  <canvas id="gameCanvas" width="400" height="600"></canvas>

  <div id="startScreen">
    <h2>اختر اللغة</h2>
    <select id="languageSelect">
      <option value="ar">العربية</option>
      <option value="en">English</option>
      <option value="fr">Français</option>
      <option value="es">Español</option>
      <option value="de">Deutsch</option>
      <option value="zh">中文</option>
      <option value="ru">Русский</option>
      <option value="hi">हिन्दी</option>
    </select>
    <button onclick="goToMenu()">ابدأ</button>
  </div>

  <div id="menu" style="display:none">
    <h2>اختر الخلفية</h2>
    <select id="backgroundSelect">
      <option value="#222">ليلية</option>
      <option value="#0a0">غابة</option>
      <option value="#00a">بحرية</option>
      <option value="#a60">صحراء</option>
    </select>
    <h2>اختر لون السيارة</h2>
    <select id="carColorSelect">
      <option value="red">أحمر</option>
      <option value="blue">أزرق</option>
      <option value="green">أخضر</option>
      <option value="black">أسود</option>
    </select>
    <button onclick="startGame()">ابدأ اللعب</button>
  </div>

  <div id="gameOverMenu" style="display:none">
    <h2>انتهت اللعبة</h2>
    <button onclick="restartGame()">إعادة</button>
  </div>

  <div id="winMenu" style="display:none">
    <h2>فزت!</h2>
    <button onclick="restartGame()">إعادة</button>
    <button onclick="nextLevel()">المرحلة التالية</button>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreDisplay = document.getElementById('score');

    let backgroundColor = '#222';
    let carColor = 'red';
    let score = 0;
    let gameOver = false;
    let win = false;
    let speed = 5;
    let level = 1;
    let winScore = 100;

    const car = {
      x: canvas.width / 2 - 20,
      y: canvas.height - 70,
      width: 40,
      height: 60
    };

    const obstacles = [];

    function drawRoad() {
      ctx.fillStyle = backgroundColor;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    function drawCar() {
      ctx.fillStyle = carColor;
      ctx.fillRect(car.x, car.y, car.width, car.height);
      ctx.strokeStyle = 'white';
      ctx.strokeRect(car.x, car.y, car.width, car.height);
    }

    function drawObstacles() {
      ctx.fillStyle = 'yellow';
      obstacles.forEach(obs => {
        ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
        ctx.strokeStyle = 'black';
        ctx.strokeRect(obs.x, obs.y, obs.width, obs.height);
      });
    }

    function updateObstacles() {
      obstacles.forEach(obs => obs.y += speed);
      for (let i = obstacles.length - 1; i >= 0; i--) {
        if (obstacles[i].y > canvas.height) {
          obstacles.splice(i, 1);
          score += 10;
          scoreDisplay.innerText = `النقاط: ${score} | المرحلة: ${level}`;
          if (score >= winScore) {
            win = true;
            endGame();
          }
        }
      }
    }

    function checkCollision() {
      for (const obs of obstacles) {
        if (
          car.x < obs.x + obs.width &&
          car.x + car.width > obs.x &&
          car.y < obs.y + obs.height &&
          car.y + car.height > obs.y
        ) {
          gameOver = true;
          endGame();
        }
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawRoad();
      drawCar();
      drawObstacles();
      ctx.fillStyle = 'white';
      ctx.font = '20px Arial';
      ctx.fillText(`مرحلة ${level} - استمر!`, 10, 30);
    }

    function update() {
      if (!gameOver && !win) {
        updateObstacles();
        checkCollision();
        draw();
        requestAnimationFrame(update);
      }
    }

    function spawnObstacle() {
      const x = Math.random() * (canvas.width - 40);
      obstacles.push({ x, y: -60, width: 40, height: 60 });
    }

    function endGame() {
      if (gameOver) document.getElementById('gameOverMenu').style.display = 'flex';
      else if (win) document.getElementById('winMenu').style.display = 'flex';
    }

    function restartGame() {
      score = 0;
      gameOver = false;
      win = false;
      speed = 5;
      level = 1;
      winScore = 100;
      obstacles.length = 0;
      car.x = canvas.width / 2 - 20;
      scoreDisplay.innerText = `النقاط: ${score} | المرحلة: ${level}`;
      document.getElementById('gameOverMenu').style.display = 'none';
      document.getElementById('winMenu').style.display = 'none';
      update();
    }

    function nextLevel() {
      level++;
      if (level > 20) {
        alert("ألف مبروك! لقد أنهيت جميع المراحل!");
        restartGame();
        return;
      }
      speed += 2;
      winScore += 50;
      score = 0;
      win = false;
      obstacles.length = 0;
      car.x = canvas.width / 2 - 20;
      scoreDisplay.innerText = `النقاط: ${score} | المرحلة: ${level}`;
      document.getElementById('winMenu').style.display = 'none';
      update();
    }

    function goToMenu() {
      document.getElementById('startScreen').style.display = 'none';
      document.getElementById('menu').style.display = 'flex';
    }

    function startGame() {
      backgroundColor = document.getElementById('backgroundSelect').value;
      carColor = document.getElementById('carColorSelect').value;
      document.getElementById('menu').style.display = 'none';
      update();
      setInterval(() => {
        if (!gameOver && !win) spawnObstacle();
      }, 1000);
    }

    document.addEventListener('keydown', e => {
      if (e.key === 'ArrowLeft' && car.x > 0) car.x -= 10;
      if (e.key === 'ArrowRight' && car.x + car.width < canvas.width) car.x += 10;
    });
  </script>
</body>
</html>

<meta name="description" content="لعبة سباق سيارات ممتعة فيها تحديات ومراحل متعددة من تصميم Janaa. العب مجانًا!">
<meta name="keywords" content="Car World, لعبة سيارات, ألعاب سباق, Janaa, العاب مجانية, car racing game">
<meta name="author" content="Janaa Ahmed">
<title>Car World by Janaa</title>
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-XXXXXXX-X"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-XXXXXXX-X');
</script>
تعديل لتحسين SEO
