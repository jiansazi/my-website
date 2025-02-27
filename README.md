# my-website
教师点名系统
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>随机点名系统</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      background-color: #0a0a20;
      overflow: hidden;
      flex-direction: column;
    }

    /* 星空背景 */
    .stars {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: -1;
    }

    .star {
      position: absolute;
      background: #fff;
      border-radius: 50%;
      animation: twinkle var(--duration) infinite ease-in-out;
    }

    @keyframes twinkle {
      0%, 100% { opacity: 0.2; }
      50% { opacity: 1; }
    }

    .blackboard {
      width: 300px;
      height: 300px;
      border-radius: 50%; /* 圆形呼吸球 */
      display: flex;
      justify-content: center;
      align-items: center;
      box-shadow: 0 0 25px rgba(255, 255, 255, 0.5);
      animation: breathe 8s infinite, colorChange 15s infinite; /* 呼吸动画和颜色变换 */
      cursor: pointer; /* 点击光标 */
      position: relative;
      transition: all 0.3s ease;
    }

    .blackboard:hover {
      transform: scale(1.05);
      box-shadow: 0 0 35px rgba(255, 255, 255, 0.8);
    }

    .blackboard:active {
      transform: scale(0.95);
    }

    @keyframes breathe {
      0% { transform: scale(1); }
      50% { transform: scale(1.1); }
      100% { transform: scale(1); }
    }

    @keyframes colorChange {
      0% { background-color: #ff5252; }
      20% { background-color: #ffb142; }
      40% { background-color: #33d9b2; }
      60% { background-color: #34ace0; }
      80% { background-color: #b33771; }
      100% { background-color: #ff5252; }
    }

    .circle {
      width: 220px;
      height: 220px;
      border-radius: 50%;
      background-color: rgba(0, 0, 0, 0.2);
      backdrop-filter: blur(5px);
      display: flex;
      justify-content: center;
      align-items: center;
      border: 2px solid rgba(255, 255, 255, 0.3);
      box-shadow: inset 0 0 20px rgba(255, 255, 255, 0.2);
    }

    .name {
      font-size: 28px;
      font-weight: bold;
      color: #fff;
      text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
      text-align: center;
      padding: 10px;
    }

    .msg {
      margin-top: 30px;
      font-size: 22px;
      color: #fff;
      background-color: rgba(0, 0, 0, 0.6);
      padding: 12px 20px;
      border-radius: 10px;
      box-shadow: 0 0 15px rgba(255, 255, 255, 0.1);
      transition: all 0.5s ease;
      opacity: 0;
      transform: translateY(20px);
    }

    .msg.show {
      opacity: 1;
      transform: translateY(0);
    }

    /* 选中特效 */
    @keyframes glow {
      0% {
        text-shadow: 0 0 10px #fff, 0 0 20px #fff, 0 0 30px #fff;
        transform: scale(1) rotate(0deg);
        color: #fff;
      }
      50% {
        text-shadow: 0 0 20px #fff, 0 0 30px #ff0, 0 0 40px #ff0;
        transform: scale(1.2) rotate(5deg);
        color: #ff0;
      }
      100% {
        text-shadow: 0 0 10px #fff, 0 0 20px #fff, 0 0 30px #fff;
        transform: scale(1) rotate(0deg);
        color: #fff;
      }
    }

    .selected {
      animation: glow 1.5s infinite alternate;
    }

    /* 庆祝特效 */
    .confetti {
      position: absolute;
      width: 10px;
      height: 10px;
      opacity: 0;
    }

    @keyframes fall {
      0% {
        transform: translateY(-100px) rotate(0deg);
        opacity: 1;
      }
      100% {
        transform: translateY(calc(100vh - 100px)) rotate(360deg);
        opacity: 0;
      }
    }

    /* 计数器 */
    .counter {
      position: absolute;
      top: 20px;
      right: 20px;
      background-color: rgba(0, 0, 0, 0.6);
      color: white;
      padding: 10px;
      border-radius: 5px;
      font-size: 14px;
    }
  </style>
</head>
<body>

  <div class="stars" id="stars"></div>

  <div class="blackboard" id="randomSelector">
    <div class="circle">
      <span class="name" id="currentStudent">点击开始点名</span>
    </div>
  </div>

  <div id="message" class="msg"></div>
  <div class="counter" id="counter"></div>

  <script>
    const students = [
      '孟皓燃', '万仕成', '邵玉麟', '范袁鹏', '沈心燕',
      '干予琪', '孙语芯', '何思雨', '王雨欣', '俞芷玥',
      '张思悦', '郎建豪', '李妍希', '夏雨琳', '贺学铱',
      '梁蕊妮', '张子睿', '曾嘉羿', '朱瑾芊', '向宇阳'
    ];

    let isRolling = false;
    let interval;
    let selectedHistory = [];
    
    // 跟踪已选择的学生
    const selectionCounts = {};
    students.forEach(student => {
      selectionCounts[student] = 0;
    });

    const randomSelector = document.getElementById('randomSelector');
    const currentStudentElement = document.getElementById('currentStudent');
    const message = document.getElementById('message');
    const counter = document.getElementById('counter');

    // 创建星空背景
    function createStars() {
      const starsContainer = document.getElementById('stars');
      const starCount = 200;
      
      for (let i = 0; i < starCount; i++) {
        const star = document.createElement('div');
        star.classList.add('star');
        
        // 随机位置
        const x = Math.random() * 100;
        const y = Math.random() * 100;
        
        // 随机大小
        const size = Math.random() * 3;
        
        // 随机闪烁速度
        const duration = 3 + Math.random() * 5;
        
        star.style.left = `${x}%`;
        star.style.top = `${y}%`;
        star.style.width = `${size}px`;
        star.style.height = `${size}px`;
        star.style.setProperty('--duration', `${duration}s`);
        
        starsContainer.appendChild(star);
      }
    }

    // 创建五彩纸屑
    function createConfetti() {
      const colors = ['#ff5252', '#ffb142', '#33d9b2', '#34ace0', '#b33771', '#fffa65'];
      const confettiCount = 100;
      
      for (let i = 0; i < confettiCount; i++) {
        const confetti = document.createElement('div');
        confetti.classList.add('confetti');
        
        // 随机颜色
        const color = colors[Math.floor(Math.random() * colors.length)];
        
        // 随机位置
        const x = Math.random() * 100;
        
        // 随机形状
        const isRectangle = Math.random() > 0.5;
        if (isRectangle) {
          confetti.style.width = `${5 + Math.random() * 10}px`;
          confetti.style.height = `${5 + Math.random() * 5}px`;
        } else {
          const size = 5 + Math.random() * 10;
          confetti.style.width = `${size}px`;
          confetti.style.height = `${size}px`;
          confetti.style.borderRadius = '50%';
        }
        
        confetti.style.backgroundColor = color;
        confetti.style.left = `${x}%`;
        confetti.style.position = 'absolute';
        confetti.style.top = '0';
        
        // 随机动画时间
        const duration = 1 + Math.random() * 3;
        confetti.style.animation = `fall ${duration}s linear forwards`;
        confetti.style.animationDelay = `${Math.random() * 2}s`;
        
        document.body.appendChild(confetti);
        
        // 动画结束后移除元素
        setTimeout(() => {
          confetti.remove();
        }, duration * 1000 + 2000);
      }
    }

    function toggleRoll() {
      if (isRolling) {
        clearInterval(interval);
        isRolling = false;
        
        // 更新选择计数
        const selectedStudent = currentStudentElement.textContent;
        selectionCounts[selectedStudent]++;
        updateCounter();
        
        // 记录历史
        selectedHistory.push(selectedStudent);
        if (selectedHistory.length > 10) {
          selectedHistory.shift();
        }
        
        // 显示消息
        message.textContent = `恭喜 ${selectedStudent} 被选中!`;
        message.classList.add('show');
        
        // 添加选中特效
        currentStudentElement.classList.add('selected');
        
        // 触发庆祝特效
        createConfetti();
        
        // 缓慢恢复正常动画
        setTimeout(() => {
          currentStudentElement.classList.remove('selected');
          message.classList.remove('show');
        }, 5000);
      } else {
        isRolling = true;
        message.textContent = '正在随机选择幸运学生...';
        message.classList.add('show');
        currentStudentElement.classList.remove('selected');
        
        // 改变滚动速度:从快到慢
        let speed = 50;
        const maxSpeed = 300;
        const increment = 1;
        
        interval = setInterval(() => {
          // 根据已被选次数减小被选概率
          let weightedStudents = [...students];
          const maxCount = Math.max(...Object.values(selectionCounts));
          
          // 确保所有学生至少有一定概率被选中
          for (const student of students) {
            // 增加未被选次数少的学生的权重
            const weight = maxCount - selectionCounts[student] + 1;
            // 添加额外的实例来提高权重
            for (let i = 0; i < weight; i++) {
              weightedStudents.push(student);
            }
          }
          
          const randomIndex = Math.floor(Math.random() * weightedStudents.length);
          currentStudentElement.textContent = weightedStudents[randomIndex];
          
          // 增加速度(减慢)
          speed += increment;
          if (speed >= maxSpeed) {
            toggleRoll(); // 自动停止
          } else if (speed > maxSpeed / 2) {
            clearInterval(interval);
            interval = setInterval(arguments.callee, speed);
          }
        }, speed);
      }
    }

    // 更新计数器
    function updateCounter() {
      let countText = "被选次数:<br>";
      for (const student of students) {
        if (selectionCounts[student] > 0) {
          countText += `${student}: ${selectionCounts[student]}次<br>`;
        }
      }
      counter.innerHTML = countText;
    }

    randomSelector.addEventListener('click', () => {
      // 添加点击效果
      randomSelector.style.transform = 'scale(0.95)';
      setTimeout(() => {
        randomSelector.style.transform = '';
      }, 150);
      
      toggleRoll();
    });

    // 初始化星空背景
    createStars();
  </script>

</body>
</html>
