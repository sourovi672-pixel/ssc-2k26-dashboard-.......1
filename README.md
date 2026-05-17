# Quotex Style Candle Trading Website (HTML + CSS + JS)

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Trading Dashboard</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-chart-financial"></script>
<style>
*{
    margin:0;
    padding:0;
    box-sizing:border-box;
    font-family:Arial, sans-serif;
}

body{
    background:#0d1323;
    color:white;
    overflow:hidden;
}

.container{
    display:flex;
    height:100vh;
}

.sidebar{
    width:80px;
    background:#111827;
    display:flex;
    flex-direction:column;
    align-items:center;
    padding-top:20px;
    gap:20px;
    border-right:1px solid rgba(255,255,255,0.08);
}

.logo{
    font-size:26px;
}

.side-btn{
    width:55px;
    height:55px;
    background:#1f2937;
    border-radius:14px;
    display:flex;
    justify-content:center;
    align-items:center;
    cursor:pointer;
    transition:0.3s;
    font-size:22px;
}

.side-btn:hover{
    background:#2563eb;
    transform:scale(1.05);
}

.main{
    flex:1;
    display:flex;
    flex-direction:column;
}

.topbar{
    height:70px;
    background:#111827;
    display:flex;
    align-items:center;
    justify-content:space-between;
    padding:0 20px;
    border-bottom:1px solid rgba(255,255,255,0.08);
}

.market-tabs{
    display:flex;
    gap:12px;
}

.tab{
    background:#1f2937;
    padding:10px 18px;
    border-radius:10px;
    font-size:14px;
    color:#fff;
    cursor:pointer;
    transition:0.3s;
}

.tab:hover{
    background:#2563eb;
}

.balance{
    background:#16a34a;
    padding:10px 20px;
    border-radius:10px;
    font-weight:bold;
}

.content{
    flex:1;
    display:flex;
}

.chart-section{
    flex:1;
    background:#0f172a;
    position:relative;
    padding:20px;
}

canvas{
    width:100% !important;
    height:100% !important;
}

.trade-panel{
    width:260px;
    background:#111827;
    padding:20px;
    border-left:1px solid rgba(255,255,255,0.08);
}

.trade-box{
    background:#1f2937;
    border-radius:18px;
    padding:20px;
}

.trade-box h2{
    margin-bottom:20px;
    font-size:20px;
}

.input-group{
    margin-bottom:18px;
}

.input-group label{
    display:block;
    margin-bottom:8px;
    color:#9ca3af;
}

.input-group input{
    width:100%;
    padding:12px;
    border:none;
    border-radius:10px;
    background:#0f172a;
    color:white;
    font-size:16px;
}

.btn{
    width:100%;
    padding:15px;
    border:none;
    border-radius:12px;
    font-size:18px;
    font-weight:bold;
    cursor:pointer;
    margin-top:12px;
    transition:0.3s;
}

.up{
    background:#16a34a;
    color:white;
}

.down{
    background:#dc2626;
    color:white;
}

.btn:hover{
    opacity:0.8;
    transform:scale(1.02);
}

.live-dot{
    width:10px;
    height:10px;
    background:#22c55e;
    border-radius:50%;
    animation:pulse 1s infinite;
    display:inline-block;
    margin-right:8px;
}

@keyframes pulse{
    0%{transform:scale(1);opacity:1}
    50%{transform:scale(1.7);opacity:0.4}
    100%{transform:scale(1);opacity:1}
}

.price-box{
    position:absolute;
    top:20px;
    left:20px;
    background:#111827;
    padding:10px 16px;
    border-radius:12px;
    font-size:18px;
    font-weight:bold;
    z-index:10;
}

@media(max-width:1000px){
    .trade-panel{
        display:none;
    }

    .sidebar{
        width:65px;
    }
}
</style>
</head>
<body>

<div class="container">

    <div class="sidebar">
        <div class="logo">☰</div>
        <div class="side-btn">📈</div>
        <div class="side-btn">💰</div>
        <div class="side-btn">⚙</div>
        <div class="side-btn">👤</div>
    </div>

    <div class="main">

        <div class="topbar">
            <div class="market-tabs">
                <div class="tab">GBP/USD 80%</div>
                <div class="tab">USD/CAD 77%</div>
                <div class="tab">AUD/USD 85%</div>
            </div>

            <div class="balance">Demo $9999</div>
        </div>

        <div class="content">

            <div class="chart-section">

                <div class="price-box">
                    <span class="live-dot"></span>
                    LIVE MARKET
                </div>

                <canvas id="tradingChart"></canvas>
            </div>

            <div class="trade-panel">

                <div class="trade-box">
                    <h2>Trade Panel</h2>

                    <div class="input-group">
                        <label>Amount</label>
                        <input type="number" value="1">
                    </div>

                    <div class="input-group">
                        <label>Time</label>
                        <input type="text" value="00:01:00">
                    </div>

                    <button class="btn up">UP</button>
                    <button class="btn down">DOWN</button>
                </div>

            </div>

        </div>

    </div>

</div>

<script>
const ctx = document.getElementById('tradingChart');

function randomCandle(prevClose){
    const open = prevClose;
    const close = open + (Math.random() - 0.5) * 10;
    const high = Math.max(open, close) + Math.random() * 5;
    const low = Math.min(open, close) - Math.random() * 5;

    return {
        o: open,
        h: high,
        l: low,
        c: close
    };
}

let data = [];
let lastClose = 100;

for(let i=0;i<30;i++){
    const candle = randomCandle(lastClose);
    lastClose = candle.c;

    data.push({
        x: new Date(Date.now() + i * 60000),
        o: candle.o,
        h: candle.h,
        l: candle.l,
        c: candle.c
    });
}

const chart = new Chart(ctx, {
    type:'candlestick',
    data:{
        datasets:[{
            label:'Market',
            data:data,
            borderColor:'#fff',
            color:{
                up:'#16a34a',
                down:'#ef4444',
                unchanged:'#999'
            }
        }]
    },
    options:{
        responsive:true,
        maintainAspectRatio:false,
        plugins:{
            legend:{display:false}
        },
        scales:{
            x:{
                ticks:{color:'#9ca3af'},
                grid:{color:'rgba(255,255,255,0.05)'}
            },
            y:{
                ticks:{color:'#9ca3af'},
                grid:{color:'rgba(255,255,255,0.05)'}
            }
        }
    }
});

setInterval(()=>{
    const last = data[data.length - 1];

    const candle = randomCandle(last.c);

    data.push({
        x:new Date(),
        o:candle.o,
        h:candle.h,
        l:candle.l,
        c:candle.c
    });

    if(data.length > 40){
        data.shift();
    }

    chart.update();
},2000);
</script>

</body>
</html>
```

## Run করার নিয়ম

1. `index.html` নামে save করো
2. Browser এ open করো
3. Live candle chart দেখতে পাবে
4. Mobile + PC দুইটাতেই কাজ করবে

## Features

* Real style candle chart
* Green/Red live candles
* Quotex style UI
* Trading buttons
* Dark futuristic theme
* Responsive design
* Animated live market effect
