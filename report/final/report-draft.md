# Enterprise Architecture Design - Contineous Assessement 1

## 1. Introduction

This project implements a serverless-style nanoservices architecture using both Docker Compose and Kubernetes (K3s), with a focus on scalability, observability, and fault tolerance. The system models a simplified checkout workflow composed of multiple independent services, including pricing, inventory, checkout, and a gateway, with PostgreSQL used for persistence.

The primary objective of the assignment is to evaluate how nanoservices behave when deployed using traditional container orchestration (Docker Compose) versus Kubernetes, and to extend the Kubernetes deployment with platform-level capabilities such as autoscaling, tracing, and persistence. In particular, the project explores the concept of "serverless-style" execution using KEDA, enabling scale-to-zero behaviour for HTTP workloads.

The architecture demonstrates key distributed system characteristics, including synchronous service composition, partial failure scenarios, and increased latency due to multiple network hops. It also highlights operational concerns such as deployment management, readiness and liveness, service discovery, and troubleshooting using Kubernetes-native tools.

Throughout the implementation, evidence is collected to compare Compose and Kubernetes in terms of setup complexity, operational control, resilience, and networking. Additional experiments include cold-start measurement, request correlation using X-Request-Id, and persistence validation using PostgreSQL with a Persistent Volume Claim (PVC).

This report presents the system design, implementation details, testing results, and operational insights gained from deploying and managing nanoservices in a Kubernetes environment.

## 2. Solution Design

The system is designed as a nanoservices-based architecture, where each service is responsible for a single, well-defined function. The core services include:

- **pricing-fn**: calculates tax and total price based on a given subtotal  
- **inventory-fn**: determines whether a product is in stock  
- **checkout-fn**: composes pricing and inventory responses to complete a transaction  
- **gateway**: serves the user interface and routes HTTP requests to backend services  
- **postgres**: provides persistent storage for checkout records  

The services communicate synchronously over HTTP, forming a request chain where the gateway forwards requests to the checkout service, which in turn calls pricing and inventory services. This design demonstrates service composition and highlights how latency accumulates across multiple network hops.

### Nanoservices Architecture

The use of nanoservices provides clear separation of concerns, making each service simple and focused. This enables easier development, testing, and replacement of individual components. However, it introduces trade-offs such as increased network communication, higher latency, and a greater risk of partial failures when dependencies become unavailable.

### Serverless-Style Execution with KEDA

To simulate serverless behaviour, the checkout service is configured with KEDA (Kubernetes Event-Driven Autoscaler) and the HTTP add-on. This allows the service to scale down to zero replicas when idle and scale up automatically when incoming HTTP traffic is detected.

The KEDA HTTP interceptor acts as an entry point for checkout requests, buffering incoming traffic and triggering the creation of new pods when required. This enables scale-from-zero functionality, which is not possible with standard Kubernetes Services alone.

### Persistence Layer

A PostgreSQL database is deployed using a Kubernetes Deployment, with a Persistent Volume Claim (PVC) used to ensure data durability. The checkout service writes transaction records to the database, allowing verification that data persists across pod restarts and replacements.

### Request Flow

A typical request follows this sequence:

1. The client sends an HTTP request to the gateway  
2. The gateway forwards the request to the checkout endpoint  
3. The KEDA interceptor handles scaling if no checkout pods are running  
4. The checkout service processes the request  
5. The checkout service calls pricing and inventory services  
6. Results are combined and stored in PostgreSQL  
7. A response is returned to the client  

This flow demonstrates synchronous orchestration, dependency management, and distributed system behaviour under load and failure conditions.

## 3. Implementation

## 4. Testing and Troubleshooting

## 5. Performance and Scaling

## 6. Conclusion