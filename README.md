<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>方塊跳躍遊戲</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: linear-gradient(180deg, #a0d8ef 0%, #fff 100%);
    font-family: "Noto Sans TC", -apple-system, BlinkMacSystemFont, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }
  #gameContainer {
    position: relative;
  }
  #gameCanvas {
    display: block;
    background: linear-gradient(180deg, #e0f7fa 0%, #b2ebf2 100%);
    border: 4px solid #00796b;
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  }
  #score {
    position: absolute;
    top: 15px;
    left: 15px;
    font-size: 24px;
    color: #004d40;
    font-weight: bold;
    text-shadow: 2px 2px 4px rgba(255, 255, 255, 0.8);
  }
  #highScore {
    position: absolute;
    top: 45px;
    left: 15px;
    font-size: 18px;
    color: #00695c;
    font-weight: bold;
    text-shadow: 2px 2px 4px rgba(255, 255, 255, 0.8);
  }
  #gameOver {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(255, 255, 255, 0.95);
    padding: 40px;
    border-radius: 20px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.4);
    text-align: center;
    display: none;
  }
  #gameOver h2 {
    font-size: 36px;
    color: #d32f2f;
    margin: 0 0 10px 0;
  }
  #gameOver p {
    font-size: 20px;
    color: #004d40;
    margin: 10px 0;
  }
  button {
    font-size: 20px;
    background: linear-gradient(135deg, #00796b 0%, #004d40 100%);
    color: white;
    border: none;
    padding: 12px 30px;
    border-radius: 10px;
    cursor: pointer;
    margin-top: 15px;
    transition: transform 0.2s, box-shadow 0.2s;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
  }
  button:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 15px rgba(0, 0, 0, 0.4);
  }
  button:active {
    transform: translateY(0);
  }
  #instructions {
    position: absolute;
    bottom: 15px;
    left: 50%;
    transform: translateX(-50%);
    color: #004d40;
    font-size: 16px;
    text-align: center;
    text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
  }
</style>
</head>
<body>
<div id="gameContainer">
  <div id="score">分數：0</div>
  <div id="highScore">最高分：0</div>
  <div id="instructions">點擊畫面或按空白鍵跳躍</div>
  <div id="gameOver">
    <h2>遊戲結束！</h2>
    <p id="finalScore">最終分數：0</p>
    <p id="newRecord" style="color: #f57c00; display: none;">🎉 新紀錄！</p>
    <button onclick="restart()">重新開始</button>
  </div>
  <canvas id="gameCanvas" width="800" height="400"></canvas>
</div>
<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

let player = { 
  x: 80, 
  y: 300, 
  w: 30, 
  h: 30, 
  dy: 0, 
  jumpPower: 13, 
  onGround: true,
  rotation: 0
};

let gravity = 0.7;
let obstacles = [];
let score = 0;
let highScore = 0;
let gameOver = false;
let speed = 5;
let frameCount = 0;
let clouds = [];

// 初始化雲朵
for (let i = 0; i < 5; i++) {
  clouds.push({
    x: Math.random() * canvas.width,
    y: Math.random() * 100 + 20,
    w: 60 + Math.random() * 40,
    speed: 0.3 + Math.random() * 0.5
  });
}

function drawPlayer() {
  ctx.save();
  ctx.translate(player.x + player.w / 2, player.y + player.h / 2);
  
  if (!player.onGround) {
    player.rotation += 0.15;
    ctx.rotate(player.rotation);
  } else {
    player.rotation = 0;
  }
  
  // 漸層效果
  const gradient = ctx.createLinearGradient(-player.w/2, -player.h/2, player.w/2, player.h/2);
  gradient.addColorStop(0, "#00897b");
  gradient.addColorStop(1, "#00695c");
  
  ctx.fillStyle = gradient;
  ctx.fillRect(-player.w / 2, -player.h / 2, player.w, player.h);
  
  // 眼睛
  ctx.fillStyle = "white";
  ctx.fillRect(-8, -8, 6, 6);
  ctx.fillRect(2, -8, 6, 6);
  ctx.fillStyle = "black";
  ctx.fillRect(-6, -6, 3, 3);
  ctx.fillRect(4, -6, 3, 3);
  
  ctx.restore();
}

function drawObstacles() {
  obstacles.forEach(obs => {
    const gradient = ctx.createLinearGradient(obs.x, obs.y, obs.x, obs.y + obs.h);
    gradient.addColorStop(0, "#e53935");
    gradient.addColorStop(1, "#c62828");
    
    ctx.fillStyle = gradient;
    ctx.fillRect(obs.x, obs.y, obs.w, obs.h);
    
    // 邊框
    ctx.strokeStyle = "#b71c1c";
    ctx.lineWidth = 2;
    ctx.strokeRect(obs.x, obs.y, obs.w, obs.h);
  });
}

function drawClouds() {
  ctx.fillStyle = "rgba(255, 255, 255, 0.7)";
  clouds.forEach(cloud => {
    ctx.beginPath();
    ctx.arc(cloud.x, cloud.y, 15, 0, Math.PI * 2);
    ctx.arc(cloud.x + 20, cloud.y, 20, 0, Math.PI * 2);
    ctx.arc(cloud.x + 40, cloud.y, 15, 0, Math.PI * 2);
    ctx.fill();
    
    cloud.x -= cloud.speed;
    if (cloud.x < -60) {
      cloud.x = canvas.width + 60;
      cloud.y = Math.random() * 100 + 20;
    }
  });
}

function drawGround() {
  // 地面漸層
  const gradient = ctx.createLinearGradient(0, 350, 0, 400);
  gradient.addColorStop(0, "#4db6ac");
  gradient.addColorStop(1, "#26a69a");
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 350, canvas.width, 50);
  
  // 草地效果
  ctx.fillStyle = "#00897b";
  for (let i = 0; i < canvas.width; i += 15) {
    ctx.fillRect(i, 350, 2, 8);
  }
}

function update() {
  if (gameOver) return;
  
  frameCount++;
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // 背景元素
  drawClouds();
  drawGround();
  
  // 玩家跳躍邏輯
  player.y += player.dy;
  if (player.y + player.h < 350) {
    player.dy += gravity;
    player.onGround = false;
  } else {
    player.y = 350 - player.h;
    player.dy = 0;
    player.onGround = true;
  }
  
  // 更新障礙物
  obstacles.forEach(obs => obs.x -= speed);
  
  // 生成新障礙物
  if (obstacles.length === 0 || obstacles[obstacles.length - 1].x < 550) {
    let height = 25 + Math.random() * 40;
    obstacles.push({ 
      x: canvas.width, 
      y: 350 - height, 
      w: 20 + Math.random() * 25, 
      h: height 
    });
  }
  
  // 碰撞檢測（更精確）
  obstacles.forEach(obs => {
    if (
      player.x + 2 < obs.x + obs.w &&
      player.x + player.w - 2 > obs.x &&
      player.y + 2 < obs.y + obs.h &&
      player.y + player.h > obs.y
    ) {
      gameOver = true;
      endGame();
    }
  });
  
  // 移除離開畫面的障礙物
  obstacles = obstacles.filter(obs => obs.x + obs.w > 0);
  
  // 畫出障礙物和玩家
  drawObstacles();
  drawPlayer();
  
  // 加分與加速
  score++;
  if (frameCount % 300 === 0 && speed < 10) {
    speed += 0.3;
  }
  
  document.getElementById("score").textContent = "分數：" + Math.floor(score / 10);
  
  requestAnimationFrame(update);
}

function jump() {
  if (player.onGround && !gameOver) {
    player.dy = -player.jumpPower;
    player.onGround = false;
  }
}

function endGame() {
  const finalScore = Math.floor(score / 10);
  document.getElementById("finalScore").textContent = "最終分數：" + finalScore;
  
  if (finalScore > highScore) {
    highScore = finalScore;
    document.getElementById("highScore").textContent = "最高分：" + highScore;
    document.getElementById("newRecord").style.display = "block";
  } else {
    document.getElementById("newRecord").style.display = "none";
  }
  
  document.getElementById("gameOver").style.display = "block";
}

function restart() {
  obstacles = [];
  score = 0;
  speed = 5;
  frameCount = 0;
  player.y = 300;
  player.dy = 0;
  player.rotation = 0;
  player.onGround = true;
  gameOver = false;
  document.getElementById("gameOver").style.display = "none";
  update();
}

// 控制
window.addEventListener("touchstart", jump);
window.addEventListener("mousedown", jump);
window.addEventListener("keydown", (e) => {
  if (e.code === "Space") {
    e.preventDefault();
    jump();
  }
});

update();
</script>
</body>
</html>
