# FlashSale Microservices Platform

Microservices-based flash sale platform built to handle high-concurrency,
limited-stock purchase scenarios — with safe inventory locking, event-driven
communication, and AI-powered fraud detection and demand forecasting.

## 🧩 Services

| Service       | Description |         Repo |
|---|---|---|
| Order Service | Order creation, status tracking, checkout flow | [flash-sale-order-service](https://github.com/dhirajbhambhu/flash-sale-order-service) |
| Inventory Service | Stock management, Redis-backed reservation locking | [flash-sale-inventory-service](https://github.com/dhirajbhambhu/flash-sale-inventory-service) |
| Payment Service | Mock payment processing | [flash-sale-payment-service](https://github.com/dhirajbhambhu/flash-sale-payment-service) |
| Notification Service | Order confirmations via Kafka events | [flash-sale-notification-service](https://github.com/dhirajbhambhu/flash-sale-notification-service) |
| AI Service | Fraud detection, demand forecasting, RAG chatbot | [flash-sale-ai-service](https://github.com/dhirajbhambhu/flash-sale-ai-service) |

## 🏗️ Architecture

Full system design, key decisions, and data flow are documented in
[ARCHITECTURE.md](./ARCHITECTURE.md).

**High-level flow:**

Order Service ──REST──> Inventory Service ──> Redis (lock + stock cache)
│
│ (publishes OrderPlaced event)
▼
Kafka
│
├──> Payment Service
├──> Notification Service
└──> AI Service (fraud detection, forecasting)

## 🛠️ Tech Stack

`Java` `Spring Boot` `Spring Security` `MySQL` `Redis` `Apache Kafka`
`Python` `FastAPI` `Docker` `React`

## 📌 Key Highlights

- **Redis distributed locking** with atomic `DECR` to safely handle
  concurrent stock reservation under flash-sale load
- **Database-per-service** pattern — each service owns its data independently
- **Kafka event streaming** for asynchronous, multi-consumer communication
  between services
- **AI-driven fraud detection** (Isolation Forest) and **demand forecasting**
  consuming live order events

## 🚀 Status

🔨 In active development — architecture and design finalized, implementation
in progress. See [ARCHITECTURE.md](./ARCHITECTURE.md) for full details and
current build status.

## 📄 Documentation

- [Architecture Documentation](./ARCHITECTURE.md)
- [API Contracts](./API_CONTRACTS.md) *(coming soon)*
