# Step 10 Notes - Troubleshooting and Evidence

## 1. Baseline state

The baseline cluster state was inspected using:

- kubectl get deploy,pods,svc,ingress
- kubectl get endpoints,endpointslices

This confirmed that the main workloads and services were present in the ecommerce namespace.  
It also showed which services had active endpoints and which pods were currently available.

---

## 2. Inside-cluster connectivity using toolbox

A toolbox pod was used to test service discovery and internal HTTP connectivity from inside the Kubernetes cluster.

Commands used:
- kubectl exec -it toolbox -- bash
- nslookup checkout-svc
- curl -i http://checkout-svc/health
- curl -i http://pricing-svc/health
- curl -i http://inventory-svc/health

Results:
- checkout-svc resolved successfully using Kubernetes DNS
- pricing-svc, inventory-svc, and checkout-svc all returned HTTP 200 on /health

Interpretation:
This confirmed that internal cluster networking and Kubernetes DNS were functioning correctly.

---

## 3. Partial failure scenario

The inventory service was intentionally scaled down to zero replicas.

Command used:
- kubectl scale deployment inventory-fn --replicas=0

Validation:
- kubectl get pods -l app=inventory-fn
- curl -s http://localhost/api/ping
- curl -s http://localhost/api/arch
- curl -i -H 'Content-Type: application/json' -d '{"sku":1,"subtotal":100}' http://localhost/api/checkout
- kubectl get endpoints inventory-svc -o wide
- kubectl logs -l app=checkout-fn --tail=100

Results:
- gateway endpoints such as /api/ping and /api/arch continued to work
- checkout failed clearly because inventory was unavailable
- inventory-svc had no endpoints

Interpretation:
This demonstrated partial system failure. The public entrypoint remained operational, but the checkout workflow failed due to a missing dependency.

Recovery:
- kubectl scale deployment inventory-fn --replicas=1
- kubectl rollout status deployment/inventory-fn

---

## 4. Bad rollout scenario

A bad rollout was simulated by setting the pricing service image to a non-existent tag.

Command used:
- kubectl set image deployment/pricing-fn pricing-fn=ead/pricing-fn:broken-tag

Validation:
- kubectl rollout status deployment/pricing-fn
- kubectl get pods -l app=pricing-fn -o wide
- kubectl describe pod -l app=pricing-fn
- kubectl get events --sort-by=.metadata.creationTimestamp | tail -n 30

Observed behaviour:
- Kubernetes failed to pull the image
- pod status moved to ErrImagePull or ImagePullBackOff
- describe and events showed that the image did not exist or could not be pulled

Example interpretation:
The deployment failed because the specified image tag did not exist. Kubernetes attempted to pull the image and reported an authorization or repository-not-found error.

Recovery:
- kubectl set image deployment/pricing-fn pricing-fn=ead/pricing-fn:v2
- kubectl rollout status deployment/pricing-fn
- kubectl get pods -l app=pricing-fn

Additional note:
During recovery, rollout completion was not immediate because Kubernetes replaced broken pods gradually. The rollout status showed intermediate states such as 1 out of 2 updated replicas, which is normal during rolling replacement.
---
A bad rollout was simulated by setting the pricing service image to a non-existent tag:

ead/pricing-fn:broken-tag  

The deployment failed with an image pull error (ErrImagePull / ImagePullBackOff).  
kubectl describe pod and kubectl get events showed that the image could not be pulled.

During recovery, the rollout did not complete immediately because Kubernetes replaces pods gradually. The rollout status showed partial progress (for example, 1 out of 2 replicas updated), indicating that one new pod was running while the second was still starting or waiting for readiness. This behaviour is expected during rolling updates.

The issue was resolved by restoring the correct image:

ead/pricing-fn:v2  

The deployment successfully rolled out after the fix.
---
During troubleshooting, inventory and pricing service startup issues were traced to outdated image references in the deployment manifests. After rebuilding and updating both services to v3, the rollout completed successfully and service endpoints were restored.
---

## 5. Readiness failure scenario

A readiness failure was simulated by intentionally changing the readiness probe path for pricing-fn from /health to an invalid path.

Validation commands:
- kubectl apply -f k8s/manifests/10-functions.yaml
- kubectl get pods -l app=pricing-fn -w
- kubectl get pods -l app=pricing-fn
- kubectl get endpoints pricing-svc -o wide
- kubectl describe pod -l app=pricing-fn

Observed behaviour:
- the pod entered Running state
- the pod did not become Ready
- pricing-svc had no endpoints

Interpretation:
This demonstrated the difference between Running and Ready. A running container is not automatically eligible to receive traffic. Kubernetes only routes traffic to pods that pass readiness checks.

Recovery:
The readiness probe path was restored to /health and the deployment was reapplied successfully.

---

## 6. Logs, describe, and events workflow

The following repeatable troubleshooting process was used throughout the assignment:

1. kubectl get deploy,pods,svc,ingress
2. kubectl describe pod <pod-name>
3. kubectl logs <pod-name>
4. kubectl get events --sort-by=.metadata.creationTimestamp | tail -n 40
5. kubectl get endpoints,endpointslices

Interpretation:
This workflow made it possible to distinguish between:
- image pull failures
- readiness failures
- missing service endpoints
- dependency outages
- successful service recovery

---

## 7. Summary of operational evidence

Step 10 produced evidence for:
- baseline workload visibility
- internal DNS and service reachability
- partial failure handling
- bad rollout detection and recovery
- readiness failure behaviour
- use of logs, events, describe output, and service endpoints for diagnosis
