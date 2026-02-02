---
title: 摸鱼时间
date: 2026-02-02 12:00:00
---

<style>
.game-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  font-family: Arial, sans-serif;
}
#gameCanvas {
  border: 2px solid #4A90E2;
  background: linear-gradient(to bottom, #87CEEB 0%, #E0F6FF 100%);
  display: block;
  margin: 20px auto;
  cursor: pointer;
}
.score-board {
  font-size: 24px;
  margin: 10px 0;
  color: #4A90E2;
  font-weight: bold;
}
.game-info {
  color: #666;
  margin: 10px 0;
  text-align: center;
  font-size: 14px;
}
</style>

<div class="game-wrapper">
  <div class="score-board">🐟 得分: <span id="score">0</span></div>
  <canvas id="gameCanvas" width="400" height="600"></canvas>
  <div class="game-info">点击画布 或 按空格/↑/W/J 让小鱼游动</div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreElement = document.getElementById('score');

let score = 0;
let isGameOver = false;
let gameStarted = false;

// 小鱼
const fish = {
  x: 80,
  y: canvas.height / 2,
  width: 30,
  height: 24,
  velocity: 0,
  gravity: 0.4,
  jump: -7
};

// 管道数组
let pipes = [];
const pipeWidth = 60;
const pipeGap = 180;
const pipeSpeed = 2;
let frameCount = 0;

// 键盘和鼠标控制
document.addEventListener('keydown', (e) => {
  if (e.code === 'Space' || e.code === 'ArrowUp' || e.code === 'KeyW' || e.code === 'KeyJ') {
    e.preventDefault();
    if (!gameStarted) {
      gameStarted = true;
    }
    if (!isGameOver) {
      fish.velocity = fish.jump;
    } else {
      restart();
    }
  }
});

canvas.addEventListener('click', () => {
  if (!gameStarted) {
    gameStarted = true;
  }
  if (!isGameOver) {
    fish.velocity = fish.jump;
  } else {
    restart();
  }
});

// 创建管道
function createPipe() {
  const minHeight = 50;
  const maxHeight = canvas.height - pipeGap - minHeight;
  const height = Math.floor(Math.random() * (maxHeight - minHeight + 1)) + minHeight;
  
  pipes.push({
    x: canvas.width,
    topHeight: height,
    bottomY: height + pipeGap,
    passed: false
  });
}

// 绘制小鱼
function drawFish() {
  ctx.save();
  ctx.translate(fish.x + fish.width / 2, fish.y + fish.height / 2);
  
  // 根据速度旋转小鱼
  const rotation = Math.min(Math.max(fish.velocity * 3, -30), 90) * Math.PI / 180;
  ctx.rotate(rotation);
  
  // 鱼身体
  ctx.fillStyle = '#FFA500';
  ctx.beginPath();
  ctx.ellipse(0, 0, fish.width / 2, fish.height / 2, 0, 0, Math.PI * 2);
  ctx.fill();
  
  // 鱼尾巴
  ctx.fillStyle = '#FF8C00';
  ctx.beginPath();
  ctx.moveTo(-fish.width / 2, 0);
  ctx.lineTo(-fish.width / 2 - 10, -8);
  ctx.lineTo(-fish.width / 2 - 10, 8);
  ctx.closePath();
  ctx.fill();
  
  // 鱼眼睛
  ctx.fillStyle = '#FFF';
  ctx.beginPath();
  ctx.arc(fish.width / 4, -fish.height / 6, 4, 0, Math.PI * 2);
  ctx.fill();
  
  ctx.fillStyle = '#000';
  ctx.beginPath();
  ctx.arc(fish.width / 4 + 1, -fish.height / 6, 2, 0, Math.PI * 2);
  ctx.fill();
  
  ctx.restore();
}

// 绘制管道（水草）
function drawPipes() {
  pipes.forEach(pipe => {
    // 上方水草
    const gradient1 = ctx.createLinearGradient(pipe.x, 0, pipe.x + pipeWidth, 0);
    gradient1.addColorStop(0, '#2E7D32');
    gradient1.addColorStop(0.5, '#4CAF50');
    gradient1.addColorStop(1, '#2E7D32');
    ctx.fillStyle = gradient1;
    ctx.fillRect(pipe.x, 0, pipeWidth, pipe.topHeight);
    
    // 上方水草顶部
    ctx.fillStyle = '#1B5E20';
    ctx.fillRect(pipe.x - 5, pipe.topHeight - 20, pipeWidth + 10, 20);
    
    // 下方水草
    const gradient2 = ctx.createLinearGradient(pipe.x, pipe.bottomY, pipe.x + pipeWidth, canvas.height);
    gradient2.addColorStop(0, '#2E7D32');
    gradient2.addColorStop(0.5, '#4CAF50');
    gradient2.addColorStop(1, '#2E7D32');
    ctx.fillStyle = gradient2;
    ctx.fillRect(pipe.x, pipe.bottomY, pipeWidth, canvas.height - pipe.bottomY);
    
    // 下方水草顶部
    ctx.fillStyle = '#1B5E20';
    ctx.fillRect(pipe.x - 5, pipe.bottomY, pipeWidth + 10, 20);
  });
}

// 绘制气泡
function drawBubbles() {
  if (frameCount % 20 === 0) {
    ctx.fillStyle = 'rgba(255, 255, 255, 0.3)';
    for (let i = 0; i < 3; i++) {
      const x = Math.random() * canvas.width;
      const y = canvas.height - 20;
      ctx.beginPath();
      ctx.arc(x, y, 3, 0, Math.PI * 2);
      ctx.fill();
    }
  }
}

// 更新游戏
function update() {
  if (!gameStarted || isGameOver) return;
  
  frameCount++;
  
  // 更新小鱼
  fish.velocity += fish.gravity;
  fish.y += fish.velocity;
  
  // 检查边界
  if (fish.y + fish.height > canvas.height || fish.y < 0) {
    gameOver();
    return;
  }
  
  // 创建新管道
  if (frameCount % 90 === 0) {
    createPipe();
  }
  
  // 更新管道
  pipes.forEach((pipe, index) => {
    pipe.x -= pipeSpeed;
    
    // 移除屏幕外的管道
    if (pipe.x + pipeWidth < 0) {
      pipes.splice(index, 1);
    }
    
    // 计分
    if (!pipe.passed && pipe.x + pipeWidth < fish.x) {
      pipe.passed = true;
      score++;
      scoreElement.textContent = score;
    }
    
    // 碰撞检测
    if (
      fish.x + fish.width > pipe.x &&
      fish.x < pipe.x + pipeWidth &&
      (fish.y < pipe.topHeight || fish.y + fish.height > pipe.bottomY)
    ) {
      gameOver();
    }
  });
}

// 绘制游戏
function draw() {
  // 清空画布（天空背景）
  const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
  gradient.addColorStop(0, '#87CEEB');
  gradient.addColorStop(1, '#E0F6FF');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  
  // 绘制水波纹效果
  ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
  ctx.lineWidth = 2;
  for (let i = 0; i < 3; i++) {
    ctx.beginPath();
    ctx.moveTo(0, 20 + i * 30);
    for (let x = 0; x < canvas.width; x += 20) {
      ctx.lineTo(x, 20 + i * 30 + Math.sin((x + frameCount * 2) * 0.05) * 5);
    }
    ctx.stroke();
  }
  
  drawBubbles();
  drawPipes();
  drawFish();
  
  if (!gameStarted) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = '#FFF';
    ctx.font = 'bold 30px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('🐟 小鱼快游 🐟', canvas.width / 2, canvas.height / 2 - 40);
    ctx.font = '18px Arial';
    ctx.fillText('点击或按空格开始', canvas.width / 2, canvas.height / 2 + 10);
  }
  
  if (isGameOver) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = '#FFF';
    ctx.font = 'bold 36px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('游戏结束!', canvas.width / 2, canvas.height / 2 - 40);
    ctx.font = '20px Arial';
    ctx.fillText('得分: ' + score, canvas.width / 2, canvas.height / 2);
    ctx.font = '16px Arial';
    ctx.fillText('点击或按空格重新开始', canvas.width / 2, canvas.height / 2 + 40);
  }
}

// 游戏结束
function gameOver() {
  isGameOver = true;
}

// 重新开始
function restart() {
  isGameOver = false;
  gameStarted = false;
  score = 0;
  fish.y = canvas.height / 2;
  fish.velocity = 0;
  pipes = [];
  frameCount = 0;
  scoreElement.textContent = score;
}

// 游戏循环
function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

> 🐟 小鱼快游！点击或按空格让小鱼穿过水草，看你能得多少分！
