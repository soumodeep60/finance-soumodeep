# Soumodeep Finance Website - Deployment Guide

## Overview
A professional finance website with black and white aesthetic featuring stock analysis, quantitative modeling, financial dictionary, and contact form.

## Features Implemented

### ✅ Tab 1: Stock Overview
- Company details display
- Outstanding shares information
- Price chart placeholder (ready for API integration)
- Latest news sections (Industry, Economy, Company)
- 3-year financial statements (Income Statement, Balance Sheet, Cash Flow)
- Company valuation (Fair/High/Low)
- Buy/Sell/Hold verdict

### ✅ Tab 2: Quant Finance Modeling
- Security type selection (Stock, Option, Futures)
- Quant model selection:
  - Black-Scholes Model
  - Monte Carlo Simulation
  - Binomial Options Pricing
  - Value at Risk (VaR)
  - CAPM Model
- Time series prediction with moving average choices:
  - Simple Moving Average (SMA)
  - Exponential Moving Average (EMA)
  - ARIMA
  - GARCH
  - LSTM Neural Network

### ✅ Tab 3: Financial Terms Dictionary
- Investopedia-style definitions
- Search functionality
- Popular terms quick access
- 15+ pre-loaded financial terms

### ✅ Tab 4: Contact Us
- Contact form with validation
- Name, Email, Subject, Message fields
- Success confirmation
- Additional contact information display

### ✅ Design Elements
- Black and white theme
- Times New Roman font throughout
- Animated background pattern
- Responsive design
- Professional footer with multiple sections

## Deploying to Railway

### Option 1: Static HTML Deployment

1. **Prepare your project:**
   ```bash
   mkdir soumodeep-finance
   cd soumodeep-finance
   # Copy the HTML file to this directory as index.html
   ```

2. **Initialize Git:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```

3. **Deploy to Railway:**
   - Go to [railway.app](https://railway.app)
   - Sign up or log in
   - Click "New Project"
   - Select "Deploy from GitHub repo"
   - Connect your repository
   - Railway will auto-detect and deploy

### Option 2: Node.js Backend (Recommended for API Integration)

1. **Create package.json:**
```json
{
  "name": "soumodeep-finance",
  "version": "1.0.0",
  "description": "Financial analysis platform",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.6.0",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1"
  }
}
```

2. **Create server.js:**
```javascript
const express = require('express');
const axios = require('axios');
const cors = require('cors');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());
app.use(express.static('.'));

// API Routes
app.get('/api/stock/:ticker', async (req, res) => {
  const { ticker } = req.params;
  try {
    // Integrate your APIs here
    const data = await fetchStockData(ticker);
    res.json(data);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch stock data' });
  }
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

3. **Create .env file:**
```
ALPHAVANTAGE_API_KEY=your_key_here
FINNHUB_API_KEY=your_key_here
PORT=3000
```

4. **Deploy to Railway:**
```bash
npm install
git add .
git commit -m "Add backend"
railway login
railway init
railway up
```

## API Integration Guide

### Yahoo Finance API (Free)
```javascript
// Using yfinance or yahoo-finance2 npm package
const yahooFinance = require('yahoo-finance2').default;

async function getStockData(ticker) {
  const quote = await yahooFinance.quote(ticker);
  const chart = await yahooFinance.chart(ticker, {
    period1: '2023-01-01',
    interval: '1d'
  });
  return { quote, chart };
}
```

### Finnhub API
```javascript
const axios = require('axios');

async function getFinnhubData(ticker) {
  const apiKey = process.env.FINNHUB_API_KEY;
  const response = await axios.get(
    `https://finnhub.io/api/v1/quote?symbol=${ticker}&token=${apiKey}`
  );
  return response.data;
}

// Get company news
async function getCompanyNews(ticker) {
  const apiKey = process.env.FINNHUB_API_KEY;
  const today = new Date().toISOString().split('T')[0];
  const lastWeek = new Date(Date.now() - 7*24*60*60*1000).toISOString().split('T')[0];
  
  const response = await axios.get(
    `https://finnhub.io/api/v1/company-news?symbol=${ticker}&from=${lastWeek}&to=${today}&token=${apiKey}`
  );
  return response.data;
}
```

### Alpha Vantage API
```javascript
async function getAlphaVantageData(ticker) {
  const apiKey = process.env.ALPHAVANTAGE_API_KEY;
  
  // Get quote
  const quoteUrl = `https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=${ticker}&apikey=${apiKey}`;
  const quote = await axios.get(quoteUrl);
  
  // Get financial statements
  const incomeUrl = `https://www.alphavantage.co/query?function=INCOME_STATEMENT&symbol=${ticker}&apikey=${apiKey}`;
  const income = await axios.get(incomeUrl);
  
  const balanceUrl = `https://www.alphavantage.co/query?function=BALANCE_SHEET&symbol=${ticker}&apikey=${apiKey}`;
  const balance = await axios.get(balanceUrl);
  
  const cashFlowUrl = `https://www.alphavantage.co/query?function=CASH_FLOW&symbol=${ticker}&apikey=${apiKey}`;
  const cashFlow = await axios.get(cashFlowUrl);
  
  return {
    quote: quote.data,
    income: income.data,
    balance: balance.data,
    cashFlow: cashFlow.data
  };
}
```

## Integrating APIs into Frontend

Update the `handleSearch` function in the HTML file:

```javascript
const handleSearch = async () => {
  if (!ticker.trim()) {
    setError('Please enter a ticker symbol');
    return;
  }

  setLoading(true);
  setError('');
  setSearchedTicker(ticker.toUpperCase());

  try {
    // Call your backend API
    const response = await fetch(`/api/stock/${ticker}`);
    
    if (!response.ok) {
      throw new Error('Failed to fetch data');
    }
    
    const data = await response.json();
    setStockData(data);
    setLoading(false);
  } catch (err) {
    setError('Failed to fetch stock data. Please try again.');
    setLoading(false);
  }
};
```

## Adding Chart Visualization

For real-time charts, integrate Chart.js or Recharts:

```javascript
// Example with Chart.js
function PriceChart({ data }) {
  const chartRef = useRef(null);
  
  useEffect(() => {
    if (!data || !chartRef.current) return;
    
    const ctx = chartRef.current.getContext('2d');
    new Chart(ctx, {
      type: 'line',
      data: {
        labels: data.dates,
        datasets: [{
          label: 'Price',
          data: data.prices,
          borderColor: '#ffffff',
          backgroundColor: 'rgba(255, 255, 255, 0.1)',
          borderWidth: 2
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            labels: { color: '#ffffff' }
          }
        },
        scales: {
          x: {
            ticks: { color: '#a0a0a0' },
            grid: { color: '#333333' }
          },
          y: {
            ticks: { color: '#a0a0a0' },
            grid: { color: '#333333' }
          }
        }
      }
    });
  }, [data]);
  
  return <canvas ref={chartRef}></canvas>;
}
```

## Implementing Quant Models

### Black-Scholes Model Example:
```javascript
function blackScholes(S, K, T, r, sigma, type = 'call') {
  // S: Current stock price
  // K: Strike price
  // T: Time to expiration (years)
  // r: Risk-free rate
  // sigma: Volatility
  
  const d1 = (Math.log(S / K) + (r + 0.5 * sigma ** 2) * T) / (sigma * Math.sqrt(T));
  const d2 = d1 - sigma * Math.sqrt(T);
  
  function normalCDF(x) {
    return (1 + erf(x / Math.sqrt(2))) / 2;
  }
  
  if (type === 'call') {
    return S * normalCDF(d1) - K * Math.exp(-r * T) * normalCDF(d2);
  } else {
    return K * Math.exp(-r * T) * normalCDF(-d2) - S * normalCDF(-d1);
  }
}
```

## Environment Variables for Railway

In Railway dashboard:
1. Go to your project
2. Click "Variables"
3. Add:
   - `ALPHAVANTAGE_API_KEY`
   - `FINNHUB_API_KEY`
   - `NODE_ENV=production`

## Next Steps

1. ✅ Get API keys:
   - Alpha Vantage: https://www.alphavantage.co/support/#api-key
   - Finnhub: https://finnhub.io/register

2. ✅ Set up backend (server.js)

3. ✅ Replace mock data with real API calls

4. ✅ Implement chart visualizations

5. ✅ Add quant model calculations

6. ✅ Test locally: `npm start`

7. ✅ Deploy to Railway

8. ✅ Configure custom domain (optional)

## Performance Optimization

- Cache API responses (Redis/Memory)
- Rate limit API calls
- Implement loading states
- Add error boundaries
- Optimize images and assets

## Security Considerations

- Never expose API keys in frontend
- Use environment variables
- Implement CORS properly
- Validate all inputs
- Rate limit contact form

## Support

For issues or questions:
- Email: contact@soumodeep.finance
- Documentation: Check Railway docs at railway.app/docs

---

**Built with:**
- React 18
- Express.js
- Chart.js
- Times New Roman Typography
- Professional Black & White Design
