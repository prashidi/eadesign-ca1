# Enterprise Architecture Design - Contineous Assessment 1

## 1. Introduction
- Brief context
- What the assignment required
- Scope of the solution

## 2. Solution Design
- Nanoservices architecture
- Gateway, checkout, pricing, inventory, postgres
- KEDA scale-to-zero decision
- Timeout and failure handling
- Request correlation using X-Request-Id
- Persistence using Postgres + PVC

## 3. Implementation
- Docker Compose baseline
- Kubernetes Deployments, Services, Ingress
- ConfigMaps
- Secrets
- PVC
- KEDA HTTP scaling

## 4. Testing and Troubleshooting
- Happy path
- Out-of-stock
- Partial failure
- Bad rollout
- Readiness failure
- Toolbox connectivity
- Logs, events, endpoints evidence

## 5. Performance and Scaling
- Cold vs warm checkout timings
- Scale-from-zero
- Scale-to-zero
- Short discussion of cold start trade-off

## 6. Conclusions and Recommendations
- Summary of what was achieved
- Benefits and trade-offs of nanoservices
- Suitability of KEDA for this architecture
