<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <title>S.trading — Candles Demo (Simulated Trading)</title>
  <style>
    :root{
      --bg:#0c1218; --panel:#0b1116; --muted:#9fb0c8; --accent:#0A84FF; --green:#1fc27b; --red:#ff6b6b;
    }
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,Segoe UI,Roboto,Arial;background:linear-gradient(180deg,#071026,#0b1220);color:#e6eef8}
    .frame{max-width:1200px;margin:18px auto;border-radius:8px;overflow:hidden}
    .topbar{display:flex;justify-content:space-between;padding:12px 16px;background:rgba(255,255,255,0.02);align-items:center}
    .left{display:flex;align-items:center;gap:10px}
    .logo{width:36px;height:36px;border-radius:6px;background:linear-gradient(90deg,var(--accent),#0066D6);display:flex;align-items:center;justify-content:center;font-weight:700}
    .title{font-weight:700}
    .main{display:flex;gap:12px;padding:14px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00))}
    .chart-area{flex:1;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));border-radius:8px;padding:8px;position:relative;height:540px}
    .panel{width:300px;background:var(--panel);padding:12px;border-radius:8px;color:var(--muted)}
    .pair{font-weight:700;margin-bottom:6px}
    .price{font-size:20px;margin-bottom:12px}
    .controls{display:flex;gap:8px;margin-bottom:12px}
    button{background:var(--accent);border:none;color:#fff;padding:8px;border-radius:6px;cursor:pointer}
    button[disabled]{opacity:0.5}
    .order-section label{display:block;margin:8px 0;color:var(--muted);font-size:13px}
    input,select{width:100%;padding:8px;border-radius:6px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:#fff}
    .place{margin-top:8px;background:linear-gradient(90deg,var(--accent),#0066D6);border:none;padding:10px;border-radius:6px;cursor:pointer}
    #orders{list-style:none;padding:0;margin:0;max-height:180px;overflow:auto}
    #orders li{padding:8px;border-bottom:1px dashed rgba(255,255,255,0.03);font-size:13px;color:var(--muted)}
    .price-line{position:absolute;right:12px;top:50%;transform:translateY(-50%);background:rgba(10,132,255,0.12);padding:6px 10px;border-radius:12px;color:#fff;border:1px solid rgba(10,132,255,0.35)}
    .foot{text-align:center;padding:12px;color:var(--muted);font-size:13px;margin-top:8px}
    /* ensure canvas fills area */
    #candleChart{width:100% !important; height:100% !important; display:block}
  </style>
  <!-- Chart.js CDN -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
</head>
<body>
  <div class="frame">
    <header class="topbar">
      <div class="left"><div class="logo">SA</div><div class="title">S.trading</div></div>
      <div class="right">Demo Candles</div>
    </header>

    <main class="main">
      <section class="chart-area">
        <canvas id="candleChart"></canvas>
        <div class="price-line" id="priceLine"></div>
      </section>

      <aside class="panel">
        <div class="pair">BTC/USDT</div>
        <div class="price" id="livePrice">29,500.00</div>

        <div class="controls">
          <button id="start">Start</button>
          <button id="stop" disabled>Stop</button>
          <button id="reset">Reset</button>
        </div>

        <div class="order-section">
          <h4>Place Order (demo)</h4>
          <label>Type
            <select id="orderType"><option value="buy">Buy</option><option value="sell">Sell</option></select>
          </label>
          <label>Amount <input id="amt" type="number" value="0.01" step="0.01"></label>
          <label>Price <input id="priceIn" type="number" value="29500" step="0.01"></label>
          <button id="place" class="place">Place Order</button>
          <h4>Orders</h4>
          <ul id="orders"></ul>
        </div>
      </aside>
    </main>
    <footer class="foot">Demo only — no real trading</footer>
  </div>

  <script>
    // high-fidelity candlestick simulation + demo matching for frontend-only trading
    const ctx = document.getElementById('candleChart').getContext('2d');
    let running=false, feedInterval=null;
    const livePriceEl = document.getElementById('livePrice');
    const priceLineEl = document.getElementById('priceLine');
    const ordersEl = document.getElementById('orders');

    // generate initial OHLC data
    function genOHLC(count=80, start=29500){
      const out=[]; let price=start;
      for(let i=0;i<count;i++){
        const o = price;
        const c = price * (1 + (Math.random()-0.5)/40);
        const h = Math.max(o,c) * (1 + Math.random()/80);
        const l = Math.min(o,c) * (1 - Math.random()/80);
        out.push({t: new Date(Date.now() - (count-i)*60000), o, h, l, c, v: Math.floor(Math.random()*1200)});
        price = c;
      }
      return out;
    }
    let candles = genOHLC(80,29500);

    // build chart
    function buildChart(){
      const labels = candles.map(d=>d.t.toLocaleTimeString());
      const closeVals = candles.map(d=>d.c);
      const bgColors = candles.map(d=> d.c>=d.o ? '#1fc27b' : '#ff6b6b');

      if(window.candleChart) window.candleChart.destroy();

      window.candleChart = new Chart(ctx, {
        type:'bar',
        data:{
          labels,
          datasets:[
            {
              type:'line',
              label:'wickHigh',
              data: candles.map(d=>d.h),
              borderColor:'#9fb0c8',
              borderWidth:1,
              pointRadius:0,
              tension:0,
              yAxisID:'y'
            },
            {
              type:'bar',
              label:'body',
              data: closeVals,
              backgroundColor:bgColors,
              barPercentage:1.0,
              categoryPercentage:1.0,
              borderRadius:3,
              yAxisID:'y'
            }
          ]
        },
        options:{
          animation:false,
          responsive:true,
          maintainAspectRatio:false,
          scales:{
            x:{display:false},
            y:{
              grid:{color:'rgba(255,255,255,0.03)'},
              position:'right'
            }
          },
          plugins:{
            tooltip:{
              enabled:true,
              callbacks:{
                title:(items)=> candles[items[0].dataIndex].t.toLocaleString(),
                label:(ctx)=>{
                  const i = ctx.dataIndex; const d = candles[i];
                  return ['O: '+d.o.toFixed(2), 'H: '+d.h.toFixed(2), 'L: '+d.l.toFixed(2), 'C: '+d.c.toFixed(2), 'V: '+d.v];
                }
              },
              backgroundColor:'#0b1116',
              borderColor:'rgba(255,255,255,0.06)',
              borderWidth:1,
              titleColor:'#fff',
              bodyColor:'#cfe7ff'
            },
            legend:{display:false}
          },
          interaction:{mode:'index',intersect:false}
        }
      });
      livePriceEl.textContent = candles[candles.length-1].c.toFixed(2);
      priceLineEl.textContent = '$' + candles[candles.length-1].c.toFixed(2);
    }
    buildChart();

    // feed: append new candle periodically
    function pushCandle(){
      const last = candles[candles.length-1];
      const o = last.c;
      const c = o * (1 + (Math.random()-0.5)/40);
      const h = Math.max(o,c)*(1 + Math.random()/80);
      const l = Math.min(o,c) * (1 - Math.random()/80);
      const v = Math.floor(Math.random()*1200);
      const next = {t:new Date(), o,h,l,c,v};
      candles.push(next); if(candles.length>120) candles.shift();

      candleChart.data.labels = candles.map(d=>d.t.toLocaleTimeString());
      candleChart.data.datasets[0].data = candles.map(d=>d.h);
      candleChart.data.datasets[1].data = candles.map(d=>d.c);
      candleChart.data.datasets[1].backgroundColor = candles.map(d=> d.c>=d.o ? '#1fc27b' : '#ff6b6b');
      candleChart.update('none');

      livePriceEl.textContent = next.c.toFixed(2);
      priceLineEl.textContent = '$' + next.c.toFixed(2);
    }

    // controls
    document.getElementById('start').onclick = ()=>{
      if(running) return;
      running=true; document.getElementById('start').disabled=true; document.getElementById('stop').disabled=false;
      feedInterval = setInterval(pushCandle,1500);
    };
    document.getElementById('stop').onclick = ()=>{
      running=false; clearInterval(feedInterval); document.getElementById('start').disabled=false; document.getElementById('stop').disabled=true;
    };
    document.getElementById('reset').onclick = ()=>{
      clearInterval(feedInterval); running=false; document.getElementById('start').disabled=false; document.getElementById('stop').disabled=true;
      candles = genOHLC(80,29500); buildChart();
    };

    // simple demo matching: place orders and execute immediately if price compatible
    document.getElementById('place').onclick = ()=>{
      const type = document.getElementById('orderType').value;
      const amt = parseFloat(document.getElementById('amt').value)||0;
      const price = parseFloat(document.getElementById('priceIn').value)||0;
      const cur = candles[candles.length-1].c;
      const li = document.createElement('li');
      const time = new Date().toLocaleTimeString();

      // simple immediate execution rules for demo:
      let status='Placed';
      if((type==='buy' && price >= cur) || (type==='sell' && price <= cur)){
        status = 'Executed';
      }
      li.textContent = `${time} — ${type.toUpperCase()} ${amt} @ ${price.toFixed(2)} — ${status}`;
      ordersEl.prepend(li);
      // if executed, push small price impact
      if(status==='Executed'){
        const impact = (type==='buy'?1: -1) * (amt/10) * (Math.random()/100);
        candles[candles.length-1].c = candles[candles.length-1].c * (1 + impact);
        buildChart();
      }
    };

    // basic crosshair: show horizontal price badge following mouse Y
    const canvas = document.getElementById('candleChart');
    canvas.addEventListener('mousemove', (e)=>{
      const rect = canvas.getBoundingClientRect();
      const y = e.clientY - rect.top;
      const chartArea = candleChart.chartArea;
      if(y < chartArea.top || y > chartArea.bottom) { priceLineEl.style.display='none'; return; }
      priceLineEl.style.display='block';
      const scale = candleChart.scales.y;
      const val = scale.getValueForPixel(y);
      priceLineEl.textContent = '$' + val.toFixed(2);
      priceLineEl.style.top = (y - 12) + 'px';
    });
    // hide on leave
    canvas.addEventListener('mouseleave', ()=>{ priceLineEl.style.display='none'; });
    canvas.addEventListener('mouseenter', ()=>{ priceLineEl.style.display='block'; });
  </script>
</body>
</html>
