<div align="center">
  
# 🚀 BYTE Exchange
**High-Performance Order Matching Engine & Real-Time Trading Terminal**

[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-blue.svg?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-22.x-green.svg?style=for-the-badge&logo=node.js)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-4.21-lightgrey.svg?style=for-the-badge&logo=express)](https://expressjs.com/)
[![React](https://img.shields.io/badge/React-18.3-61dafb.svg?style=for-the-badge&logo=react)](https://react.dev/)
[![SQLite](https://img.shields.io/badge/SQLite-WAL_Mode-blue.svg?style=for-the-badge&logo=sqlite)](https://www.sqlite.org/)
[![Vitest](https://img.shields.io/badge/Vitest-3.0-yellow.svg?style=for-the-badge&logo=vitest)](https://vitest.dev/)

An interview-quality, production-grade **Order Matching Engine and Real-Time Trading Terminal** built for a fictional asset called **`BYTE`**.

</div>

---

## 🔗 Live Production Links

- **🌐 Live Trading Terminal**: [https://byte-nu.vercel.app](https://byte-nu.vercel.app)
- **⚙️ Live Backend Service**: [https://byte-exchange-backend.onrender.com](https://byte-exchange-backend.onrender.com)

Designed and implemented following **Clean Architecture principles**, robust Price-Time Priority order matching, persistent dual SQLite/PostgreSQL storage, real-time WebSocket broadcasting, and 50 automated Vitest unit tests.

---

## 🌟 Project Overview & Key Features

### ⚙️ Core Matching Engine & System Capabilities

#### 📊 Order Types & Matching Behavior
- **Limit Orders (`LIMIT`)**:
  - **BUY LIMIT**: Placed with a maximum willingness-to-pay price. Matches against lowest SELL asks $\le$ buy price. Unfilled remaining quantity rests on the Buy Order Book (Bids) sorted descending by price, then ascending by creation time (`createdAt`).
  - **SELL LIMIT**: Placed with a minimum willingness-to-accept price. Matches against highest BUY bids $\ge$ sell price. Unfilled remaining quantity rests on the Sell Order Book (Asks) sorted ascending by price, then ascending by creation time (`createdAt`).
- **Market Orders (`MARKET`)**:
  - **BUY MARKET**: Instantly consumes lowest available SELL asks across multiple price levels until requested quantity is satisfied or liquidity ends. Unfilled portion is cancelled gracefully; market orders **never rest on orderbooks** or remain pending.
  - **SELL MARKET**: Instantly consumes highest available BUY bids across multiple price levels until requested quantity is satisfied or liquidity ends. Unfilled portion is cancelled gracefully; market orders **never rest on orderbooks** or remain pending.
  - **Zero-Liquidity Protection**: If no opposite-side orders exist in the orderbook, the engine cleanly rejects the market order with HTTP 400 (`NO_LIQUIDITY`).

#### ⚡ Trade Execution Rules
- **Price-Time Priority (FIFO)**: High-priority matching based first on price advantage (highest bid / lowest ask), then strictly by arrival timestamp (FIFO) for orders at the same price level.
- **Specification-Compliant Execution Price**: Trade execution price strictly records at the **maker order price** ($95$ in `BUY 100 vs SELL 95`), exactly adhering to ByteVox exchange matching rules.
- **Partial Fills & Multi-Level Depth Sweeps**: Automatically splits orders when liquidity spans across multiple price levels.
- **Order Cancellation**: Instant cancellation of resting open orders from in-memory engine and database (`DELETE /api/orders/:id`), releasing un-filled quantities cleanly.
- **One-Click Engine Reset**: Full state purge feature (`POST /api/orders/reset`) resetting active orderbooks, trade history streams, and exchange statistics back to clean initial state.

### 💾 Persistence & Real-Time Sync
- **Atomic Persistence**: Dual SQLite WAL Mode & PostgreSQL connection pooling. Orderbook state is automatically hydrated from persistent database rows on server startup.
- **Real-Time WebSockets (`ws`)**: Instant event streaming (`ORDER_BOOK_UPDATE`, `TRADE_EXECUTED`, `STATS_UPDATE`) to connected frontend clients.

### 🎨 Trading Dashboard UI
- **Modern Dark Trading Terminal**: Built with React 18 + Vite + TypeScript + TailwindCSS.
- **Visual Liquidity Depth Bars**: Orderbook columns dynamically visualize volume depth ratios per price level.
- **Real-Time Trade Stream**: Live executed trade stream showing Price, Quantity, and Time (`HH:mm:ss`).
- **Order Entry Panel**: Tabbed BUY/SELL selector, LIMIT/MARKET toggle, quick quantity presets (+1, +5, +10, +25, +50), estimated total calculation, and Zod error toast display.

### 🧪 Quality Assurance & Containerization
- **Automated Vitest Unit Tests**: Complete 50-test regression suite covering 20 Limit Order tests and 20 Market Order tests including multi-level sweeps, partial fills, FIFO, price priority, and liquidity protections (**50/50 passed in <1s**).
- **Docker & Docker-Compose**: Production-ready multi-stage Docker builds for Express backend and Nginx-served frontend.

---

## 📁 Repository Directory Structure

```text
BYTE/
├── backend/            # Express.js + TypeScript + SQLite Engine
│   ├── src/
│   │   ├── config/           # Environment & configuration loader
│   │   ├── controllers/      # REST API request handlers
│   │   ├── database/         # SQLite connection & schema
│   │   ├── matching-engine/  # Price-Time Priority Matching Engine
│   │   ├── middlewares/      # Zod validation & error handling
│   │   ├── models/           # SQLite Data Repositories
│   │   ├── routes/           # Express router endpoints
│   │   ├── services/         # Business logic layer
│   │   ├── types/            # TypeScript domain interfaces
│   │   └── utils/            # Logger & helper utilities
│   └── tests/                # Vitest unit test suite
├── frontend/           # React 18 + Vite + TypeScript + TailwindCSS
│   ├── src/
│   │   ├── components/       # UI Components
│   │   ├── hooks/            # Custom Hooks (useWebSocket)
│   │   ├── pages/            # View Layouts
│   │   ├── services/         # API client
│   │   └── types/            # Shared types
├── docker-compose.yml  # Container orchestration
├── design-decisions.md # Technical decisions & analysis
├── architecture.md     # System architecture diagrams
└── README.md           # Master project documentation
```

---

## ⚡ Quick Start & Running Locally

### Prerequisites
- **Node.js**: v18+ (v22 recommended)
- **npm**: v9+

### 1. Setup Dependencies

```bash
git clone https://github.com/Anujdassat/byte.git
cd byte

# Install both backend and frontend dependencies
npm run setup
```

### 2. Running Dev Servers

Launch both the Express Backend and React Frontend dev servers concurrently:

```bash
# Terminal 1: Launch Backend Server (Port 5000)
cd backend
npm run dev

# Terminal 2: Launch Frontend Dashboard (Port 5173)
cd frontend
npm run dev
```

Open **[http://localhost:5173](http://localhost:5173)** in your browser!

---

## 🐳 Running via Docker & Docker-Compose

Run the complete containerized application using Docker:

```bash
docker-compose up --build
```

- **Frontend Dashboard**: `http://localhost:5173`
- **Backend API**: `http://localhost:5000`

---

## 🧪 Running Automated Unit Tests

Execute the Vitest matching engine unit test suite:

```bash
# Run tests inside backend directory
cd backend
npm test
```

---

## 📖 API Documentation

**Base URL:** `http://localhost:5000/api`

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/orders` | `POST` | Submit a New Order (Limit/Market) |
| `/orderbook` | `GET` | Get Order Book Depth |
| `/trades?limit=50` | `GET` | Get Recent Trades |
| `/stats` | `GET` | Get Exchange Statistics |
| `/orders/:id` | `DELETE` | Cancel Order |
| `/orders/reset` | `POST` | Reset Exchange Engine |

<details>
<summary><strong>View Example Requests & Responses</strong></summary>

### 1. Submit New Order (`POST /api/orders`)
- **Limit Order Body**:
```json
{
  "side": "BUY",
  "type": "LIMIT",
  "price": 100.00,
  "quantity": 5
}
```
- **Response (`201 Created`)**:
```json
{
  "success": true,
  "message": "Order created successfully",
  "data": {
    "order": {
      "id": "buy_1785824486782_46ee677e",
      "side": "BUY",
      "type": "LIMIT",
      "price": 100,
      "quantity": 5,
      "remainingQuantity": 5,
      "status": "PENDING"
    },
    "trades": []
  }
}
```

### 2. Get Order Book Depth (`GET /api/orderbook`)
```json
{
  "success": true,
  "data": {
    "bids": [{ "price": 100, "quantity": 7, "orderCount": 1 }],
    "asks": [{ "price": 105, "quantity": 12, "orderCount": 2 }]
  }
}
```
</details>

---

## 🚀 Scaling Strategy & Architectural Analysis

For architectural design deep-dives, sequence diagrams, and throughput scaling analysis (scaling to **100,000 active orders** and **10,000 trades/minute**), refer to:
- 📄 **[Design Decisions](./design-decisions.md)**
- 📐 **[System Architecture](./architecture.md)**