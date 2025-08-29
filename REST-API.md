# 🌐 REST API Documentation

Base URL: `https://launch-api.paxeer.app/api/v1`

## 📊 Pool Endpoints

### GET /pools
Get all active pools with current data
  
**Response:**
```json
{
  "status": "success",
  "data": {
    "pools": [
      {
        "id": 1,
        "pool_address": "0xDF6a7e620C94bf13EDDefEA0149fd5A9eA3b74c9",
        "project_token": "0x1bf5d6530cC013835eB481A4dDD951665461d485",
        "token_name": "MyProjectToken",
        "token_symbol": "MPT",
        "current_price": 0.08174520,
        "market_cap": 81745200000,
        "total_volume_usdc": 50100,
        "total_trades": 6,
        "price_change_24h": 0.15,
        "ath_price": 0.08174520,
        "atl_price": 0.00085333,
        "created_at": "2025-08-29T08:03:42.000Z"
      }
    ]
  }
}
```

### GET /pools/:address
Get specific pool data

**Parameters:**
- `address` - Pool contract address

**Response:** Single pool object

### GET /pools/:address/chart
Get price chart data for specific timeframes

**Parameters:**
- `address` - Pool contract address
- `timeframe` - Query parameter: `1h`, `24h`, `7d`, `30d`

**Example:** `/pools/0xDF6a7e620C94bf13EDDefEA0149fd5A9eA3b74c9/chart?timeframe=24h`

**Response:**
```json
{
  "status": "success",
  "data": {
    "chart": [
      {
        "timestamp": "2025-08-29T08:00:00.000Z",
        "price": 0.00085333,
        "volume": 10000,
        "block_number": 15698
      },
      {
        "timestamp": "2025-08-29T09:00:00.000Z",
        "price": 0.08174520,
        "volume": 40100,
        "block_number": 15785
      }
    ],
    "timeframe": "24h",
    "total_points": 2
  }
}
```

## 📈 Market Endpoints

### GET /market/overview
Get overall market statistics

**Response:**
```json
{
  "status": "success",
  "data": {
    "overview": {
      "total_pools": 1,
      "total_volume_usdc": 50100,
      "total_traders": 1,
      "total_transactions": 6,
      "top_pool_by_volume": "0xDF6a7e620C94bf13EDDefEA0149fd5A9eA3b74c9",
      "updated_at": "2025-08-29T09:28:51.000Z"
    }
  }
}
```

## 🔧 Utility Endpoints

### POST /force-stats-update
Force recalculation of all pool statistics (admin endpoint)

**Response:**
```json
{
  "status": "success",
  "message": "Statistics update initiated"
}
```

### GET /health
Health check endpoint

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2025-08-29T09:28:51.000Z",
  "services": {
    "database": "connected",
    "websocket": "running"
  }
}
```

## 🛠️ Integration Examples

### React Hook Example
```javascript
import { useState, useEffect } from 'react';

const usePools = () => {
  const [pools, setPools] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchPools = async () => {
      try {
        const response = await fetch('https://launch-api.paxeer.app/api/v1/pools');
        const data = await response.json();
        setPools(data.data.pools);
      } catch (error) {
        console.error('Failed to fetch pools:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchPools();
  }, []);

  return { pools, loading };
};
```

### Chart Data Fetching
```javascript
const fetchChartData = async (poolAddress, timeframe = '24h') => {
  const response = await fetch(
    `https://launch-api.paxeer.app/api/v1/pools/${poolAddress}/chart?timeframe=${timeframe}`
  );
  const data = await response.json();
  return data.data.chart;
};
```

## 📝 Error Handling

All endpoints return consistent error responses:

```json
{
  "status": "error",
  "message": "Pool not found",
  "code": "POOL_NOT_FOUND"
}
```

**Common Error Codes:**
- `POOL_NOT_FOUND` - Pool address doesn't exist
- `INVALID_TIMEFRAME` - Invalid chart timeframe
- `DATABASE_ERROR` - Internal database error
- `INVALID_ADDRESS` - Malformed contract address
