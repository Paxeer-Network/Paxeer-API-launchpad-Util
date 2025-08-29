# 🔌 WebSocket API Documentation

WebSocket URL: `wss://launch-api.paxeer.app/ws`

## 📡 Real-time Data Streaming

### Connection Setup

```javascript
const ws = new WebSocket('wss://launch-api.paxeer.app');

ws.onopen = () => {
  console.log('🔌 Connected to Paxeer WebSocket');
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  handleRealtimeUpdate(data);
};

ws.onclose = () => {
  console.log('🔌 WebSocket connection closed');
  // Implement reconnection logic
};
```

## 📊 Event Types

### 1. Price Updates
Real-time price changes for all pools

**Event Type:** `price_update`

```json
{
  "type": "price_update",
  "data": {
    "pool_address": "0xDF6a7e620C94bf13EDDefEA0149fd5A9eA3b74c9",
    "token_symbol": "MPT",
    "current_price": 0.08174520,
    "price_change": 0.00125,
    "price_change_percent": 1.55,
    "volume_24h": 50100,
    "market_cap": 81745200000,
    "timestamp": "2025-08-29T09:28:51.000Z"
  }
}
```

### 2. New Trades
Live trade notifications

**Event Type:** `new_trade`

```json
{
  "type": "new_trade",
  "data": {
    "pool_address": "0xDF6a7e620C94bf13EDDefEA0149fd5A9eA3b74c9",
    "trader": "0x2fccd991Ecc9bEe62Bd10d751A5c5492e2a788C7",
    "token_symbol": "MPT",
    "usdc_amount": 10000,
    "token_amount": 14024222.63733921,
    "price": 0.08174520,
    "transaction_hash": "0xe3b21c3722848f5cc4d6fbff3014569b23ebca0fc64738fa5f49406bb0eddebb",
    "block_number": 15785,
    "timestamp": "2025-08-29T09:20:13.000Z"
  }
}
```

### 3. Pool Status
Pool lifecycle events

**Event Type:** `pool_status`

```json
{
  "type": "pool_status",
  "data": {
    "pool_address": "0xDF6a7e620C94bf13EDDefEA0149fd5A9eA3b74c9",
    "status": "active", // "created", "active", "trading_complete"
    "token_symbol": "MPT",
    "message": "Pool is actively trading",
    "timestamp": "2025-08-29T08:03:42.000Z"
  }
}
```

## 🛠️ React Implementation

### Custom Hook for WebSocket
```javascript
import { useState, useEffect, useRef } from 'react';

const useWebSocket = (url) => {
  const [data, setData] = useState(null);
  const [connected, setConnected] = useState(false);
  const ws = useRef(null);

  useEffect(() => {
    const connectWebSocket = () => {
      ws.current = new WebSocket(url);
      
      ws.current.onopen = () => {
        setConnected(true);
        console.log('🔌 WebSocket connected');
      };
      
      ws.current.onmessage = (event) => {
        const message = JSON.parse(event.data);
        setData(message);
      };
      
      ws.current.onclose = () => {
        setConnected(false);
        console.log('🔌 WebSocket disconnected');
        // Reconnect after 3 seconds
        setTimeout(connectWebSocket, 3000);
      };
      
      ws.current.onerror = (error) => {
        console.error('🔌 WebSocket error:', error);
      };
    };

    connectWebSocket();

    return () => {
      if (ws.current) {
        ws.current.close();
      }
    };
  }, [url]);

  return { data, connected };
};

export default useWebSocket;
```

### Price Updates Component
```javascript
import React, { useState, useEffect } from 'react';
import useWebSocket from './hooks/useWebSocket';

const PriceTracker = ({ poolAddress }) => {
  const [prices, setPrices] = useState({});
  const { data, connected } = useWebSocket('wss://launch-api.paxeer.app');

  useEffect(() => {
    if (data && data.type === 'price_update') {
      const { pool_address, current_price, token_symbol } = data.data;
      setPrices(prev => ({
        ...prev,
        [pool_address]: {
          price: current_price,
          symbol: token_symbol,
          lastUpdated: new Date()
        }
      }));
    }
  }, [data]);

  const poolPrice = prices[poolAddress];

  return (
    <div className="price-tracker">
      <div className={`connection-status ${connected ? 'connected' : 'disconnected'}`}>
        {connected ? '🟢 Live' : '🔴 Disconnected'}
      </div>
      
      {poolPrice && (
        <div className="price-display">
          <span className="symbol">{poolPrice.symbol}</span>
          <span className="price">${poolPrice.price.toFixed(8)}</span>
          <span className="updated">
            Updated: {poolPrice.lastUpdated.toLocaleTimeString()}
          </span>
        </div>
      )}
    </div>
  );
};

export default PriceTracker;
```

### Trade Feed Component
```javascript
import React, { useState, useEffect } from 'react';
import useWebSocket from './hooks/useWebSocket';

const TradeFeed = () => {
  const [trades, setTrades] = useState([]);
  const { data } = useWebSocket('wss://launch-api.paxeer.app');

  useEffect(() => {
    if (data && data.type === 'new_trade') {
      setTrades(prev => [data.data, ...prev.slice(0, 49)]); // Keep last 50 trades
    }
  }, [data]);

  return (
    <div className="trade-feed">
      <h3>🔥 Live Trades</h3>
      <div className="trades-list">
        {trades.map((trade, index) => (
          <div key={index} className="trade-item">
            <div className="trade-token">{trade.token_symbol}</div>
            <div className="trade-amount">
              ${trade.usdc_amount.toLocaleString()}
            </div>
            <div className="trade-price">
              ${trade.price.toFixed(8)}
            </div>
            <div className="trade-time">
              {new Date(trade.timestamp).toLocaleTimeString()}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};

export default TradeFeed;
```

## 🔄 Connection Management

### Automatic Reconnection
```javascript
class WebSocketManager {
  constructor(url) {
    this.url = url;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 10;
    this.listeners = new Map();
  }

  connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      console.log('🔌 Connected to WebSocket');
      this.reconnectAttempts = 0;
    };
    
    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.notifyListeners(data);
    };
    
    this.ws.onclose = () => {
      console.log('🔌 WebSocket closed');
      this.attemptReconnect();
    };
  }

  attemptReconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);
      console.log(`🔄 Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);
      setTimeout(() => this.connect(), delay);
    }
  }

  subscribe(eventType, callback) {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, []);
    }
    this.listeners.get(eventType).push(callback);
  }

  notifyListeners(data) {
    const callbacks = this.listeners.get(data.type);
    if (callbacks) {
      callbacks.forEach(callback => callback(data));
    }
  }
}

// Usage
const wsManager = new WebSocketManager('wss://launch-api.paxeer.app');
wsManager.connect();

wsManager.subscribe('price_update', (data) => {
  console.log('Price update:', data);
});

wsManager.subscribe('new_trade', (data) => {
  console.log('New trade:', data);
});
```
