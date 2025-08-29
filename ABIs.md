# 🔒 Smart Contract ABIs & Integration

## 📋 Contract Addresses

```javascript
const CONTRACTS = {
  FACTORY: "0xDE63a33db52EfAf44f28e6d99406874Cffe5820C",
  ROUTER: "0xFAc155Aa5bdcAA5F7931fF581C7fF01eeB00C667",
  POOL_IMPLEMENTATION: "0xa76A418d1584e08Ce60D3650ebb2703F7991B714"
};

const NETWORK_CONFIG = {
  chainId: 80000,
  name: "Paxeer Network",
  rpcUrl: "https://rpc-paxeer-network-djjz47ii4b.t.conduit.xyz",
  nativeCurrency: {
    name: "Paxeer",
    symbol: "PAX",
    decimals: 18
  }
};
```

## 🏭 Factory Contract ABI

**Purpose:** Create new launchpad pools

```javascript
const FACTORY_ABI = [
  {
    "type": "function",
    "name": "createPool",
    "inputs": [
      {
        "name": "_projectToken",
        "type": "address",
        "internalType": "address"
      },
      {
        "name": "_projectName",
        "type": "string",
        "internalType": "string"
      },
      {
        "name": "_projectSymbol", 
        "type": "string",
        "internalType": "string"
      }
    ],
    "outputs": [
      {
        "name": "pool",
        "type": "address",
        "internalType": "address"
      }
    ],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "getPoolByToken",
    "inputs": [
      {
        "name": "token",
        "type": "address",
        "internalType": "address"
      }
    ],
    "outputs": [
      {
        "name": "",
        "type": "address",
        "internalType": "address"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "getAllPools",
    "inputs": [],
    "outputs": [
      {
        "name": "",
        "type": "address[]",
        "internalType": "address[]"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "event",
    "name": "PoolCreated",
    "inputs": [
      {
        "name": "pool",
        "type": "address",
        "indexed": true,
        "internalType": "address"
      },
      {
        "name": "projectToken",
        "type": "address",
        "indexed": true,
        "internalType": "address"
      },
      {
        "name": "creator",
        "type": "address",
        "indexed": true,
        "internalType": "address"
      },
      {
        "name": "projectName",
        "type": "string",
        "indexed": false,
        "internalType": "string"
      }
    ],
    "anonymous": false
  }
];
```

## 🔀 Router Contract ABI

**Purpose:** Execute swaps between USDC and project tokens

```javascript
const ROUTER_ABI = [
  {
    "type": "function",
    "name": "swapUSDCForTokens",
    "inputs": [
      {
        "name": "pool",
        "type": "address",
        "internalType": "address"
      },
      {
        "name": "usdcAmount",
        "type": "uint256",
        "internalType": "uint256"
      },
      {
        "name": "minTokensOut",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "outputs": [
      {
        "name": "tokensOut",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "swapTokensForUSDC",
    "inputs": [
      {
        "name": "pool",
        "type": "address",
        "internalType": "address"
      },
      {
        "name": "tokenAmount",
        "type": "uint256",
        "internalType": "uint256"
      },
      {
        "name": "minUSDCOut",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "outputs": [
      {
        "name": "usdcOut",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "getAmountOut",
    "inputs": [
      {
        "name": "pool",
        "type": "address",
        "internalType": "address"
      },
      {
        "name": "amountIn",
        "type": "uint256",
        "internalType": "uint256"
      },
      {
        "name": "isUSDCIn",
        "type": "bool",
        "internalType": "bool"
      }
    ],
    "outputs": [
      {
        "name": "amountOut",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "event",
    "name": "Swap",
    "inputs": [
      {
        "name": "trader",
        "type": "address",
        "indexed": true,
        "internalType": "address"
      },
      {
        "name": "pool",
        "type": "address",
        "indexed": true,
        "internalType": "address"
      },
      {
        "name": "projectToken",
        "type": "address",
        "indexed": true,
        "internalType": "address"
      },
      {
        "name": "usdcIn",
        "type": "uint256",
        "indexed": false,
        "internalType": "uint256"
      },
      {
        "name": "tokensOut",
        "type": "uint256",
        "indexed": false,
        "internalType": "uint256"
      }
    ],
    "anonymous": false
  }
];
```

## 💧 Pool Contract ABI

**Purpose:** Individual pool interactions and state queries

```javascript
const POOL_ABI = [
  {
    "type": "function",
    "name": "getReserves",
    "inputs": [],
    "outputs": [
      {
        "name": "usdcReserve",
        "type": "uint256",
        "internalType": "uint256"
      },
      {
        "name": "tokenReserve",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "projectToken",
    "inputs": [],
    "outputs": [
      {
        "name": "",
        "type": "address",
        "internalType": "address"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "projectName",
    "inputs": [],
    "outputs": [
      {
        "name": "",
        "type": "string",
        "internalType": "string"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "projectSymbol",
    "inputs": [],
    "outputs": [
      {
        "name": "",
        "type": "string",
        "internalType": "string"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "getCurrentPrice",
    "inputs": [],
    "outputs": [
      {
        "name": "",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "stateMutability": "view"
  }
];
```

## 🪙 ERC20 Token ABI (for USDC & Project Tokens)

```javascript
const ERC20_ABI = [
  {
    "type": "function",
    "name": "balanceOf",
    "inputs": [{"name": "account", "type": "address"}],
    "outputs": [{"name": "", "type": "uint256"}],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "approve",
    "inputs": [
      {"name": "spender", "type": "address"},
      {"name": "amount", "type": "uint256"}
    ],
    "outputs": [{"name": "", "type": "bool"}],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "allowance",
    "inputs": [
      {"name": "owner", "type": "address"},
      {"name": "spender", "type": "address"}
    ],
    "outputs": [{"name": "", "type": "uint256"}],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "transfer",
    "inputs": [
      {"name": "to", "type": "address"},
      {"name": "amount", "type": "uint256"}
    ],
    "outputs": [{"name": "", "type": "bool"}],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "name",
    "inputs": [],
    "outputs": [{"name": "", "type": "string"}],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "symbol", 
    "inputs": [],
    "outputs": [{"name": "", "type": "string"}],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "decimals",
    "inputs": [],
    "outputs": [{"name": "", "type": "uint8"}],
    "stateMutability": "view"
  }
];
```

## 🛠️ Web3 Integration Setup

### Contract Instance Creation
```javascript
import { ethers } from 'ethers';

// Provider setup
const provider = new ethers.providers.WebSocketProvider('wss://rpc-paxeer.tech');

// Contract instances
const factoryContract = new ethers.Contract(
  CONTRACTS.FACTORY,
  FACTORY_ABI,
  provider
);

const routerContract = new ethers.Contract(
  CONTRACTS.ROUTER,
  ROUTER_ABI,
  provider
);

// With signer for transactions
const getSigner = async () => {
  const provider = new ethers.providers.Web3Provider(window.ethereum);
  await provider.send("eth_requestAccounts", []);
  return provider.getSigner();
};

const getRouterWithSigner = async () => {
  const signer = await getSigner();
  return new ethers.Contract(CONTRACTS.ROUTER, ROUTER_ABI, signer);
};
```

### Event Listening
```javascript
// Listen for new pools
factoryContract.on('PoolCreated', (pool, projectToken, creator, projectName) => {
  console.log('🎉 New pool created:', {
    poolAddress: pool,
    tokenAddress: projectToken,
    creator: creator,
    name: projectName
  });
});

// Listen for swaps
routerContract.on('Swap', (trader, pool, projectToken, usdcIn, tokensOut) => {
  console.log('💱 New swap:', {
    trader,
    pool,
    projectToken,
    usdcAmount: ethers.utils.formatUnits(usdcIn, 6),
    tokensOut: ethers.utils.formatUnits(tokensOut, 18)
  });
});
```
