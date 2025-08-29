# 💸 Transaction Examples & Implementation

## 🔧 Setup & Configuration

### MetaMask Network Configuration
```javascript
const addPaxeerNetwork = async () => {
  try {
    await window.ethereum.request({
      method: 'wallet_addEthereumChain',
      params: [{
        chainId: '0x13880', // 80000 in hex
        chainName: 'Paxeer Network',
        nativeCurrency: {
          name: 'Paxeer',
          symbol: 'PAX',
          decimals: 18
        },
        rpcUrls: ['https://rpc-paxeer-network-djjz47ii4b.t.conduit.xyz'],
        blockExplorerUrls: ['https://paxscan.paxeer.app']
      }]
    });
  } catch (error) {
    console.error('Failed to add network:', error);
  }
};
```

### USDC Contract Address
```javascript
const USDC_ADDRESS = "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85"; // Paxeer Network USDC
```

## 🛒 Common Transaction Patterns

### 1. Buy Tokens (USDC → Project Token)

```javascript
import { ethers } from 'ethers';

const buyTokens = async (poolAddress, usdcAmount, slippageTolerance = 5) => {
  try {
    const signer = await getSigner();
    const routerContract = new ethers.Contract(CONTRACTS.ROUTER, ROUTER_ABI, signer);
    const usdcContract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, signer);
    
    // Convert USDC amount (6 decimals)
    const usdcAmountWei = ethers.utils.parseUnits(usdcAmount.toString(), 6);
    
    // Step 1: Get quote for tokens out
    const tokensOut = await routerContract.getAmountOut(
      poolAddress,
      usdcAmountWei,
      true // isUSDCIn = true
    );
    
    // Step 2: Calculate minimum tokens with slippage
    const slippageMultiplier = (100 - slippageTolerance) / 100;
    const minTokensOut = tokensOut.mul(
      ethers.BigNumber.from(Math.floor(slippageMultiplier * 100))
    ).div(100);
    
    // Step 3: Check and approve USDC allowance
    const currentAllowance = await usdcContract.allowance(
      await signer.getAddress(),
      CONTRACTS.ROUTER
    );
    
    if (currentAllowance.lt(usdcAmountWei)) {
      console.log('💰 Approving USDC...');
      const approveTx = await usdcContract.approve(
        CONTRACTS.ROUTER,
        usdcAmountWei
      );
      await approveTx.wait();
      console.log('✅ USDC approved');
    }
    
    // Step 4: Execute swap
    console.log('🔄 Executing swap...');
    const swapTx = await routerContract.swapUSDCForTokens(
      poolAddress,
      usdcAmountWei,
      minTokensOut
    );
    
    console.log('📋 Transaction hash:', swapTx.hash);
    const receipt = await swapTx.wait();
    console.log('✅ Swap completed!');
    
    return {
      success: true,
      txHash: swapTx.hash,
      receipt: receipt,
      tokensReceived: tokensOut
    };
    
  } catch (error) {
    console.error('❌ Swap failed:', error);
    return {
      success: false,
      error: error.message
    };
  }
};

// Usage
const result = await buyTokens('0xDF6a7e620C94bf13EDDefEA0149fd5A9eA3b74c9', 100); // Buy with 100 USDC
```

### 2. Sell Tokens (Project Token → USDC)

```javascript
const sellTokens = async (poolAddress, tokenAmount, slippageTolerance = 5) => {
  try {
    const signer = await getSigner();
    const routerContract = new ethers.Contract(CONTRACTS.ROUTER, ROUTER_ABI, signer);
    const poolContract = new ethers.Contract(poolAddress, POOL_ABI, signer);
    
    // Get project token address
    const projectTokenAddress = await poolContract.projectToken();
    const tokenContract = new ethers.Contract(projectTokenAddress, ERC20_ABI, signer);
    
    // Convert token amount (18 decimals)
    const tokenAmountWei = ethers.utils.parseEther(tokenAmount.toString());
    
    // Step 1: Get quote for USDC out
    const usdcOut = await routerContract.getAmountOut(
      poolAddress,
      tokenAmountWei,
      false // isUSDCIn = false
    );
    
    // Step 2: Calculate minimum USDC with slippage
    const slippageMultiplier = (100 - slippageTolerance) / 100;
    const minUSDCOut = usdcOut.mul(
      ethers.BigNumber.from(Math.floor(slippageMultiplier * 100))
    ).div(100);
    
    // Step 3: Check and approve token allowance
    const currentAllowance = await tokenContract.allowance(
      await signer.getAddress(),
      CONTRACTS.ROUTER
    );
    
    if (currentAllowance.lt(tokenAmountWei)) {
      console.log('🪙 Approving tokens...');
      const approveTx = await tokenContract.approve(
        CONTRACTS.ROUTER,
        tokenAmountWei
      );
      await approveTx.wait();
      console.log('✅ Tokens approved');
    }
    
    // Step 4: Execute swap
    console.log('🔄 Executing sell...');
    const swapTx = await routerContract.swapTokensForUSDC(
      poolAddress,
      tokenAmountWei,
      minUSDCOut
    );
    
    console.log('📋 Transaction hash:', swapTx.hash);
    const receipt = await swapTx.wait();
    console.log('✅ Sell completed!');
    
    return {
      success: true,
      txHash: swapTx.hash,
      receipt: receipt,
      usdcReceived: usdcOut
    };
    
  } catch (error) {
    console.error('❌ Sell failed:', error);
    return {
      success: false,
      error: error.message
    };
  }
};
```

### 3. Create New Pool

```javascript
const createPool = async (tokenAddress, projectName, projectSymbol) => {
  try {
    const signer = await getSigner();
    const factoryContract = new ethers.Contract(CONTRACTS.FACTORY, FACTORY_ABI, signer);
    
    console.log('🏭 Creating new pool...');
    const createTx = await factoryContract.createPool(
      tokenAddress,
      projectName,
      projectSymbol
    );
    
    console.log('📋 Transaction hash:', createTx.hash);
    const receipt = await createTx.wait();
    
    // Extract pool address from events
    const poolCreatedEvent = receipt.events.find(
      event => event.event === 'PoolCreated'
    );
    const poolAddress = poolCreatedEvent.args.pool;
    
    console.log('✅ Pool created at:', poolAddress);
    
    return {
      success: true,
      poolAddress: poolAddress,
      txHash: createTx.hash,
      receipt: receipt
    };
    
  } catch (error) {
    console.error('❌ Pool creation failed:', error);
    return {
      success: false,
      error: error.message
    };
  }
};
```

## 📊 Data Fetching Functions

### Get Pool Information
```javascript
const getPoolInfo = async (poolAddress) => {
  const poolContract = new ethers.Contract(poolAddress, POOL_ABI, provider);
  
  const [
    reserves,
    projectToken,
    projectName,
    projectSymbol,
    currentPrice
  ] = await Promise.all([
    poolContract.getReserves(),
    poolContract.projectToken(),
    poolContract.projectName(),
    poolContract.projectSymbol(),
    poolContract.getCurrentPrice()
  ]);
  
  return {
    poolAddress,
    projectToken,
    projectName,
    projectSymbol,
    usdcReserve: ethers.utils.formatUnits(reserves.usdcReserve, 6),
    tokenReserve: ethers.utils.formatEther(reserves.tokenReserve),
    currentPrice: ethers.utils.formatUnits(currentPrice, 18)
  };
};
```

### Get User Balances
```javascript
const getUserBalances = async (userAddress, poolAddress) => {
  const poolContract = new ethers.Contract(poolAddress, POOL_ABI, provider);
  const projectTokenAddress = await poolContract.projectToken();
  
  const usdcContract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, provider);
  const tokenContract = new ethers.Contract(projectTokenAddress, ERC20_ABI, provider);
  
  const [usdcBalance, tokenBalance] = await Promise.all([
    usdcContract.balanceOf(userAddress),
    tokenContract.balanceOf(userAddress)
  ]);
  
  return {
    usdc: ethers.utils.formatUnits(usdcBalance, 6),
    projectToken: ethers.utils.formatEther(tokenBalance)
  };
};
```

### Calculate Swap Quote
```javascript
const getSwapQuote = async (poolAddress, amountIn, isUSDCIn) => {
  const routerContract = new ethers.Contract(CONTRACTS.ROUTER, ROUTER_ABI, provider);
  
  const amountInWei = isUSDCIn 
    ? ethers.utils.parseUnits(amountIn.toString(), 6)
    : ethers.utils.parseEther(amountIn.toString());
  
  const amountOut = await routerContract.getAmountOut(
    poolAddress,
    amountInWei,
    isUSDCIn
  );
  
  const formattedOut = isUSDCIn
    ? ethers.utils.formatEther(amountOut)
    : ethers.utils.formatUnits(amountOut, 6);
  
  return {
    amountIn: amountIn,
    amountOut: formattedOut,
    priceImpact: calculatePriceImpact(poolAddress, amountInWei, isUSDCIn)
  };
};
```

## 🎯 React Hook Examples

### Swap Hook
```javascript
import { useState, useCallback } from 'react';

const useSwap = () => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const executeBuy = useCallback(async (poolAddress, usdcAmount, slippage) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await buyTokens(poolAddress, usdcAmount, slippage);
      if (!result.success) {
        throw new Error(result.error);
      }
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);

  const executeSell = useCallback(async (poolAddress, tokenAmount, slippage) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await sellTokens(poolAddress, tokenAmount, slippage);
      if (!result.success) {
        throw new Error(result.error);
      }
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);

  return {
    executeBuy,
    executeSell,
    loading,
    error
  };
};
```

### Pool Data Hook
```javascript
import { useState, useEffect } from 'react';

const usePoolData = (poolAddress) => {
  const [poolData, setPoolData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchPoolData = async () => {
      try {
        const data = await getPoolInfo(poolAddress);
        setPoolData(data);
      } catch (error) {
        console.error('Failed to fetch pool data:', error);
      } finally {
        setLoading(false);
      }
    };

    if (poolAddress) {
      fetchPoolData();
    }
  }, [poolAddress]);

  return { poolData, loading };
};
```

## 🚨 Error Handling

### Common Errors & Solutions
```javascript
const handleTransactionError = (error) => {
  if (error.code === 4001) {
    return "Transaction rejected by user";
  } else if (error.code === -32603) {
    return "Insufficient balance or allowance";
  } else if (error.message.includes("slippage")) {
    return "Price moved too much, try increasing slippage tolerance";
  } else if (error.message.includes("INSUFFICIENT_LIQUIDITY")) {
    return "Not enough liquidity in the pool";
  } else {
    return `Transaction failed: ${error.message}`;
  }
};
```
