<!DOCTYPE html>
<html>
<head>
  <title>Orderbook Viewer</title>
  <style>
    body{font-family:Arial;margin:20px;max-width:900px;}
    #controls{margin-bottom:15px;}
    #symbol{width:120px;padding:5px;font-size:1em;}
    #load{padding:5px 10px;font-size:1em;}
    #status{margin-top:10px;color:#555;}
    table{border-collapse:collapse;width:100%;}
    th,td{border:1px solid #ccc;padding:4px;}
    th{background:#f0f0f0;text-align:left;}
    tr:nth-child(odd){background:#f9f9f9;}
  </style>
</head>
<body>
  <h1>Orderbook Viewer</h1>
  <div id="controls">
    <label for="symbol">Asset (e.g., XAUUSD, BTCUSDT): </label>
    <input type="text" id="symbol" value="XAUUSD">
    <button id="load">Load</button>
  </div>
  <p id="status"></p>
  <table id="orderbook">
    <thead><tr><th>Ask (price / qty)</th><th>Bid (price / qty)</th></tr></thead>
    <tbody></tbody>
  </table>

  <script>
    const symIn = document.getElementById('symbol');
    const loadBtn = document.getElementById('load');
    const status = document.getElementById('status');
    const tbody = document.querySelector('#orderbook tbody');

    let pollTimer;

    function fetchOrderbook() {
      const sym = symIn.value.trim().toUpperCase();
      if (!sym) { status.textContent = 'Please enter a symbol'; return; }
      status.textContent = 'Loading…';
      fetch(`https://api.binance.com/fapi/v1/depth?symbol=${sym}&limit=10`)
        .then(resp => { if (!resp.ok) throw new Error('Failed fetch'); return resp.json(); })
        .then(data => {
          const bids = data.bids.map(d => [parseFloat(d[0]), parseFloat(d[1])]);
          const asks = data.asks.map(d => [parseFloat(d[0]), parseFloat(d[1])]);

          tbody.innerHTML = '';
          const rows = Math.max(bids.length, asks.length);
          for (let i = 0; i < rows; i++) {
            const tr = document.createElement('tr');
            const askCell = document.createElement('td');
            const bidCell = document.createElement('td');
            askCell.textContent = i < asks.length ? `${asks[i][0]} (${asks[i][1]})` : '-';
            bidCell.textContent = i < bids.length ? `${bids[bids.length - 1 - i][0]} (${bids[bids.length - 1 - i][1]})` : '-';
            tr.appendChild(askCell);
            tr.appendChild(bidCell);
            tbody.appendChild(tr);
          }
          status.textContent = `Orderbook for ${sym}`;
        })
        .catch(err => { status.textContent = 'Error: ' + err.message; });
    }

    function startPolling() {
      clearInterval(pollTimer);
      pollTimer = setInterval(fetchOrderbook, 2000);   // 2‑second interval
    }

    loadBtn.onclick = () => { fetchOrderbook(); startPolling(); };

    // Load once immediately on page load
    fetchOrderbook();
    startPolling();
  </script>
</body>
</html>
