// 使用Express.js搭建简易代理（部署到Heroku/Vercel）
const express = require('express');
const axios = require('axios');
const app = express();

app.get('/fx-proxy', async (req, res) => {
  try {
    const symbol = req.query.symbol;
    const response = await axios.get(
      `https://marketdataapi.com/api/v1/asset/${symbol}/quote`,
      { params: { apikey: process.env.API_KEY } }
    );
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: '数据获取失败' });
  }
});

app.listen(process.env.PORT || 3000);
