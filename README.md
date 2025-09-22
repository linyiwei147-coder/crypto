<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>比特币模拟交易</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 20px; background: #f0f2f5; }
        #chartContainer { width: 80%; margin: auto; }
        input { padding: 5px; margin: 5px; width: 100px; }
        button { padding: 5px 10px; margin: 5px; }
        #info { margin-top: 20px; }
    </style>
</head>
<body>
    <h1>比特币模拟交易</h1>

    <div id="chartContainer">
        <canvas id="btcChart"></canvas>
    </div>

    <div id="trade">
        <h3>虚拟余额: $<span id="balance"></span></h3>
        <h3>持有比特币: <span id="btcAmount"></span> BTC</h3>
        <input type="number" id="tradeAmount" placeholder="数量">
        <button onclick="buy()">买入</button>
        <button onclick="sell()">卖出</button>
    </div>

    <div id="info"></div>

    <script>
        // 初始虚拟资金和持仓
        let balance = parseFloat(localStorage.getItem('balance')) || 10000;
        let btcAmount = parseFloat(localStorage.getItem('btcAmount')) || 0;

        const balanceEl = document.getElementById('balance');
        const btcAmountEl = document.getElementById('btcAmount');
        const infoEl = document.getElementById('info');

        balanceEl.innerText = balance.toFixed(2);
        btcAmountEl.innerText = btcAmount.toFixed(4);

        // 图表数据
        let labels = [];
        let data = [];
        const ctx = document.getElementById('btcChart').getContext('2d');
        const btcChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    label: 'BTC/USD',
                    data: data,
                    borderColor: 'rgba(255, 99, 132, 1)',
                    tension: 0.2
                }]
            },
            options: {
                responsive: true,
                scales: {
                    x: { display: true, title: { display: true, text: '时间' } },
                    y: { display: true, title: { display: true, text: '价格(USD)' } }
                }
            }
        });

        // 获取价格
        async function fetchPrice() {
            try {
                const res = await fetch('https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd');
                const json = await res.json();
                const price = json.bitcoin.usd;
                const now = new Date().toLocaleTimeString();

                if(labels.length > 20){ 
                    labels.shift();
                    data.shift();
                }
                labels.push(now);
                data.push(price);
                btcChart.update();

                return price;
            } catch (err) {
                console.error('获取价格失败', err);
            }
        }

        // 买入
        async function buy() {
            const amt = parseFloat(document.getElementById('tradeAmount').value);
            const price = await fetchPrice();
            if(!amt || amt <= 0){ infoEl.innerText = '请输入正确数量'; return; }
            const cost = amt * price;
            if(cost > balance){ infoEl.innerText = '余额不足'; return; }
            balance -= cost;
            btcAmount += amt;
            balanceEl.innerText = balance.toFixed(2);
            btcAmountEl.innerText = btcAmount.toFixed(4);
            infoEl.innerText = `买入成功，花费 $${cost.toFixed(2)}`;
            saveData();
        }

        // 卖出
        async function sell() {
            const amt = parseFloat(document.getElementById('tradeAmount').value);
            const price = await fetchPrice();
            if(!amt || amt <= 0){ infoEl.innerText = '请输入正确数量'; return; }
            if(amt > btcAmount){ infoEl.innerText = '比特币不足'; return; }
            const earn = amt * price;
            balance += earn;
            btcAmount -= amt;
            balanceEl.innerText = balance.toFixed(2);
            btcAmountEl.innerText = btcAmount.toFixed(4);
            infoEl.innerText = `卖出成功，获得 $${earn.toFixed(2)}`;
            saveData();
        }

        function saveData() {
            localStorage.setItem('balance', balance);
            localStorage.setItem('btcAmount', btcAmount);
        }

        // 每5秒更新价格
        fetchPrice();
        setInterval(fetchPrice, 5000);
    </script>
</body>
</html>
