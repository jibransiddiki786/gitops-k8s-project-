const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;
const VERSION = process.env.APP_VERSION || 'v1.0.0';
const ENV = process.env.NODE_ENV || 'development';

app.get('/', (req, res) => {
  res.json({
    message: 'GitOps Demo App - Running!',
    version: VERSION,
    environment: ENV,
    timestamp: new Date().toISOString()
  });
});

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy', uptime: process.uptime() });
});

app.get('/ready', (req, res) => {
  res.status(200).json({ status: 'ready' });
});

app.listen(PORT, () => {
  console.log(`App running on port ${PORT} | Version: ${VERSION} | Env: ${ENV}`);
});
console.log("v2!")
console.log("v2!")
