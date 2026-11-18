# score_math-2627
<!DOCTYPE html>
<html lang="zh-HK">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
  <!-- 支援 iPad 全螢幕 Web App 模式 -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="6人計分板">
  <title>6人小組計分板</title>
  <style>
    * { 
      box-sizing: border-box; 
      user-select: none; 
      -webkit-user-select: none; 
    }
    body {
      margin: 0;
      padding: 10px 16px;
      height: 100vh;
      display: flex;
      flex-direction: column;
      background: #F3F4F6;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang HK", sans-serif;
      overflow: hidden;
    }
    .header {
      text-align: center;
      font-size: 1.6rem;
      font-weight: 900;
      color: #374151;
      height: 35px;
      line-height: 35px;
      margin-bottom: 5px;
    }
    /* 3 欄 × 2 行 網格結構，配合 calc 動態高度，確保任何 iPad 螢幕皆不會溢出 */
    .grid-container {
      flex: 1;
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      grid-template-rows: repeat(2, 1fr);
      gap: 12px;
      height: calc(100vh - 100px); 
    }
    .student-card {
      background: #FFFFFF;
      border-radius: 20px;
      border: 3px solid #E5E7EB;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-between;
      padding: 12px 14px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.05);
      overflow: hidden;
    }
    
    /* 6 組獨立高辨識主題色 */
    .theme-1 { border-color: #60A5FA; background: #EFF6FF; } /* 藍 */
    .theme-2 { border-color: #34D399; background: #ECFDF5; } /* 綠 */
    .theme-3 { border-color: #FBBF24; background: #FFFBEB; } /* 橙 */
    .theme-4 { border-color: #C084FC; background: #FAF5FF; } /* 紫 */
    .theme-5 { border-color: #F87171; background: #FEF2F2; } /* 粉紅 */
    .theme-6 { border-color: #2DD4BF; background: #F0FDFA; } /* 青綠 */

    .card-top {
      display: flex;
      align-items: center;
      justify-content: space-between;
      width: 100%;
      padding: 0 4px;
    }
    .avatar-name-group {
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .avatar { font-size: 2.2rem; }
    .student-name {
      font-size: 1.4rem;
      font-weight: 800;
      color: #1F2937;
      background: rgba(255, 255, 255, 0.85);
      padding: 4px 12px;
      border-radius: 12px;
      cursor: pointer;
      border: 1px dashed #9CA3AF;
    }
    .score-number {
      font-size: 1.8rem;
      font-weight: 900;
      color: #374151;
    }

    /* 3 顆具象化星星進度槽 */
    .stars-track {
      display: flex;
      gap: 12px; 
      background: rgba(255, 255, 255, 0.95);
      padding: 6px 24px;
      border-radius: 14px;
      margin: 8px 0;
    }
    .star-slot {
      font-size: 1.8rem;
      color: #D1D5DB;
      transition: all 0.2s ease;
    }
    .star-slot.filled {
      color: #F59E0B;
      transform: scale(1.25);
    }

    /* 拍掣按鈕 */
    .btn-add {
      width: 100%;
      height: 52px;
      border-radius: 16px;
      border: none;
      color: #FFFFFF;
      font-size: 1.5rem;
      font-weight: 900;
      cursor: pointer;
      box-shadow: 0 6px 12px rgba(0,0,0,0.12);
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      outline: none;
    }
    .btn-add:active { transform: scale(0.96); }
    .theme-1 .btn-add { background: #3B82F6; }
    .theme-2 .btn-add { background: #10B981; }
    .theme-3 .btn-add { background: #F59E0B; }
    .theme-4 .btn-add { background: #8B5CF6; }
    .theme-5 .btn-add { background: #EF4444; }
    .theme-6 .btn-add { background: #14B8A6; }

    /* 底部功能欄 */
    .footer-bar {
      height: 45px;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-top: 10px;
    }
    .btn-reset-all {
      background: #E5E7EB;
      color: #4B5563;
      border: none;
      border-radius: 8px;
      padding: 8px 20px;
      font-size: 0.95rem;
      font-weight: 600;
      cursor: pointer;
    }

    /* 滿 3 分全螢幕大獎勵視窗 */
    .grand-modal {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background: rgba(0, 0, 0, 0.75);
      z-index: 100;
      align-items: center;
      justify-content: center;
    }
    .grand-modal.active { display: flex; }
    .modal-card {
      background: #FFFFFF;
      border: 8px solid #F59E0B;
      border-radius: 35px;
      padding: 35px 50px;
      text-align: center;
      animation: popIn 0.35s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }
    @keyframes popIn {
      from { transform: scale(0.6); opacity: 0; }
      to { transform: scale(1); opacity: 1; }
    }
    .modal-winner { font-size: 3rem; font-weight: 900; color: #D97706; margin-bottom: 8px; }
    .modal-icon { font-size: 6.5rem; margin: 8px 0; }
    .btn-confirm {
      background: #10B981;
      color: #FFFFFF;
      border: none;
      border-radius: 18px;
      padding: 14px 40px;
      font-size: 1.8rem;
      font-weight: bold;
      cursor: pointer;
      margin-top: 15px;
    }
  </style>
</head>
<body>

  <div class="header">🌟 6人小組課堂計分板 🌟</div>

  <div class="grid-container" id="gridContainer">
    <!-- 學生卡片動態渲染 -->
  </div>

  <div class="footer-bar">
    <button class="btn-reset-all" id="resetAllBtn">重新開始整組計分</button>
  </div>

  <!-- 滿 3 分專屬表揚彈窗 -->
  <div class="grand-modal" id="grandModal">
    <div class="modal-card">
      <div class="modal-winner" id="winnerName">學生姓名</div>
      <div class="modal-icon">🏆</div>
      <div style="font-size: 2rem; font-weight: bold; color: #374151;">太棒了！累積滿 3 粒星！</div>
      <button class="btn-confirm" id="modalCloseBtn">繼續努力 👏</button>
    </div>
  </div>

  <script>
    // 預設 6 位同學清單
    const students = [
      { id: 1, name: "小熊", icon: "🐻", theme: "theme-1", score: 0 },
      { id: 2, name: "小兔", icon: "🐰", theme: "theme-2", score: 0 },
      { id: 3, name: "小貓", icon: "🐱", theme: "theme-3", score: 0 },
      { id: 4, name: "小狗", icon: "🐶", theme: "theme-4", score: 0 },
      { id: 5, name: "小獅", icon: "🦁", theme: "theme-5", score: 0 },
      { id: 6, name: "熊貓", icon: "🐼", theme: "theme-6", score: 0 }
    ];

    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

    // 加 1 分短音階
    function playPing() {
      if (audioCtx.state === 'suspended') audioCtx.resume();
      const notes = [523.25, 659.25];
      notes.forEach((f, i) => {
        const osc = audioCtx.createOscillator();
        const g = audioCtx.createGain();
        osc.frequency.value = f;
        g.gain.setValueAtTime(0.12, audioCtx.currentTime + i * 0.08);
        g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + i * 0.08 + 0.25);
        osc.connect(g);
        g.connect(audioCtx.destination);
        osc.start(audioCtx.currentTime + i * 0.08);
        osc.stop(audioCtx.currentTime + i * 0.08 + 0.25);
      });
    }

    // 滿 3 分勝利號角長音
    function playFanfare() {
      if (audioCtx.state === 'suspended') audioCtx.resume();
      const fanfare = [
        { f: 523.25, t: 0, d: 0.15 },
        { f: 659.25, t: 0.15, d: 0.15 },
        { f: 783.99, t: 0.3, d: 0.15 },
        { f: 1046.50, t: 0.45, d: 0.6 }
      ];
      fanfare.forEach(n => {
        const osc = audioCtx.createOscillator();
        const g = audioCtx.createGain();
        osc.frequency.value = n.f;
        g.gain.setValueAtTime(0.2, audioCtx.currentTime + n.t);
        g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + n.t + n.d);
        osc.connect(g);
        g.connect(audioCtx.destination);
        osc.start(audioCtx.currentTime + n.t);
        osc.stop(audioCtx.currentTime + n.t + n.d);
      });
    }

    const container = document.getElementById('gridContainer');
    const grandModal = document.getElementById('grandModal');
    const winnerName = document.getElementById('winnerName');
    const modalCloseBtn = document.getElementById('modalCloseBtn');
    const resetAllBtn = document.getElementById('resetAllBtn');

    // 動態渲染卡片
    function renderCards() {
      container.innerHTML = '';
      students.forEach((stu, index) => {
        const card = document.createElement('div');
        card.className = `student-card ${stu.theme}`;

        // 星星進度槽：改為 3 分循環（累積滿 3 顆星即觸發慶祝，保留總分）
        const filledStars = stu.score % 3 === 0 && stu.score > 0 ? 3 : stu.score % 3;
        let starsHtml = '';
        for (let i = 1; i <= 3; i++) {
          starsHtml += `<span class="star-slot ${i <= filledStars ? 'filled' : ''}">★</span>`;
        }

        card.innerHTML = `
          <div class="card-top">
            <div class="avatar-name-group">
              <span class="avatar">${stu.icon}</span>
              <span class="student-name" onclick="renameStudent(${index})" title="點擊修改名字">${stu.name}</span>
            </div>
            <span class="score-number">${stu.score}分</span>
          </div>
          <div class="stars-track">${starsHtml}</div>
          <button class="btn-add" onclick="addScore(${index})">
            <span>👍 答啱！</span>
          </button>
        `;
        container.appendChild(card);
      });
    }

    // 點擊加分
    window.addScore = function(index) {
      students[index].score += 1;
      renderCards();

      // 每累積 3 分（3, 6, 9...）即刻觸發大獎勵
      if (students[index].score % 3 === 0) {
        playFanfare();
        winnerName.innerText = `🎉 ${students[index].name} 🎉`;
        grandModal.classList.add('active');
      } else {
        playPing();
      }
    };

    window.renameStudent = function(index) {
      const newName = prompt("請輸入新學生名字：", students[index].name);
      if (newName && newName.trim() !== "") {
        students[index].name = newName.trim();
        renderCards();
      }
    };

    modalCloseBtn.addEventListener('click', () => {
      grandModal.classList.remove('active');
    });

    resetAllBtn.addEventListener('click', () => {
      if (confirm('確定要將全班 6 位同學的分數全部歸零嗎？')) {
        students.forEach(s => s.score = 0);
        renderCards();
      }
    });

    renderCards();
  </script>
</body>
</html>
