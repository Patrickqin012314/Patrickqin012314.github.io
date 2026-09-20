# Patrickqin012314.github.io
<img width="710" height="696" alt="background" src="https://github.com/user-attachments/assets/ee275771-450d-487b-83ef-03b9d6fab0cd" /><img width="359" height="337" alt="jellyfish" src="https://github.com/user-attachments/assets/afbe4e9d-b603-413b-801e-d109052a0eef" />
[index.html](https://github.com/user-attachments/files/32434668/index.html)
[game.html](https://github.com/user-attachments/files/32434667/game.html)

<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<title>水母点击小游戏</title>
<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    overflow: hidden;
    width: 100vw;
    height: 100vh;
    font-family: system-ui, "Microsoft YaHei", sans-serif;
    color: #fff;
    user-select: none;
    cursor: crosshair;

    /* 本地海底背景图 */
    background: url("background.jpg") no-repeat center center;
    background-size: cover;
  }

  /* 半透明遮罩，让文字更清楚 */
  body::before {
    content: "";
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.25);
    pointer-events: none;
    z-index: 0;
  }

  #info {
    position: fixed;
    top: 20px;
    left: 20px;
    font-size: 24px;
    font-weight: bold;
    text-shadow: 0 2px 8px rgba(0, 0, 0, 0.8);
    z-index: 10;
  }

  #highScore {
    position: fixed;
    top: 20px;
    right: 20px;
    font-size: 24px;
    font-weight: bold;
    text-shadow: 0 2px 8px rgba(0, 0, 0, 0.8);
    z-index: 10;
  }

  .jelly {
    position: absolute;
    width: 90px;
    height: 90px;
    cursor: pointer;
    will-change: transform, opacity;
    z-index: 2;
  }

  /* 水母爆炸消散效果 */
  .jelly.pop {
    animation: explode 0.45s ease-out forwards;
    pointer-events: none;
  }

  @keyframes explode {
    0% {
      transform: scale(1) rotate(0deg);
      opacity: 1;
      filter: brightness(1);
    }
    30% {
      transform: scale(1.4) rotate(8deg);
      opacity: 0.9;
      filter: brightness(1.8);
    }
    60% {
      transform: scale(1.8) rotate(-6deg);
      opacity: 0.5;
      filter: brightness(1.4);
    }
    100% {
      transform: scale(2.4) rotate(12deg);
      opacity: 0;
      filter: brightness(0.6);
    }
  }

  /* 爆炸光圈 */
  .burst {
    position: absolute;
    border-radius: 50%;
    border: 3px solid rgba(255, 255, 255, 0.9);
    box-shadow:
      0 0 20px rgba(255, 255, 255, 0.9),
      0 0 40px rgba(180, 240, 255, 0.7),
      inset 0 0 20px rgba(255, 255, 255, 0.6);
    pointer-events: none;
    animation: burst 0.45s ease-out forwards;
    z-index: 3;
  }

  @keyframes burst {
    0% {
      width: 20px;
      height: 20px;
      opacity: 1;
      transform: translate(-50%, -50%) scale(0.3);
    }
    100% {
      width: 160px;
      height: 160px;
      opacity: 0;
      transform: translate(-50%, -50%) scale(1);
    }
  }

  /* 爆炸碎片 */
  .particle {
    position: absolute;
    width: 10px;
    height: 10px;
    background: radial-gradient(circle, #ffffff 0%, #b3f0ff 40%, transparent 70%);
    border-radius: 50%;
    pointer-events: none;
    box-shadow: 0 0 12px #b3f0ff;
    animation: particle 0.5s ease-out forwards;
    z-index: 3;
  }

  @keyframes particle {
    0% {
      opacity: 1;
      transform: translate(-50%, -50%) scale(1);
    }
    100% {
      opacity: 0;
      transform: translate(var(--tx), var(--ty)) scale(0.2);
    }
  }

  #gameOver {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 40px;
    display: none;
    text-align: center;
    background: rgba(0, 0, 0, 0.75);
    padding: 30px 50px;
    border-radius: 20px;
    z-index: 20;
    box-shadow: 0 0 40px rgba(100, 200, 255, 0.6);
  }

  #gameOver div {
    margin: 10px 0;
  }

  button {
    margin-top: 20px;
    padding: 12px 28px;
    font-size: 20px;
    border: none;
    border-radius: 10px;
    background: linear-gradient(135deg, #38bdf8, #6366f1);
    color: white;
    cursor: pointer;
    font-weight: bold;
    transition: transform 0.15s;
  }

  button:hover {
    transform: scale(1.06);
  }
</style>
</head>
<body>

<div id="info">
  时间：<span id="timer">60</span> s &nbsp;&nbsp;
  点击得分：<span id="score">0</span>
</div>

<div id="highScore">
  最高分：<span id="hs">0</span>
</div>

<div id="gameOver">
  <div>游戏结束！</div>
  <div>你的总点击数：<span id="finalScore"></span></div>
  <div>历史最高分：<span id="finalHs"></span></div>
  <button onclick="restartGame()">再来一局</button>
</div>

<script>
  const infoTimer = document.getElementById("timer");
  const infoScore = document.getElementById("score");
  const gameOverPanel = document.getElementById("gameOver");
  const finalScore = document.getElementById("finalScore");
  const finalHs = document.getElementById("finalHs");
  const hsDisplay = document.getElementById("hs");

  let score = 0;
  let timeLeft = 60;
  let gameRunning = true;
  let countDownTimer = null;
  let spawnTimer = null;

  // 本地水母图片，请确保文件和 html 同目录
  const jellySrc = "jellyfish.png";

  // 点击音效
  let audioCtx;
  function playPopSound() {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }

    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.connect(gain);
    gain.connect(audioCtx.destination);

    osc.type = "sine";
    osc.frequency.setValueAtTime(700, audioCtx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(180, audioCtx.currentTime + 0.3);

    gain.gain.setValueAtTime(0.35, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.3);

    osc.start();
    osc.stop(audioCtx.currentTime + 0.3);
  }

  // 最高分读取
  let highScore = localStorage.getItem("jellyHighScore") || 0;
  hsDisplay.innerText = highScore;

  // 创建爆炸效果
  function createBurst(x, y) {
    // 光圈
    const burst = document.createElement("div");
    burst.className = "burst";
    burst.style.left = x + "px";
    burst.style.top = y + "px";
    document.body.appendChild(burst);
    setTimeout(() => burst.remove(), 500);

    // 碎片粒子
    const count = 10;
    for (let i = 0; i < count; i++) {
      const p = document.createElement("div");
      p.className = "particle";
      p.style.left = x + "px";
      p.style.top = y + "px";

      const angle = (Math.PI * 2 * i) / count + Math.random() * 0.6;
      const distance = 70 + Math.random() * 50;
      const tx = Math.cos(angle) * distance;
      const ty = Math.sin(angle) * distance;

      p.style.setProperty("--tx", tx + "px");
      p.style.setProperty("--ty", ty + "px");
      document.body.appendChild(p);
      setTimeout(() => p.remove(), 550);
    }
  }

  // 生成水母
  function spawnJelly() {
    if (!gameRunning) return;

    const jelly = document.createElement("img");
    jelly.className = "jelly";
    jelly.src = jellySrc;

    // 图片加载失败时的兜底
    jelly.onerror = function () {
      jelly.style.display = "none";
    };

    const maxX = window.innerWidth - 120;
    const maxY = window.innerHeight - 120;
    let x = 60 + Math.random() * maxX;
    let y = 80 + Math.random() * maxY;

    jelly.style.left = x + "px";
    jelly.style.top = y + "px";

    // 随机大小
    const size = 70 + Math.random() * 40;
    jelly.style.width = size + "px";
    jelly.style.height = size + "px";

    document.body.appendChild(jelly);

    // 点击爆炸
    jelly.onclick = function (e) {
      if (!gameRunning) return;
      if (jelly.classList.contains("pop")) return;

      score++;
      infoScore.innerText = score;
      playPopSound();

      // 爆炸位置
      const rect = jelly.getBoundingClientRect();
      const burstX = rect.left + rect.width / 2;
      const burstY = rect.top + rect.height / 2;

      createBurst(burstX, burstY);
      jelly.classList.add("pop");

      setTimeout(() => jelly.remove(), 450);
    };

    // 左右漂浮
    const speedX = (Math.random() - 0.5) * 0.8;
    const speedY = (Math.random() - 0.5) * 0.4;

    function swim() {
      if (!gameRunning || !jelly.parentElement) return;

      x += speedX;
      y += speedY;

      if (x < 40 || x > window.innerWidth - 140) {
        speedX *= -1;
      }
      if (y < 60 || y > window.innerHeight - 140) {
        speedY *= -1;
      }

      jelly.style.left = x + "px";
      jelly.style.top = y + "px";

      requestAnimationFrame(swim);
    }

    swim();

    // 自动消失
    setTimeout(() => {
      if (jelly.parentElement && !jelly.classList.contains("pop")) {
        jelly.style.opacity = "0";
        jelly.style.transition = "opacity 0.3s";
        setTimeout(() => jelly.remove(), 300);
      }
    }, 2600);
  }

  // 倒计时
  function startCountdown() {
    countDownTimer = setInterval(() => {
      timeLeft--;
      infoTimer.innerText = timeLeft;

      if (timeLeft <= 0) {
        endGame();
      }
    }, 1000);
  }

  // 开始生成水母
  function startSpawn() {
    spawnTimer = setInterval(() => {
      if (gameRunning) {
        spawnJelly();
      }
    }, 700);
  }

  // 游戏结束
  function endGame() {
    gameRunning = false;
    clearInterval(countDownTimer);
    clearInterval(spawnTimer);

    if (score > highScore) {
      highScore = score;
      localStorage.setItem("jellyHighScore", highScore);
    }

    finalScore.innerText = score;
    finalHs.innerText = highScore;
    gameOverPanel.style.display = "block";
  }

  // 重新开始
  function restartGame() {
    document.querySelectorAll(".jelly").forEach(el => el.remove());
    document.querySelectorAll(".burst").forEach(el => el.remove());
    document.querySelectorAll(".particle").forEach(el => el.remove());

    score = 0;
    timeLeft = 60;
    gameRunning = true;

    infoScore.innerText = score;
    infoTimer.innerText = timeLeft;
    hsDisplay.innerText = highScore;
    gameOverPanel.style.display = "none";

    startCountdown();
    startSpawn();
  }

  // 首次点击页面后解锁音效
  document.addEventListener("click", function initAudio() {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }
    document.removeEventListener("click", initAudio);
  }, { once: true });

  // 启动游戏
  startCountdown();
  startSpawn();
</script>

</body>
</html>
<game html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>水母小游戏</title>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{
  background:#082844;
  font-family:system-ui;
  color:#fff;
  text-align:center;
  padding-top:200px;
}
a{
  font-size:32px;
  color:#89e8ff;
}
a:hover{
  color:#ffffff;
}
</style>
</head>
<body>
  <a href="game.html">点击开始水母小游戏</a >
</body>
</html>
