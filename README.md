# 🚀 Paxeer Protocol - Frontend Integration Guide

## Overview
Complete integration guide for frontend developers to build applications using the Paxeer Protocol infrastructure.

## 📋 Quick Reference
- **REST API**: `https://launch-api.paxeer.app/api/v1`
- **WebSocket**: `wss://launch-api.paxeer.app/ws`
- **Network**: Paxeer Network (Chain ID: 80000)
- **Block Explorer**: Custom block explorer integration availableer.app

## 📚 Documentation Sections

### 🔗 API Integration
- [REST API Documentation](./REST-API.md) - Complete endpoint reference
- [WebSocket Documentation](./WebSocket.md) - Real-time data streaming

### 🔒 Blockchain Integration
- [Smart Contract ABIs](./ABIs.md) - Contract interfaces and examples
- [Transaction Examples](./Transactions.md) - Common transaction patterns

### 🎯 Implementation Guides
- [React Integration](./guides/React-Integration.md) - React hooks and components
- [Price Chart Implementation](./guides/Price-Charts.md) - Chart.js integration
- [Wallet Connection](./guides/Wallet-Connection.md) - MetaMask & Web3 setup

## 🏗️ Architecture Overview

```
Frontend App
    ├── REST API (Port 3003)
    │   ├── Pool data & statistics
    │   ├── Market overview
    │   └── Historical charts
    │
    ├── WebSocket (Port 3004)
    │   ├── Real-time price updates
    │   ├── New trade notifications
    │   └── Pool status changes
    │
    └── Smart Contracts
        ├── Factory (Create pools)
        ├── Router (Execute swaps)
        └── Pool (Direct interactions)
```

## 🚦 Getting Started

1. **Setup Web3 Connection**
2. **Connect to REST API**
3. **Establish WebSocket Connection**
4. **Implement Contract Interactions**

See individual documentation files for detailed implementation guides.
