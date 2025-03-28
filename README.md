<!-- 行情展示容器 -->
<div class="fx-market">
  <div class="instrument" data-symbol="EURUSD"></div>
  <div class="instrument" data-symbol="GBPUSD"></div>
  <div class="instrument" data-symbol="AUDUSD"></div>
  <div class="instrument" data-symbol="USDCAD"></div>
  <div class="instrument" data-symbol="USDCHF"></div>
  <div class="instrument" data-symbol="USDCNY"></div>
  <div class="instrument" data-symbol="USDJPY"></div>
  <div class="instrument" data-symbol="XAUUSD"></div>
  <div class="instrument" data-symbol="BTCUSD"></div>
</div>

<script>
const API_KEY = 'j9Lqn6hBZeHZVK5r3bSA_h6ICS_API_KEY'; // ⚠️ 建议放到后端

// 统一数据获取函数
async function getFXData(symbol) {
  try {
    const response = await fetch(
      `https://marketdataapi.com/api/v1/asset/${symbol}/quote?apikey=${API_KEY}`
    );
    if (!response.ok) throw new Error(`API错误: ${response.status}`);
    const data = await response.json();

    return {
      price: data.last_price || 'N/A',
      change: data.change_percent || 0,
      time: new Date(data.timestamp * 1000)
    };
  } catch (error) {
    console.error(`获取 ${symbol} 数据失败:`, error);
    return null;
  }
}

// 动态更新行情
async function updateMarket() {
  document.querySelectorAll('.instrument').forEach(async (item) => {
    const symbol = item.dataset.symbol;
    item.innerHTML = `<div class="loading">加载中...</div>`; // 加载提示
    const data = await getFXData(symbol);
    
    if (!data || data.price === 'N/A') {
      item.innerHTML = `<div class="error">${symbol} 数据暂不可用</div>`;
      return;
    }

    item.innerHTML = `
      <div class="symbol">${symbol}</div>
      <div class="price ${data.change >= 0 ? 'rise' : 'fall'}">
        ${formatPrice(symbol, data.price)}
      </div>
      <div class="footer">
        <span class="change">${data.change.toFixed(2)}%</span>
        <span class="time">${data.time.toLocaleTimeString()}</span>
      </div>
    `;
  });
}

// 价格格式化
function formatPrice(symbol, price) {
  const isCrypto = ['BTCUSD'].includes(symbol);
  const isMetal = ['XAUUSD'].includes(symbol);
  return (isCrypto ? '₿' : isMetal ? 'XAU ' : '$') + parseFloat(price).toFixed(isCrypto ? 2 : 4);
}

// 初始加载 + 每15秒更新
updateMarket();
setInterval(updateMarket, 15000);
</script>

<style>
.fx-market {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 12px;
  padding: 20px;
  background: #f8fafc;
}

.instrument {
  background: white;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
  text-align: center;
}

.symbol {
  font-weight: 600;
  color: #1e293b;
  margin-bottom: 8px;
}

.price {
  font-size: 1.5em;
  font-family: 'Courier New', monospace;
}

.price.rise { color: #10b981; }
.price.fall { color: #ef4444; }

.footer {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
  font-size: 0.85em;
  color: #64748b;
}

.loading {
  color: #64748b;
  font-size: 1em;
  font-style: italic;
}

.error {
  color: #dc2626;
  padding: 10px;
}

@media (max-width: 640px) {
  .fx-market {
    grid-template-columns: 1fr 1fr;
  }
  .price {
    font-size: 1.3em;
  }
}
</style>
