# Evidence Checklist

## Design
- Architecture diagram
- Request flow diagram

## Functional tests
- /api/arch
- /api/ping
- successful checkout
- out-of-stock checkout

## Persistence
- insert row in postgres
- restart pod
- verify row still exists

## KEDA
- checkout at 0 replicas
- checkout scales up after traffic
- checkout returns to 0
- cold vs warm timing

## Request correlation
- same X-Request-Id in checkout
- same X-Request-Id in pricing
- same X-Request-Id in inventory

## Troubleshooting
- kubectl get deploy,pods,svc,ingress
- describe output
- logs output
- events output
- endpoints / endpointslices
- toolbox nslookup and curl
- bad rollout evidence
- readiness failure evidence
