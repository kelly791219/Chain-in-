<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>幸運號碼盲盒抽獎系統</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    @keyframes prizeBounce {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.15) rotate(3deg); }
    }
    .prize-anim { animation: prizeBounce 0.6s ease-in-out infinite; }
  </style>
</head>
<body class="bg-gradient-to-br from-indigo-900 via-purple-900 to-slate-900 min-h-screen text-slate-100 antialiased pb-10">

  <!-- 頂部資訊與控制列 -->
  <header class="max-w-5xl mx-auto px-4 py-4 flex flex-wrap items-center justify-between gap-3 border-b border-white/10">
    <div class="flex items-center gap-3">
      <div class="w-11 h-11 rounded-2xl bg-gradient-to-tr from-amber-400 to-orange-500 flex items-center justify-center text-slate-900 text-2xl shadow-lg shadow-orange-500/30 font-black">
        <i class="fa-solid fa-gift"></i>
      </div>
      <div>
        <h1 class="text-xl font-black tracking-wide text-white flex items-center gap-2">
          現場幸運選號抽獎
          <span id="poolStatusBadge" class="text-xs bg-indigo-500/30 text-indigo-300 border border-indigo-400/40 px-2 py-0.5 rounded-full font-bold">1~50 號碼池</span>
        </h1>
        <p class="text-xs text-purple-200">選取你的幸運號碼・隨機解鎖專屬驚喜大獎</p>
      </div>
    </div>

    <!-- 庫存數據小統計與後台按鈕 -->
    <div class="flex items-center gap-2">
      <div class="bg-white/10 backdrop-blur-md px-3.5 py-1.5 rounded-xl border border-white/10 text-xs">
        剩餘禮物：<b id="totalStockDisplay" class="text-amber-400 text-sm font-black">1000</b> 件
      </div>
      <button onclick="toggleAdminPanel()" class="bg-white/15 hover:bg-white/25 text-white px-3 py-1.5 rounded-xl text-xs font-bold transition flex items-center gap-1.5 border border-white/10">
        <i class="fa-solid fa-sliders"></i> 後台庫存管理
      </button>
    </div>
  </header>

  <main class="max-w-5xl mx-auto px-4 mt-6">

    <!-- 抽獎主畫面：民眾選號盤 -->
    <div class="bg-white/10 backdrop-blur-xl border border-white/15 rounded-3xl p-5 md:p-8 shadow-2xl">
      <div class="text-center mb-6">
        <h2 class="text-2xl font-black text-amber-300 tracking-wider">請挑選一個幸運號碼</h2>
        <p class="text-xs text-slate-300 mt-1">每次點擊號碼，皆從剩餘獎項庫中即時隨機開出驚喜禮物！</p>
      </div>

      <!-- 號碼按鈕網格 (動態產生 1~50 或 1~25) -->
      <div id="numberGrid" class="grid grid-cols-5 sm:grid-cols-7 md:grid-cols-10 gap-2.5">
        <!-- 由 JS 動態繪製 -->
      </div>
    </div>

    <!-- 現場中獎跑馬燈 / 最近中獎紀錄 -->
    <div class="mt-6 bg-white/5 border border-white/10 rounded-2xl p-4">
      <div class="flex justify-between items-center mb-2">
        <h3 class="text-xs font-bold text-slate-300 flex items-center gap-1.5">
          <i class="fa-solid fa-clock-rotate-left text-amber-400"></i> 最近抽獎歷程 (共 <span id="historyCount">0</span> 筆)
        </h3>
        <button onclick="exportHistoryCSV()" class="text-xs text-indigo-300 hover:text-indigo-200 underline">匯出完整中獎名單 (CSV)</button>
      </div>
      <div id="recentHistory" class="flex flex-wrap gap-2 text-xs">
        <span class="text-slate-400">尚無抽獎紀錄</span>
      </div>
    </div>

  </main>

  <!-- 開獎動畫 Modal -->
  <div id="revealModal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md z-50 hidden flex items-center justify-center p-4">
    <div class="bg-gradient-to-b from-slate-900 to-indigo-950 border border-white/20 rounded-3xl max-w-sm w-full p-6 text-center shadow-2xl relative">
      <div id="boxAnimating" class="py-6">
        <div class="text-6xl text-amber-400 prize-anim mb-3">
          <i class="fa-solid fa-gift"></i>
        </div>
        <div class="text-sm font-bold text-amber-200">選中幸運號碼 <span id="pickedNumText" class="text-xl text-white underline font-black">--</span></div>
        <div class="text-xs text-slate-400 mt-1 animate-pulse">正在開啟盲盒禮物...</div>
      </div>

      <div id="boxResult" class="hidden py-4">
        <div class="text-5xl text-amber-400 mb-2">
          <i class="fa-solid fa-award"></i>
        </div>
        <div class="text-xs text-slate-400 font-bold tracking-widest uppercase">CONGRATULATIONS</div>
        <div id="rewardBadge" class="inline-block bg-amber-400 text-slate-950 font-black text-xs px-3 py-1 rounded-full mt-2 mb-1">A 獎</div>
        <div id="rewardName" class="text-2xl font-black text-white mt-1">特獎禮品名稱</div>
        <p class="text-[11px] text-slate-300 mt-2">請向現場工作人員領取您的專屬獎品！</p>
        
        <button onclick="closeRevealModal()" class="mt-6 w-full bg-indigo-600 hover:bg-indigo-500 text-white font-bold py-3 rounded-2xl shadow-lg shadow-indigo-600/40 transition">
          確認領獎 / 下一位
        </button>
      </div>
    </div>
  </div>

  <!-- 後台庫存管理 Modal -->
  <div id="adminModal" class="fixed inset-0 bg-slate-950/70 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
    <div class="bg-slate-900 border border-white/20 rounded-3xl max-w-2xl w-full p-6 text-slate-100 shadow-2xl max-h-[90vh] flex flex-col">
      <div class="flex justify-between items-center border-b border-white/10 pb-3">
        <h3 class="font-bold text-base flex items-center gap-2 text-white">
          <i class="fa-solid fa-boxes-stacked text-amber-400"></i> A~I 獎項庫存與門檻配置
        </h3>
        <button onclick="toggleAdminPanel()" class="text-slate-400 hover:text-white"><i class="fa-solid fa-xmark text-lg"></i></button>
      </div>

      <div class="py-3 text-xs text-slate-300">
        💡 規則說明：平時提供 <b>1~50</b> 號碼選號；當整體剩餘庫存降至 <b>100 件以下</b> 時，號碼池自動切換為 <b>1~25</b> 號碼。
      </div>

      <div class="flex-1 overflow-y-auto space-y-2 pr-1" id="prizeStockEditor">
        <!-- 動態列出 A~I 獎編輯欄位 -->
      </div>

      <div class="border-t border-white/10 pt-4 mt-3 flex justify-between items-center">
        <button onclick="resetAllDefaults()" class="text-xs text-rose-400 hover:text-rose-300 underline font-semibold">
          重置為預設 1000 份庫存
        </button>
        <div class="flex gap-2">
          <button onclick="toggleAdminPanel()" class="px-4 py-2 text-xs bg-slate-700 hover:bg-slate-600 rounded-xl">關閉</button>
          <button onclick="saveAdminChanges()" class="px-5 py-2 text-xs bg-indigo-600 hover:bg-indigo-500 font-bold rounded-xl shadow-md">儲存設定</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    // 預設 A~I (共 10 個獎項) 初始 1000 份分配
    const DEFAULT_PRIZES = [
      { code: 'A', name: '特獎・極致大禮盒', initial: 10, remaining: 10 },
      { code: 'B', name: '二獎・超商禮券500元', initial: 20, remaining: 20 },
      { code: 'C', name: '三獎・貓空雙人套餐', initial: 40, remaining: 40 },
      { code: 'D', name: '四獎・精選飲品兌換券', initial: 80, remaining: 80 },
      { code: 'E', name: '五獎・超萌捏捏樂盲盒', initial: 120, remaining: 120 },
      { code: 'F', name: '六獎・特色文創明信片', initial: 150, remaining: 150 },
      { code: 'G', name: '七獎・幸運抽抽樂禮包', initial: 180, remaining: 180 },
      { code: 'H', name: '八獎・貓空精選茶包', initial: 200, remaining: 200 },
      { code: 'I', name: '普獎・造型紀念徽章', initial: 150, remaining: 150 },
      { code: '特別獎', name: '彩蛋・神祕驚喜小禮', initial: 50, remaining: 50 }
    ];

    let prizes = [];
    let history = [];
    let isDrawing = false;

    // 1. 初始化資料庫 (localStorage)
    function initData() {
      const savedPrizes = localStorage.getItem('blind_lottery_prizes');
      const savedHistory = localStorage.getItem('blind_lottery_history');

      if (savedPrizes) {
        try { prizes = JSON.parse(savedPrizes); } catch(e) { prizes = DEFAULT_PRIZES; }
      } else {
        prizes = JSON.parse(JSON.stringify(DEFAULT_PRIZES));
      }

      if (savedHistory) {
        try { history = JSON.parse(savedHistory); } catch(e) { history = []; }
      } else {
        history = [];
      }

      updateUI();
    }

    function saveData() {
      localStorage.setItem('blind_lottery_prizes', JSON.stringify(prizes));
      localStorage.setItem('blind_lottery_history', JSON.stringify(history));
    }

    // 計算總剩餘庫存
    function getTotalRemaining() {
      return prizes.reduce((sum, p) => sum + (p.remaining || 0), 0);
    }

    // 更新介面狀態 (判斷是要 50 個號碼還是 25 個號碼)
    function updateUI() {
      const totalRemaining = getTotalRemaining();
      document.getElementById('totalStockDisplay').innerText = totalRemaining;

      // 判斷號碼池上限：大於 100 份時為 50 個號碼；100 份以下時自動改為 25 個號碼
      let maxNumbers = 50;
      const badge = document.getElementById('poolStatusBadge');
      if (totalRemaining <= 100) {
        maxNumbers = 25;
        badge.innerText = '🔥 最後倒數！1~25 號碼池';
        badge.className = 'text-xs bg-rose-500/30 text-rose-300 border border-rose-400/40 px-2 py-0.5 rounded-full font-bold animate-pulse';
      } else {
        maxNumbers = 50;
        badge.innerText = '1~50 號碼池';
        badge.className = 'text-xs bg-indigo-500/30 text-indigo-300 border border-indigo-400/40 px-2 py-0.5 rounded-full font-bold';
      }

      renderNumberGrid(maxNumbers);
      renderRecentHistory();
    }

    // 渲染號碼按鈕
    function renderNumberGrid(count) {
      const grid = document.getElementById('numberGrid');
      grid.innerHTML = '';

      for (let i = 1; i <= count; i++) {
        const btn = document.createElement('button');
        btn.className = `h-14 sm:h-16 rounded-2xl font-black text-lg sm:text-xl flex flex-col items-center justify-center transition-all duration-200 transform active:scale-95 shadow-lg border ${
          count === 25 
            ? 'bg-gradient-to-b from-rose-500/40 to-rose-700/60 border-rose-400/30 text-rose-100 hover:from-rose-500 hover:to-rose-600'
            : 'bg-gradient-to-b from-white/10 to-white/5 border-white/20 text-white hover:from-indigo-600/60 hover:to-purple-600/80 hover:border-indigo-400'
        }`;
        btn.innerHTML = `<span>${i}</span><span class="text-[9px] opacity-60 font-normal">LUCKY</span>`;
        btn.onclick = () => handleDraw(i);
        grid.appendChild(btn);
      }
    }

    // 核心：點選號碼後，純隨機抽取目前剩餘有庫存的獎品
    function handleDraw(num) {
      if (isDrawing) return;
      const totalRemaining = getTotalRemaining();
      if (totalRemaining <= 0) {
        alert('全部 1000 份禮物已經全數抽出完畢！活動圓滿結束！');
        return;
      }

      isDrawing = true;

      // 建立抽獎池 (依照目前各獎項剩餘庫存權重隨機抽取)
      const pool = [];
      prizes.forEach(p => {
        for (let i = 0; i < p.remaining; i++) {
          pool.push(p);
        }
      });

      // 純隨機抽選一個獎項（號碼與獎品完全不掛鉤）
      const randomIndex = Math.floor(Math.random() * pool.length);
      const wonPrize = pool[randomIndex];

      // 扣除該獎項庫存 1 件
      wonPrize.remaining -= 1;

      // 寫入紀錄
      const record = {
        number: num,
        code: wonPrize.code,
        name: wonPrize.name,
        time: new Date().toLocaleTimeString()
      };
      history.unshift(record);
      saveData();

      // 開啟開獎動畫 Modal
      openRevealAnimation(num, wonPrize);
    }

    // 開獎動畫
    function openRevealAnimation(num, prize) {
      document.getElementById('pickedNumText').innerText = num;
      document.getElementById('boxAnimating').classList.remove('hidden');
      document.getElementById('boxResult').classList.add('hidden');
      document.getElementById('revealModal').classList.remove('hidden');

      setTimeout(() => {
        document.getElementById('boxAnimating').classList.add('hidden');
        document.getElementById('boxResult').classList.remove('hidden');
        document.getElementById('rewardBadge').innerText = `${prize.code} 獎`;
        document.getElementById('rewardName').innerText = prize.name;
        updateUI();
      }, 1000);
    }

    function closeRevealModal() {
      document.getElementById('revealModal').classList.add('hidden');
      isDrawing = false;
    }

    // 渲染最近抽獎紀錄
    function renderRecentHistory() {
      const container = document.getElementById('recentHistory');
      document.getElementById('historyCount').innerText = history.length;
      if (history.length === 0) {
        container.innerHTML = '<span class="text-slate-400">尚無抽獎紀錄</span>';
        return;
      }
      container.innerHTML = '';
      history.slice(0, 10).forEach(h => {
        const item = document.createElement('div');
        item.className = 'bg-white/10 border border-white/10 px-2.5 py-1 rounded-lg flex items-center gap-1.5';
        item.innerHTML = `
          <span class="text-amber-400 font-bold">#${h.number}號</span>
          <span class="text-slate-300">➔</span>
          <span class="bg-indigo-600/80 text-[10px] px-1.5 py-0.2 rounded font-bold">${h.code}</span>
          <span class="text-slate-200">${escapeHtml(h.name)}</span>
        `;
        container.appendChild(item);
      });
    }

    // 匯出 CSV 報表
    function exportHistoryCSV() {
      if (history.length === 0) return alert('目前尚無抽獎紀錄！');
      let csv = "\uFEFF點選號碼,得獎級別,獎品名稱,抽取時間\n";
      history.forEach(h => {
        csv += `"${h.number}","${h.code}","${h.name.replace(/"/g, '""')}","${h.time}"\n`;
      });
      const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = `盲盒抽獎完整名冊_${new Date().toISOString().slice(0,10)}.csv`;
      link.click();
    }

    // 後台庫存編輯器
    function toggleAdminPanel() {
      const modal = document.getElementById('adminModal');
      if (modal.classList.contains('hidden')) {
        renderStockEditor();
        modal.classList.remove('hidden');
      } else {
        modal.classList.add('hidden');
      }
    }

    function renderStockEditor() {
      const editor = document.getElementById('prizeStockEditor');
      editor.innerHTML = '';
      prizes.forEach((p, idx) => {
        const row = document.createElement('div');
        row.className = 'grid grid-cols-12 gap-2 items-center bg-white/5 p-2 rounded-xl border border-white/10';
        row.innerHTML = `
          <div class="col-span-2">
            <input type="text" id="p_code_${idx}" value="${escapeHtml(p.code)}" class="w-full bg-slate-800 border border-white/20 rounded-lg p-1.5 text-center font-bold text-xs">
          </div>
          <div class="col-span-6">
            <input type="text" id="p_name_${idx}" value="${escapeHtml(p.name)}" class="w-full bg-slate-800 border border-white/20 rounded-lg p-1.5 text-xs">
          </div>
          <div class="col-span-4 flex items-center gap-1">
            <span class="text-[10px] text-slate-400">剩餘:</span>
            <input type="number" min="0" id="p_rem_${idx}" value="${p.remaining}" class="w-full bg-slate-800 border border-white/20 rounded-lg p-1.5 text-center font-bold text-xs text-amber-400">
          </div>
        `;
        editor.appendChild(row);
      });
    }

    function saveAdminChanges() {
      prizes.forEach((p, idx) => {
        p.code = document.getElementById(`p_code_${idx}`).value.trim();
        p.name = document.getElementById(`p_name_${idx}`).value.trim();
        p.remaining = parseInt(document.getElementById(`p_rem_${idx}`).value, 10) || 0;
      });
      saveData();
      toggleAdminPanel();
      updateUI();
      alert('庫存設定已成功更新！');
    }

    function resetAllDefaults() {
      if (!confirm('確定要重置所有獎項庫存至預設的 1000 份嗎？歷史紀錄將會保留。')) return;
      prizes = JSON.parse(JSON.stringify(DEFAULT_PRIZES));
      saveData();
      renderStockEditor();
      updateUI();
    }

    function escapeHtml(text) {
      return text.replace(/[&<>"']/g, function(m) {
        return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' }[m];
      });
    }

    window.onload = initData;
  </script>
</body>
</html>
