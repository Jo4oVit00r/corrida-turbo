<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Corrida 2D</title>
  <style>
    body {
      background: #111;
      color: white;
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 0;
    }
    canvas {
      background: #333;
      display: block;
      margin: 20px auto;
      border: 2px solid #fff;
    }
    #score {
      font-size: 20px;
      margin-top: 10px;
    }
    #menu {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(0,0,0,0.8);
      padding: 20px;
      border: 2px solid #fff;
      border-radius: 10px;
      display: none;
    }
    button {
      padding: 10px 20px;
      margin-top: 10px;
      font-size: 16px;
      cursor: pointer;
      border: none;
      border-radius: 5px;
      background: #ff4444;
      color: white;
    }
  </style>
</head>
<body>

<h1>🚗 Corrida 2D 🚧</h1>
<div id="score">Pontuação: 0</div>
<canvas id="gameCanvas" width="400" height="600"></canvas>

<div id="menu">
  <h2 id="gameOverText">Fim de Jogo!</h2>
  <p id="finalScore"></p>
  <button onclick="startGame()">Jogar Novamente</button>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const scoreDisplay = document.getElementById("score");
const menu = document.getElementById("menu");
const finalScore = document.getElementById("finalScore");

const roadWidth = 200;
const laneCount = 2;

let player, obstacles, keys, score, gameRunning, obstacleInterval, speedFactor;
let roadLines = [];

function resetGame() {
  player = {
    x: canvas.width / 2 - 20,
    y: canvas.height - 100,
    width: 40,
    height: 80,
    speed: 5
  };
  obstacles = [];
  keys = {};
  score = 0;
  speedFactor = 1;
  roadLines = [];
  for (let i = 0; i < 20; i++) {
    roadLines.push({ x: canvas.width/2, y: i * 40 });
  }
}

function createObstacle() {
  const laneWidth = roadWidth / laneCount;
  const lane = Math.floor(Math.random() * laneCount);
  const colors = ["blue", "green", "yellow", "purple", "orange"];
  obstacles.push({
    x: canvas.width / 2 - roadWidth / 2 + lane * laneWidth + 10,
    y: -100,
    width: laneWidth - 20,
    height: 80,
    speed: 3 * speedFactor,
    color: colors[Math.floor(Math.random() * colors.length)]
  });
}

function drawRoad() {
  ctx.fillStyle = "#555";
  const roadX = canvas.width / 2 - roadWidth / 2;
  ctx.fillRect(roadX, 0, roadWidth, canvas.height);

  // Linhas de movimento
  ctx.strokeStyle = "#FFF";
  ctx.lineWidth = 4;
  for (let line of roadLines) {
    ctx.beginPath();
    ctx.moveTo(line.x, line.y);
    ctx.lineTo(line.x, line.y + 20);
    ctx.stroke();
    line.y += 10;
    if (line.y > canvas.height) line.y = -20;
  }
}

function drawPlayer() {
  ctx.fillStyle = "red";
  ctx.fillRect(player.x, player.y, player.width, player.height);
}

function drawObstacles() {
  for (let obs of obstacles) {
    ctx.fillStyle = obs.color;
    ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
  }
}

function moveObstacles() {
  for (let obs of obstacles) {
    obs.y += obs.speed;
  }
  obstacles = obstacles.filter(obs => obs.y < canvas.height);
}

function detectCollision() {
  for (let obs of obstacles) {
    if (
      player.x < obs.x + obs.width &&
      player.x + player.width > obs.x &&
      player.y < obs.y + obs.height &&
      player.y + player.height > obs.y
    ) {
      endGame();
    }
  }
}

function updatePlayer() {
  if (keys["ArrowLeft"] && player.x > canvas.width / 2 - roadWidth / 2) {
    player.x -= player.speed;
  }
  if (keys["ArrowRight"] && player.x + player.width < canvas.width / 2 + roadWidth / 2) {
    player.x += player.speed;
  }
}

function gameLoop() {
  if (!gameRunning) return;

  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawRoad();
  drawPlayer();
  drawObstacles();
  updatePlayer();
  moveObstacles();
  detectCollision();

  // Aumentar pontuação
  score++;
  scoreDisplay.textContent = "Pontuação: " + score;

  // Aumentar dificuldade
  if (score % 500 === 0) {
    speedFactor += 0.2;
  }

  requestAnimationFrame(gameLoop);
}

function startGame() {
  resetGame();
  menu.style.display = "none";
  gameRunning = true;
  obstacleInterval = setInterval(createObstacle, 1500);
  gameLoop();
}

function endGame() {
  gameRunning = false;
  clearInterval(obstacleInterval);
  finalScore.textContent = "Sua pontuação: " + score;
  menu.style.display = "block";
}

// Controles
document.addEventListener("keydown", e => {
  keys[e.key] = true;
});
document.addEventListener("keyup", e => {
  keys[e.key] = false;
});

// Iniciar automaticamente
startGame();
</script>

</body>
</html>
