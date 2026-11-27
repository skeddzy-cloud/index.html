<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta name="theme-color" content="#0f172a" />
  <title>双币种本地记账 (MYR / USD)</title>
  <style>
    :root {
      --bg: #f1f5f9;
      --card: #ffffff;
      --ink: #0f172a;
      --muted: #64748b;
      --primary: #0f766e;
      --primary-ink: #ffffff;
      --accent: #1d4ed8;
      --danger: #dc2626;
      --border: #e2e8f0;
    }
    * { box-sizing: border-box; touch-action: manipulation; }
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background: var(--bg);
      color: var(--ink);
      padding-bottom: 84px;
    }
    .wrap { max-width: 540px; margin: 0 auto; padding: 14px; }
    .summary {
      background: #0f172a;
      color: #fff;
      border-radius: 16px;
      padding: 16px;
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
      margin-bottom: 14px;
    }
    .sum-box { background: rgba(255,255,255,0.08); border-radius: 12px; padding: 12px; }
    .sum-label { font-size: 12px; color: #cbd5e1; }
    .sum-val { font-size: 20px; font-weight: 700; margin-top: 4px; }
    .card {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 16px;
      padding: 14px;
      margin-bottom: 14px;
    }
    .row { display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 10px; }
    .type-btn {
      flex: 1;
      min-height: 44px;
      border: 1px solid var(--border);
      background: #f8fafc;
      color: var(--ink);
      border-radius: 10px;
      font-weight: 600;
      font-size: 14px;
      cursor: pointer;
    }
    .type-btn.active { background: var(--primary); color: var(--primary-ink); border-color: var(--primary); }
    .amount-bar {
      display: flex;
      align-items: center;
      gap: 8px;
      background: #0f172a;
      color: #fff;
      border-radius: 14px;
      padding: 8px;
      margin-bottom: 10px;
    }
    .curr-btn {
      min-height: 48px;
      padding: 0 12px;
      border-radius: 10px;
      border: none;
      background: #334155;
      color: #fff;
      font-weight: 700;
      cursor: pointer;
    }
    .amt-input {
      flex: 1;
      min-width: 0;
      min-height: 48px;
      border: none;
      background: transparent;
      color: #fff;
      font-size: 24px;
      font-weight: 700;
      outline: none;
    }
    .save-btn {
      min-height: 48px;
      padding: 0 18px;
      border-radius: 10px;
      border: none;
      background: #10b981;
      color: #052e16;
      font-weight: 800;
      font-size: 15px;
      cursor: pointer;
    }
    input, select {
      width: 100%;
      min-height: 44px;
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 0 12px;
      font-size: 15px;
      background: #fff;
      color: var(--ink);
      margin-bottom: 10px;
    }
    .tx-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 10px 0;
      border-bottom: 1px solid var(--border);
      gap: 10px;
    }
    .tx-item:last-child { border-bottom: none; }
    .tx-title { font-weight: 600; font-size: 15px; }
    .tx-meta { font-size: 12px; color: var(--muted); margin-top: 2px; }
    .del-btn {
      min-height: 36px;
      padding: 0 10px;
      border: 1px solid #fecaca;
      background: #fef2f2;
      color: var(--danger);
      border-radius: 8px;
      cursor: pointer;
    }
    .action-btn {
      min-height: 42px;
      padding: 0 14px;
      border-radius: 10px;
      border: 1px solid var(--border);
      background: #f8fafc;
      color: var(--ink);
      font-weight: 600;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="summary">
      <div class="sum-box">
        <div class="sum-label">日常收支净额 (MYR)</div>
        <div class="sum-val" id="myrTotal">RM 0.00</div>
      </div>
      <div class="sum-box">
        <div class="sum-label">投资净投入 (USD)</div>
        <div class="sum-val" id="usdTotal">$ 0.00</div>
      </div>
    </div>

    <div class="card">
      <div class="row">
        <button type="button" class="type-btn active" data-type="expense" onclick="setType('expense')">日常开销 (MYR)</button>
        <button type="button" class="type-btn" data-type="income" onclick="setType('income')">日常收入 (MYR)</button>
        <button type="button" class="type-btn" data-type="invest" onclick="setType('invest')">投资交易 (USD)</button>
      </div>

      <div class="amount-bar">
        <button type="button" class="curr-btn" id="currBtn" onclick="toggleCurrency()">MYR ⇄</button>
        <input type="number" step="0.01" inputmode="decimal" class="amt-input" id="amountInput" placeholder="0.00" />
        <button type="button" class="save-btn" onclick="addRecord()">保存</button>
      </div>

      <input type="text" id="noteInput" placeholder="项目 / 备注（如：午餐、美股买入）" />
      <div class="row" style="margin-bottom:0">
        <input type="date" id="dateInput" style="flex:1; margin-bottom:0;" />
        <select id="catInput" style="flex:1; margin-bottom:0;">
          <option value="餐饮">餐饮</option>
          <option value="交通">交通</option>
          <option value="日用/购物">日用/购物</option>
          <option value="账单/杂费">账单/杂费</option>
          <option value="股票/ETF">股票/ETF</option>
          <option value="期权/其他投资">期权/其他投资</option>
        </select>
      </div>
    </div>

    <div class="card">
      <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:8px;">
        <strong>账单记录（本地存储）</strong>
        <button type="button" class="action-btn" onclick="exportJSON()">导出备份</button>
      </div>
      <div id="txList"></div>
    </div>
  </div>

  <script>
    const STORAGE_KEY = 'dual_currency_ledger_v1';
    let currentType = 'expense';
    let currentCurrency = 'MYR';
    let records = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');

    document.getElementById('dateInput').valueAsDate = new Date();

    function setType(type) {
      currentType = type;
      document.querySelectorAll('.type-btn').forEach(b => b.classList.toggle('active', b.dataset.type === type));
      currentCurrency = type === 'invest' ? 'USD' : 'MYR';
      updateCurrBtn();
    }

    function toggleCurrency() {
      currentCurrency = currentCurrency === 'MYR' ? 'USD' : 'MYR';
      updateCurrBtn();
    }

    function updateCurrBtn() {
      document.getElementById('currBtn').textContent = currentCurrency + ' ⇄';
    }

    function addRecord() {
      const amt = parseFloat(document.getElementById('amountInput').value);
      if (isNaN(amt) || amt <= 0) {
        alert('请输入大于 0 的金额');
        return;
      }
      const item = {
        id: Date.now(),
        type: currentType,
        currency: currentCurrency,
        amount: Number(amt.toFixed(2)),
        note: document.getElementById('noteInput').value.trim() || document.getElementById('catInput').value,
        category: document.getElementById('catInput').value,
        date: document.getElementById('dateInput').value || new Date().toISOString().slice(0, 10)
      };
      records.unshift(item);
      localStorage.setItem(STORAGE_KEY, JSON.stringify(records));
      document.getElementById('amountInput').value = '';
      document.getElementById('noteInput').value = '';
      render();
    }

    function deleteRecord(id) {
      records = records.filter(r => r.id !== id);
      localStorage.setItem(STORAGE_KEY, JSON.stringify(records));
      render();
    }

    function exportJSON() {
      const blob = new Blob([JSON.stringify(records, null, 2)], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'ledger-backup.json';
      a.click();
      URL.revokeObjectURL(url);
    }

    function render() {
      let myrNet = 0;
      let usdInvest = 0;
      records.forEach(r => {
        if (r.currency === 'MYR') {
          myrNet += r.type === 'income' ? r.amount : -r.amount;
        } else {
          usdInvest += r.amount;
        }
      });
      document.getElementById('myrTotal').textContent = 'RM ' + myrNet.toFixed(2);
      document.getElementById('usdTotal').textContent = '$ ' + usdInvest.toFixed(2);

      const listEl = document.getElementById('txList');
      if (!records.length) {
        listEl.innerHTML = '<div class="tx-meta">还没有记录，先输入一笔金额点「保存」试试。</div>';
        return;
      }
      listEl.innerHTML = records.map(r => {
        const sign = r.type === 'expense' ? '-' : '+';
        const prefix = r.currency === 'MYR' ? 'RM ' : '$ ';
        return `
          <div class="tx-item">
            <div>
              <div class="tx-title">${r.note}</div>
              <div class="tx-meta">${r.date} · ${r.category} · ${r.currency}</div>
            </div>
            <div style="display:flex; align-items:center; gap:8px;">
              <strong>${sign}${prefix}${r.amount.toFixed(2)}</strong>
              <button type="button" class="del-btn" onclick="deleteRecord(${r.id})">删</button>
            </div>
          </div>
        `;
      }).join('');
    }

    render();
  </script>
</body>
</html>
