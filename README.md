<div class="fx-market">
  <!-- 保持原有DOM结构 -->
</div>

<script>
// 通过代理服务器隐藏真实API密钥
const PROXY_URL = '/api/fxmarket';

// 支持的所有交易品种
const SYMBOLS = [
  'EURUSD', 'GBPUSD', 'AUDUSD', 
  'USDCAD', 'USDCHF', 'USD/CNY',  // 使用API接受的符号格式
  'USDJPY', 'XAU/USD', 'BTC/USD'
];

// 批量获取数据
async function fetchMarketData() {
  try {
    const response = await fetch(`${PROXY_URL}?symbols=${SYMBOLS.join(',')}`);
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    return await response.json();
  } catch (error) {
    console.error('市场数据获取失败:', error);
    return null;
  }
}

// 数据更新逻辑
async function updateMarket() {
  const marketData = await fetchMarketData();
  
  SYMBOLS.forEach(symbol => {
    const element = document.querySelector(`[data-symbol="${symbol.replace('/', '')}"]`);
    if (!element) return;

    const data = marketData?.[symbol];
    if (!data || !data.price) {
      element.innerHTML = `<div class="error">${symbol} 数据暂不可用</div>`;
      return;
    }

    element.innerHTML = `
      <div class="symbol">${formatSymbolDisplay(symbol)}</div>
      <div class="price ${data.change >= 0 ? 'rise' : 'fall'}">
        ${formatPrice(symbol, data.price)}
      </div>
      <div class="footer">
        <span class="change">${data.change?.toFixed(2) ?? '--'}%</span>
        <span class="time">${new Date(data.timestamp).toLocaleTimeString()}</span>
      </div>
    `;
  });
}

// 符号显示格式化
function formatSymbolDisplay(symbol) {
  const pairMap = {
    'USD/CNY': 'USD/CNY',
    'XAU/USD': '黄金/USD',
    'BTC/USD': '比特币/USD'
  };
  return pairMap[symbol] || symbol.replace('/', ' / ');
}

// 价格格式化增强
function formatPrice(symbol, price) {
  const formatter = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: symbol.endsWith('USD') ? 'USD' : 'CNY',
    minimumFractionDigits: symbol.includes('JPY') ? 2 : 4,
    currencyDisplay: 'narrowSymbol'
  });
  
  if (symbol.startsWith('XAU')) return `XAU ${price.toFixed(2)}`;
  if (symbol.startsWith('BTC')) return `₿${price.toFixed(2)}`;
  
  return formatter.format(price);
}

// 初始化
document.addEventListener('DOMContentLoaded', () => {
  updateMarket();
  setInterval(updateMarket, 20000); // 调整为20秒更新
});
</script>

<style>
/* 新增移动端优化样式 */
@media (max-width: 480px) {
  .fx-market {
    grid-template-columns: 1fr;
  }
  
  .instrument {
    padding: 12px;
  }
  
  .price {
    font-size: 1.2em;
  }
}

/* 新增加载动画 */
.loading::after {
  content: "";
  display: inline-block;
  width: 1em;
  height: 1em;
  border: 2px solid #ddd;
  border-radius: 50%;
  border-top-color: #666;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
