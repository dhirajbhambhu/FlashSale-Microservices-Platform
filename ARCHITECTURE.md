# FlashSale Microservices Platform — Architecture

## 1. Overview

FlashSale is a microservices-based e-commerce platform designed to handle
high-concurrency flash sale scenarios — where thousands of users attempt to
purchase limited-stock items simultaneously. The system demonstrates safe
concurrent inventory management, event-driven communication, and AI-powered
fraud detection and demand forecasting.

## 2. Services

| Service | Responsibility | Tech Stack |
|---|---|---|
| Order Service | Order creation, status tracking, checkout flow | Spring Boot, MySQL |
| Inventory Service | Stock management, reservation, race-condition handling | Spring Boot, MySQL, Redis |
| Payment Service | Mock payment processing | Spring Boot, MySQL |
| Notification Service | Order confirmations and alerts | Spring Boot, Kafka Consumer |
| AI Service | Fraud detection, demand forecasting, RAG chatbot | Python (FastAPI), Kafka Consumer |

## 3. Architecture Diagram

Order Service ──REST──> Inventory Service ──> Redis (lock + stock cache)
│
│ (publishes OrderPlaced event)
▼
Kafka
│
├──> Payment Service
├──> Notification Service
└──> AI Service (fraud detection, forecasting)

## 4. Key Design Decisions

**Decision: Redis distributed lock for stock management**
Why: MySQL row-level locking would bottleneck under flash-sale-level
concurrency (thousands of simultaneous requests). Redis's atomic `DECR`
operation completes in single-digit milliseconds and avoids DB contention.
Trade-off: Redis and DB can drift out of sync — handled via async
reconciliation after the sale window closes.

**Decision: Database-per-service pattern**
Why: Each service owns its data independently, allowing services to scale,
deploy, and evolve without tight coupling. Services reference each other only
by ID, never via foreign keys across databases.

**Decision: Kafka over RabbitMQ for event bus**
Why: OrderPlaced events need to be consumed independently by multiple
services (Payment, Notification, AI). Kafka's consumer groups support this
natively, and event replay allows a temporarily-down consumer to catch up
without losing events.

**Decision: Reject-and-rollback for over-sold stock**
Why: Chosen over a queue/waitlist approach to keep the concurrency model
simple and testable. If a decrement pushes stock below zero, the operation
is rolled back and the order rejected. Failed attempts are logged for future
use in demand forecasting.

## 5. Data Flow — Happy Path

1. User places an order → Order Service receives `POST /api/orders`
2. Order Service calls Inventory Service to reserve stock
3. Inventory Service acquires a Redis lock, checks and decrements stock
4. If stock is available, Order Service marks the order `CONFIRMED`
5. Order Service publishes an `OrderPlaced` event to Kafka
6. Payment Service consumes the event and processes payment
7. Notification Service consumes the event and sends a confirmation
8. AI Service consumes the event for fraud scoring and forecasting input

## 6. API Contracts

See `API_CONTRACTS.md` for full endpoint definitions across all services.

## 7. Status

Architecture finalized as of July 17, 2026. Implementation begins July 19,
2026 with the Spring Boot multi-module project skeleton.
