If you're looking for free Python APIs to query financial data such as stock prices, company information, and financial statements, here are some excellent options:

---

### **1. Alpha Vantage**
- **Features**: Stock prices, forex, cryptocurrency, technical indicators, and fundamental data.
- **Usage**:  
  ```python
  import requests

  API_KEY = 'your_api_key'
  symbol = 'AAPL'
  url = f'https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol={symbol}&apikey={API_KEY}'
  response = requests.get(url).json()
  print(response)
  ```
- **Pros**: Free tier includes 5 API calls per minute, supports fundamental and technical analysis data.
- **Cons**: Limited data frequency in free tier.
- **Website**: [Alpha Vantage](https://www.alphavantage.co)

---

### **2. Yahoo Finance (via yfinance Python Library)**
- **Features**: Historical stock prices, dividends, splits, financial statements, and more.
- **Usage**:  
  ```python
  import yfinance as yf

  ticker = yf.Ticker('AAPL')
  print(ticker.info)  # General company info
  print(ticker.history(period="1mo"))  # Historical data
  ```
- **Pros**: Easy to use, no API key required.
- **Cons**: Limited data customization compared to paid services.
- **Library**: [yfinance on PyPI](https://pypi.org/project/yfinance/)

---

### **3. Financial Modeling Prep (FMP)**
- **Features**: Company financials, stock data, economic indicators, ETFs, and forex data.
- **Usage**:  
  ```python
  import requests

  API_KEY = 'your_api_key'
  symbol = 'AAPL'
  url = f'https://financialmodelingprep.com/api/v3/quote/{symbol}?apikey={API_KEY}'
  response = requests.get(url).json()
  print(response)
  ```
- **Pros**: Generous free tier, wide variety of data endpoints.
- **Cons**: API limits for free users.
- **Website**: [Financial Modeling Prep](https://financialmodelingprep.com/)

---

### **4. Quandl**
- **Features**: Stock data, alternative datasets, and economic indicators.
- **Usage**:  
  ```python
  import quandl

  quandl.ApiConfig.api_key = 'your_api_key'
  data = quandl.get("WIKI/AAPL")
  print(data.head())
  ```
- **Pros**: High-quality curated datasets.
- **Cons**: Some datasets require paid subscription.
- **Website**: [Quandl](https://www.quandl.com/)

---

### **5. IEX Cloud**
- **Features**: Real-time stock prices, company data, and news.
- **Usage**:  
  ```python
  import requests

  API_KEY = 'your_api_key'
  symbol = 'AAPL'
  url = f'https://cloud.iexapis.com/stable/stock/{symbol}/quote?token={API_KEY}'
  response = requests.get(url).json()
  print(response)
  ```
- **Pros**: Free tier for individual developers; real-time data included.
- **Cons**: Some endpoints require a paid subscription.
- **Website**: [IEX Cloud](https://iexcloud.io/)

---

### **6. Twelve Data**
- **Features**: Stock prices, forex, crypto, and technical indicators.
- **Usage**:  
  ```python
  import requests

  API_KEY = 'your_api_key'
  symbol = 'AAPL'
  url = f'https://api.twelvedata.com/time_series?symbol={symbol}&interval=1day&apikey={API_KEY}'
  response = requests.get(url).json()
  print(response)
  ```
- **Pros**: Free tier includes 8 API calls per minute and technical indicators.
- **Cons**: Limited data depth on the free tier.
- **Website**: [Twelve Data](https://twelvedata.com/)

---

### **7. Polygon.io (Free Tier)**
- **Features**: Stock quotes, trades, forex, and cryptocurrency data.
- **Usage**:  
  ```python
  import requests

  API_KEY = 'your_api_key'
  url = f'https://api.polygon.io/v3/reference/tickers/AAPL?apiKey={API_KEY}'
  response = requests.get(url).json()
  print(response)
  ```
- **Pros**: Low-latency data for developers.
- **Cons**: Limited data for free tier users.
- **Website**: [Polygon.io](https://polygon.io/)

---

### **8. OpenFIGI**
- **Features**: Financial instrument identifiers (ISINs, CUSIPs, Bloomberg FIGIs).
- **Usage**:  
  ```python
  import requests

  headers = {'Content-Type': 'application/json'}
  data = [{"idType": "TICKER", "idValue": "AAPL"}]
  url = 'https://api.openfigi.com/v2/mapping'
  response = requests.post(url, json=data, headers=headers)
  print(response.json())
  ```
- **Pros**: Instrument mapping and cross-referencing.
- **Cons**: Limited focus compared to broader APIs.
- **Website**: [OpenFIGI](https://www.openfigi.com/)

---

### Summary Table

| **API**               | **Free Tier Limits**                    | **Best For**                         |
|------------------------|------------------------------------------|---------------------------------------|
| Alpha Vantage          | 5 calls/min, 500 calls/day              | Technical and fundamental analysis    |
| yfinance               | Unlimited (with rate-limiting)          | Historical and basic stock data       |
| Financial Modeling Prep| Varies by endpoint                     | Company fundamentals, stock quotes    |
| Quandl                | Limited free datasets                   | Historical and alternative datasets   |
| IEX Cloud              | 10k messages/month                     | Real-time stock and financial news    |
| Twelve Data            | 8 calls/min                            | Stock, forex, and technical analysis  |
| Polygon.io             | Limited tickers/data                   | Real-time stock and forex data        |
| OpenFIGI              | Free with registration                  | Instrument mapping and identifiers    |

Each API has unique strengths, so the best choice depends on your specific requirements!
