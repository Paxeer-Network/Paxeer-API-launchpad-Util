# ⚛️ Complete React Integration Guide

## 🏗️ Project Setup

### Install Dependencies
```bash
npm install ethers @web3-react/core @web3-react/injected-connector
npm install recharts axios socket.io-client
npm install @types/node # if using TypeScript
```

### Environment Variables (.env)
```env
REACT_APP_API_BASE_URL=https://launch-api.paxeer.app/api/v1
REACT_APP_WS_URL=wss://launch-api.paxeer.app
REACT_APP_FACTORY_ADDRESS=0xDE63a33db52EfAf44f28e6d99406874Cffe5820C
REACT_APP_ROUTER_ADDRESS=0xFAc155Aa5bdcAA5F7931fF581C7fF01eeB00C667
REACT_APP_USDC_ADDRESS=0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85
REACT_APP_CHAIN_ID=80000
```

## 🔗 Web3 Context Provider

### contexts/Web3Context.js
```javascript
import React, { createContext, useContext, useState, useEffect } from 'react';
import { ethers } from 'ethers';

const Web3Context = createContext();

export const useWeb3 = () => {
  const context = useContext(Web3Context);
  if (!context) {
    throw new Error('useWeb3 must be used within Web3Provider');
  }
  return context;
};

export const Web3Provider = ({ children }) => {
  const [provider, setProvider] = useState(null);
  const [signer, setSigner] = useState(null);
  const [account, setAccount] = useState(null);
  const [chainId, setChainId] = useState(null);
  const [connected, setConnected] = useState(false);

  const connectWallet = async () => {
    if (!window.ethereum) {
      throw new Error('MetaMask not detected');
    }

    try {
      const web3Provider = new ethers.providers.Web3Provider(window.ethereum);
      await web3Provider.send('eth_requestAccounts', []);
      
      const signer = web3Provider.getSigner();
      const account = await signer.getAddress();
      const network = await web3Provider.getNetwork();

      // Check if on correct network
      if (network.chainId !== parseInt(process.env.REACT_APP_CHAIN_ID)) {
        await switchNetwork();
      }

      setProvider(web3Provider);
      setSigner(signer);
      setAccount(account);
      setChainId(network.chainId);
      setConnected(true);

      localStorage.setItem('walletConnected', 'true');
    } catch (error) {
      console.error('Connection failed:', error);
      throw error;
    }
  };

  const switchNetwork = async () => {
    try {
      await window.ethereum.request({
        method: 'wallet_switchEthereumChain',
        params: [{ chainId: `0x${parseInt(process.env.REACT_APP_CHAIN_ID).toString(16)}` }]
      });
    } catch (error) {
      if (error.code === 4902) {
        // Network doesn't exist, add it
        await window.ethereum.request({
          method: 'wallet_addEthereumChain',
          params: [{
            chainId: `0x${parseInt(process.env.REACT_APP_CHAIN_ID).toString(16)}`,
            chainName: 'Paxeer Network',
            nativeCurrency: {
              name: 'USDC',
              symbol: 'USDC',
              decimals: 6
            },
            rpcUrls: ['https://rpc-paxeer-network-djjz47ii4b.t.conduit.xyz'],
            blockExplorerUrls: ['https://paxscan.paxeer.app']
          }]
        });
      }
    }
  };

  const disconnectWallet = () => {
    setProvider(null);
    setSigner(null);
    setAccount(null);
    setChainId(null);
    setConnected(false);
    localStorage.removeItem('walletConnected');
  };

  // Auto-connect on page load
  useEffect(() => {
    if (localStorage.getItem('walletConnected') === 'true') {
      connectWallet().catch(console.error);
    }
  }, []);

  // Listen for account changes
  useEffect(() => {
    if (window.ethereum) {
      window.ethereum.on('accountsChanged', (accounts) => {
        if (accounts.length === 0) {
          disconnectWallet();
        } else {
          connectWallet();
        }
      });

      window.ethereum.on('chainChanged', () => {
        window.location.reload();
      });
    }
  }, []);

  const value = {
    provider,
    signer,
    account,
    chainId,
    connected,
    connectWallet,
    disconnectWallet,
    switchNetwork
  };

  return <Web3Context.Provider value={value}>{children}</Web3Context.Provider>;
};
```

## 🔌 WebSocket Hook

### hooks/useWebSocket.js
```javascript
import { useState, useEffect, useRef, useCallback } from 'react';

const useWebSocket = (url) => {
  const [data, setData] = useState(null);
  const [connected, setConnected] = useState(false);
  const [error, setError] = useState(null);
  const ws = useRef(null);
  const reconnectAttempts = useRef(0);
  const maxReconnectAttempts = 10;

  const connect = useCallback(() => {
    try {
      ws.current = new WebSocket(url);

      ws.current.onopen = () => {
        setConnected(true);
        setError(null);
        reconnectAttempts.current = 0;
        console.log('🔌 WebSocket connected');
      };

      ws.current.onmessage = (event) => {
        try {
          const message = JSON.parse(event.data);
          setData(message);
        } catch (err) {
          console.error('Failed to parse WebSocket message:', err);
        }
      };

      ws.current.onclose = () => {
        setConnected(false);
        console.log('🔌 WebSocket disconnected');
        
        // Attempt reconnection
        if (reconnectAttempts.current < maxReconnectAttempts) {
          reconnectAttempts.current += 1;
          const delay = Math.min(1000 * Math.pow(2, reconnectAttempts.current), 30000);
          console.log(`🔄 Reconnecting in ${delay}ms (attempt ${reconnectAttempts.current})`);
          setTimeout(connect, delay);
        }
      };

      ws.current.onerror = (error) => {
        setError(error);
        console.error('🔌 WebSocket error:', error);
      };
    } catch (err) {
      setError(err);
      console.error('Failed to connect WebSocket:', err);
    }
  }, [url]);

  useEffect(() => {
    connect();

    return () => {
      if (ws.current) {
        ws.current.close();
      }
    };
  }, [connect]);

  const sendMessage = useCallback((message) => {
    if (ws.current && ws.current.readyState === WebSocket.OPEN) {
      ws.current.send(JSON.stringify(message));
    }
  }, []);

  return { data, connected, error, sendMessage };
};

export default useWebSocket;
```

## 📊 Pool Data Hooks

### hooks/usePools.js
```javascript
import { useState, useEffect } from 'react';
import axios from 'axios';

const API_BASE = process.env.REACT_APP_API_BASE_URL;

export const usePools = () => {
  const [pools, setPools] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchPools = async () => {
    try {
      setLoading(true);
      const response = await axios.get(`${API_BASE}/pools`);
      setPools(response.data.data.pools);
      setError(null);
    } catch (err) {
      setError(err.response?.data?.message || err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchPools();
    const interval = setInterval(fetchPools, 30000); // Refresh every 30s
    return () => clearInterval(interval);
  }, []);

  return { pools, loading, error, refetch: fetchPools };
};

export const usePool = (poolAddress) => {
  const [pool, setPool] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!poolAddress) return;

    const fetchPool = async () => {
      try {
        setLoading(true);
        const response = await axios.get(`${API_BASE}/pools/${poolAddress}`);
        setPool(response.data.data.pool);
        setError(null);
      } catch (err) {
        setError(err.response?.data?.message || err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchPool();
  }, [poolAddress]);

  return { pool, loading, error };
};
```

## 💱 Trading Component

### components/TradingInterface.jsx
```javascript
import React, { useState, useEffect } from 'react';
import { ethers } from 'ethers';
import { useWeb3 } from '../contexts/Web3Context';
import { ROUTER_ABI, ERC20_ABI } from '../contracts/abis';

const TradingInterface = ({ pool }) => {
  const { signer, account } = useWeb3();
  const [buyAmount, setBuyAmount] = useState('');
  const [sellAmount, setSellAmount] = useState('');
  const [loading, setLoading] = useState(false);
  const [balances, setBalances] = useState({ usdc: '0', token: '0' });
  const [quote, setQuote] = useState(null);

  const ROUTER_ADDRESS = process.env.REACT_APP_ROUTER_ADDRESS;
  const USDC_ADDRESS = process.env.REACT_APP_USDC_ADDRESS;

  // Fetch user balances
  useEffect(() => {
    if (!account || !pool) return;

    const fetchBalances = async () => {
      try {
        const usdcContract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, signer);
        const tokenContract = new ethers.Contract(pool.project_token, ERC20_ABI, signer);

        const [usdcBalance, tokenBalance] = await Promise.all([
          usdcContract.balanceOf(account),
          tokenContract.balanceOf(account)
        ]);

        setBalances({
          usdc: ethers.utils.formatUnits(usdcBalance, 6),
          token: ethers.utils.formatEther(tokenBalance)
        });
      } catch (error) {
        console.error('Failed to fetch balances:', error);
      }
    };

    fetchBalances();
  }, [account, pool, signer]);

  // Get buy quote
  useEffect(() => {
    if (!buyAmount || !pool) return;

    const getQuote = async () => {
      try {
        const routerContract = new ethers.Contract(ROUTER_ADDRESS, ROUTER_ABI, signer);
        const amountInWei = ethers.utils.parseUnits(buyAmount, 6);
        const tokensOut = await routerContract.getAmountOut(
          pool.pool_address,
          amountInWei,
          true
        );
        setQuote(ethers.utils.formatEther(tokensOut));
      } catch (error) {
        console.error('Quote failed:', error);
        setQuote(null);
      }
    };

    const timeoutId = setTimeout(getQuote, 500); // Debounce
    return () => clearTimeout(timeoutId);
  }, [buyAmount, pool, signer]);

  const executeBuy = async () => {
    if (!buyAmount || !signer) return;

    setLoading(true);
    try {
      const routerContract = new ethers.Contract(ROUTER_ADDRESS, ROUTER_ABI, signer);
      const usdcContract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, signer);
      
      const amountInWei = ethers.utils.parseUnits(buyAmount, 6);
      
      // Check allowance
      const allowance = await usdcContract.allowance(account, ROUTER_ADDRESS);
      if (allowance.lt(amountInWei)) {
        const approveTx = await usdcContract.approve(ROUTER_ADDRESS, amountInWei);
        await approveTx.wait();
      }

      // Calculate minimum tokens out (5% slippage)
      const tokensOut = await routerContract.getAmountOut(
        pool.pool_address,
        amountInWei,
        true
      );
      const minTokensOut = tokensOut.mul(95).div(100);

      // Execute swap
      const swapTx = await routerContract.swapUSDCForTokens(
        pool.pool_address,
        amountInWei,
        minTokensOut
      );

      await swapTx.wait();
      setBuyAmount('');
      alert('Buy successful!');
      
    } catch (error) {
      console.error('Buy failed:', error);
      alert(`Buy failed: ${error.message}`);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="trading-interface">
      <div className="trading-card">
        <h3>💰 Buy {pool?.token_symbol}</h3>
        <div className="input-group">
          <input
            type="number"
            placeholder="USDC amount"
            value={buyAmount}
            onChange={(e) => setBuyAmount(e.target.value)}
            disabled={loading}
          />
          <div className="balance">Balance: {parseFloat(balances.usdc).toFixed(2)} USDC</div>
        </div>
        
        {quote && (
          <div className="quote">
            You'll receive: ~{parseFloat(quote).toFixed(2)} {pool?.token_symbol}
          </div>
        )}
        
        <button 
          onClick={executeBuy} 
          disabled={!buyAmount || loading || !account}
          className="buy-button"
        >
          {loading ? 'Processing...' : `Buy ${pool?.token_symbol}`}
        </button>
      </div>

      <div className="trading-card">
        <h3>💸 Sell {pool?.token_symbol}</h3>
        <div className="input-group">
          <input
            type="number"
            placeholder="Token amount"
            value={sellAmount}
            onChange={(e) => setSellAmount(e.target.value)}
            disabled={loading}
          />
          <div className="balance">
            Balance: {parseFloat(balances.token).toFixed(2)} {pool?.token_symbol}
          </div>
        </div>
        
        <button 
          onClick={() => {/* Implement sell logic */}} 
          disabled={!sellAmount || loading || !account}
          className="sell-button"
        >
          {loading ? 'Processing...' : 'Sell ' + pool?.token_symbol}
        </button>
      </div>
    </div>
  );
};

export default TradingInterface;
```

## 📈 Price Chart Component

### components/PriceChart.jsx
```javascript
import React, { useState, useEffect } from 'react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
import axios from 'axios';
import useWebSocket from '../hooks/useWebSocket';

const PriceChart = ({ poolAddress, timeframe = '24h' }) => {
  const [chartData, setChartData] = useState([]);
  const [loading, setLoading] = useState(true);
  const { data: wsData } = useWebSocket(process.env.REACT_APP_WS_URL);

  // Fetch historical chart data
  useEffect(() => {
    const fetchChartData = async () => {
      try {
        setLoading(true);
        const response = await axios.get(
          `${process.env.REACT_APP_API_BASE_URL}/pools/${poolAddress}/chart?timeframe=${timeframe}`
        );
        setChartData(response.data.data.chart);
      } catch (error) {
        console.error('Failed to fetch chart data:', error);
      } finally {
        setLoading(false);
      }
    };

    if (poolAddress) {
      fetchChartData();
    }
  }, [poolAddress, timeframe]);

  // Update chart with real-time data
  useEffect(() => {
    if (wsData && wsData.type === 'price_update' && wsData.data.pool_address === poolAddress) {
      const newDataPoint = {
        timestamp: wsData.data.timestamp,
        price: wsData.data.current_price,
        block_number: Date.now() // Use timestamp as block number for real-time data
      };

      setChartData(prevData => {
        const updatedData = [...prevData, newDataPoint];
        // Keep only last 100 points for performance
        return updatedData.slice(-100);
      });
    }
  }, [wsData, poolAddress]);

  const formatXAxisTick = (tickItem) => {
    const date = new Date(tickItem);
    return date.toLocaleTimeString('en-US', { 
      hour: '2-digit', 
      minute: '2-digit' 
    });
  };

  if (loading) {
    return <div className="chart-loading">Loading chart...</div>;
  }

  return (
    <div className="price-chart">
      <div className="chart-header">
        <h3>📈 Price Chart ({timeframe})</h3>
        <div className="timeframe-buttons">
          {['1h', '24h', '7d', '30d'].map(tf => (
            <button
              key={tf}
              onClick={() => setTimeframe(tf)}
              className={timeframe === tf ? 'active' : ''}
            >
              {tf}
            </button>
          ))}
        </div>
      </div>
      
      <ResponsiveContainer width="100%" height={400}>
        <LineChart data={chartData}>
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis 
            dataKey="timestamp" 
            tickFormatter={formatXAxisTick}
            type="category"
          />
          <YAxis 
            dataKey="price"
            domain={['dataMin', 'dataMax']}
            tickFormatter={(value) => `$${value.toFixed(8)}`}
          />
          <Tooltip 
            labelFormatter={(value) => new Date(value).toLocaleString()}
            formatter={(value) => [`$${value.toFixed(8)}`, 'Price']}
          />
          <Line 
            type="monotone" 
            dataKey="price" 
            stroke="#8884d8" 
            strokeWidth={2}
            dot={false}
            activeDot={{ r: 4 }}
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
};

export default PriceChart;
```

## 📱 Main App Integration

### App.js
```javascript
import React from 'react';
import { Web3Provider } from './contexts/Web3Context';
import { usePools } from './hooks/usePools';
import TradingInterface from './components/TradingInterface';
import PriceChart from './components/PriceChart';
import WalletConnection from './components/WalletConnection';
import './App.css';

function PoolCard({ pool }) {
  return (
    <div className="pool-card">
      <div className="pool-header">
        <h2>{pool.token_name} ({pool.token_symbol})</h2>
        <div className="pool-price">
          ${pool.current_price.toFixed(8)}
          <span className={pool.price_change_24h >= 0 ? 'positive' : 'negative'}>
            {pool.price_change_24h >= 0 ? '+' : ''}
            {(pool.price_change_24h * 100).toFixed(2)}%
          </span>
        </div>
      </div>
      
      <div className="pool-stats">
        <div className="stat">
          <label>Market Cap</label>
          <value>${pool.market_cap.toLocaleString()}</value>
        </div>
        <div className="stat">
          <label>24h Volume</label>
          <value>${pool.total_volume_usdc.toLocaleString()}</value>
        </div>
        <div className="stat">
          <label>Trades</label>
          <value>{pool.total_trades}</value>
        </div>
      </div>

      <PriceChart poolAddress={pool.pool_address} />
      <TradingInterface pool={pool} />
    </div>
  );
}

function App() {
  const { pools, loading, error } = usePools();

  if (loading) return <div className="loading">Loading pools...</div>;
  if (error) return <div className="error">Error: {error}</div>;

  return (
    <Web3Provider>
      <div className="App">
        <header className="app-header">
          <h1>🚀 Paxeer Protocol</h1>
          <WalletConnection />
        </header>

        <main className="app-main">
          {pools.length === 0 ? (
            <div className="no-pools">No active pools found</div>
          ) : (
            <div className="pools-grid">
              {pools.map(pool => (
                <PoolCard key={pool.pool_address} pool={pool} />
              ))}
            </div>
          )}
        </main>
      </div>
    </Web3Provider>
  );
}

export default App;
```

## 🎨 Basic Styling (App.css)
```css
.App {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 2px solid #eee;
}

.pools-grid {
  display: grid;
  gap: 30px;
}

.pool-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 24px;
}

.pool-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.pool-price {
  font-size: 24px;
  font-weight: bold;
}

.positive { color: #10b981; }
.negative { color: #ef4444; }

.pool-stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.stat label {
  display: block;
  font-size: 14px;
  color: #6b7280;
  margin-bottom: 4px;
}

.trading-interface {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-top: 20px;
}

.trading-card {
  background: #f9fafb;
  border-radius: 8px;
  padding: 20px;
}

.input-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 16px;
}

.buy-button, .sell-button {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  margin-top: 10px;
}

.buy-button {
  background: #10b981;
  color: white;
}

.sell-button {
  background: #ef4444;
  color: white;
}

.buy-button:disabled,
.sell-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```
