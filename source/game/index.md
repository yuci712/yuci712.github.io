---
title: 摸鱼时间
date: 2026-02-02 12:00:00
---

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.muyu-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 80vh;
  background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%);
  padding: 20px;
  font-family: 'Arial', sans-serif;
}
.title {
  font-size: 2.5rem;
  color: #8B4513;
  margin-bottom: 20px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
}
.merit-display {
  font-size: 3rem;
  color: #D2691E;
  margin-bottom: 30px;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}
.muyu-wrapper {
  position: relative;
  cursor: pointer;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}
.muyu {
  width: 250px;
  height: 250px;
  background: linear-gradient(145deg, #8B4513, #A0522D);
  border-radius: 50%;
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.1s;
  position: relative;
  overflow: hidden;
}
.muyu:active {
  transform: scale(0.95);
}
.muyu::before {
  content: '木鱼';
  font-size: 3rem;
  color: #FFE4B5;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}
.muyu::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle at 30% 30%, rgba(255,255,255,0.3), transparent);
  border-radius: 50%;
}
.merit-text {
  position: absolute;
  font-size: 2rem;
  font-weight: bold;
  color: #FF6347;
  pointer-events: none;
  animation: float-up 1s ease-out forwards;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}
@keyframes float-up {
  0% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
  100% {
    opacity: 0;
    transform: translateY(-100px) scale(1.5);
  }
}
.stats {
  margin-top: 30px;
  text-align: center;
  color: #8B4513;
  font-size: 1.1rem;
}
.stats div {
  margin: 8px 0;
}
.reset-btn {
  margin-top: 20px;
  padding: 12px 30px;
  background: #8B4513;
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 1rem;
  cursor: pointer;
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
  transition: all 0.3s;
}
.reset-btn:hover {
  background: #A0522D;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0,0,0,0.3);
}
.tips {
  margin-top: 20px;
  color: #8B4513;
  font-size: 0.9rem;
  opacity: 0.8;
}
</style>

<div class="muyu-container">
  <div class="title">🙏 电子木鱼 🙏</div>
  <div class="merit-display">功德: <span id="merit">0</span></div>
  
  <div class="muyu-wrapper" id="muyu">
    <div class="muyu"></div>
  </div>
  
  <div class="stats">
    <div>今日敲击: <span id="todayCount">0</span> 次</div>
    <div>累计敲击: <span id="totalCount">0</span> 次</div>
  </div>
  
  <button class="reset-btn" onclick="resetMerit()">重置功德</button>
  
  <div class="tips">💡 点击木鱼，功德+1，心诚则灵</div>
</div>

<script>
// 音效数据 (使用 Web Audio API 生成木鱼声音)
let audioContext;
let merit = 0;
let todayCount = 0;
let totalCount = 0;

// 从 localStorage 加载数据
function loadData() {
  const saved = localStorage.getItem('muyuData');
  if (saved) {
    const data = JSON.parse(saved);
    const today = new Date().toDateString();
    
    if (data.date === today) {
      todayCount = data.todayCount || 0;
    }
    
    totalCount = data.totalCount || 0;
    merit = totalCount;
  }
  updateDisplay();
}

// 保存数据
function saveData() {
  const data = {
    date: new Date().toDateString(),
    todayCount: todayCount,
    totalCount: totalCount
  };
  localStorage.setItem('muyuData', JSON.stringify(data));
}

// 更新显示
function updateDisplay() {
  document.getElementById('merit').textContent = merit;
  document.getElementById('todayCount').textContent = todayCount;
  document.getElementById('totalCount').textContent = totalCount;
}

// 播放木鱼音效
function playSound() {
  if (!audioContext) {
    audioContext = new (window.AudioContext || window.webkitAudioContext)();
  }
  
  const oscillator = audioContext.createOscillator();
  const gainNode = audioContext.createGain();
  
  oscillator.connect(gainNode);
  gainNode.connect(audioContext.destination);
  
  oscillator.frequency.value = 800;
  oscillator.type = 'sine';
  
  gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
  gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
  
  oscillator.start(audioContext.currentTime);
  oscillator.stop(audioContext.currentTime + 0.1);
}

// 显示功德+1动画
function showMeritText(x, y) {
  const text = document.createElement('div');
  text.className = 'merit-text';
  text.textContent = '功德 +1';
  text.style.left = x + 'px';
  text.style.top = y + 'px';
  
  document.body.appendChild(text);
  
  setTimeout(() => {
    text.remove();
  }, 1000);
}

// 敲击木鱼
function knockMuyu(event) {
  merit++;
  todayCount++;
  totalCount++;
  
  updateDisplay();
  saveData();
  playSound();
  
  // 获取点击位置
  const rect = event.target.getBoundingClientRect();
  const x = event.clientX || (rect.left + rect.width / 2);
  const y = event.clientY || (rect.top + rect.height / 2);
  
  showMeritText(x, y);
}

// 重置功德
function resetMerit() {
  if (confirm('确定要重置功德吗？')) {
    merit = 0;
    todayCount = 0;
    totalCount = 0;
    updateDisplay();
    saveData();
  }
}

// 初始化
document.getElementById('muyu').addEventListener('click', knockMuyu);
document.getElementById('muyu').addEventListener('touchstart', (e) => {
  e.preventDefault();
  knockMuyu(e.touches[0]);
});

loadData();
</script>

> 🙏 木鱼一敲，烦恼全消。功德+1，心诚则灵！
