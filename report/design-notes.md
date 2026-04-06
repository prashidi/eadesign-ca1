# Enterprise Architecture Design - Continous Assessment 1

## System Overview

This project implements a nanoservices-based checkout system deployed on Kubernetes (K3s).

The system consists of the following services:

- Gateway (public entrypoint)
- Checkout service (composition logic)
- Pricing service
- Inventory service
- PostgreSQL database (persistence)

The system supports:

- POST /api/checkout
- GET /api/arch
- GET /api/ping
- UI served at /

The architecture follows a nanoservices model where each service has a single responsibility.

## Service Responsibilities

### Gateway
- Serves UI at /
- Handles /api/arch and /api/ping
- Accepts POST /api/checkout
- Forwards requests to checkout service
- Generates or forwards X-Request-Id

### Checkout Service
- Validates request input
- Calls pricing and inventory services
- Applies timeout to dependencies
- Combines responses
- Stores results in PostgreSQL
- Logs request ID

### Pricing Service
- Calculates tax and total
- Provides /health endpoint

### Inventory Service
- Checks stock availability
- Provides /health endpoint

### PostgreSQL
- Stores checkout records
- Uses Secret for credentials
- Uses PVC for persistence

## API Design

### Public Endpoints (Gateway)

GET /api/arch
Response:
{
  "arch": "kubernetes-nanoservices-checkout"
}

GET /api/ping
Response:
{
  "ok": true
}

POST /api/checkout
Request:
{
  "sku": 1,
  "subtotal": 100
}


### Internal Endpoints

Checkout Service:
- POST /checkout
- GET /health

Pricing Service:
- POST /price
- GET /health

Inventory Service:
- GET /stock/:sku
- GET /health

## Request Flow

1. Client sends POST /api/checkout to Gateway
2. Gateway generates or forwards X-Request-Id
3. Gateway forwards request to Checkout Service
4. Checkout calls Pricing Service
5. Checkout calls Inventory Service
6. Checkout combines responses
7. Checkout writes data to PostgreSQL
8. Response is returned to Gateway
9. Gateway returns response to client

## Failure Handling

If a dependency (pricing or inventory) is unavailable:

- Checkout service applies a timeout (1500ms)
- Checkout returns HTTP 503
- Gateway remains available

Example response:
{
  "error": "dependency timeout/unavailable"
}

## Request Correlation

The system supports X-Request-Id header.

- Gateway generates ID if not provided
- ID is forwarded to all services
- All services log the request ID

This allows tracing a request across multiple services.

## Scaling Strategy

The Checkout Service will use KEDA for scale-to-zero.

Reason:
- It is user-triggered and bursty
- Allows measurement of cold vs warm latency

Other services remain always running.

## Persistence Design

PostgreSQL will store checkout records.

Each record includes:
- request_id
- sku
- subtotal
- total
- stock status
- timestamp

Persistence is ensured using:
- Kubernetes Secret (for password)
- PersistentVolumeClaim (PVC)

## Partial Failure Scenario

If inventory service is unavailable:

- Checkout fails with HTTP 503
- Gateway continues to serve:
  - /
  - /api/ping
  - /api/arch

This demonstrates partial system failure.


