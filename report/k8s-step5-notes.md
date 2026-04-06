# Step 5 Kubernetes Baseline Notes

## Images built and imported
- ead/pricing-fn:v1
- ead/inventory-fn:v1
- ead/checkout-fn:v1

## Kubernetes resources created
- pricing-fn Deployment + Service
- inventory-fn Deployment + Service
- checkout-fn Deployment + Service
- gateway Deployment + Service
- ead-ingress

## Functional tests
- / worked
- /api/arch worked
- /api/ping worked
- /api/checkout happy path worked
- out-of-stock returned 409

## Troubleshooting evidence used
- kubectl get deploy,pods,svc,ingress
- kubectl describe pod
- kubectl logs
- kubectl get endpoints
- kubectl get endpointslices
