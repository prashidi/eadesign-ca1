# Step 8 Notes - KEDA Scale-to-Zero

## Scale-from-zero
- checkout-fn was at 0 replicas
- request to /api/checkout triggered pod creation
- pod became Ready and request succeeded

## Scale-to-zero
- after idle period, checkout returned to 0 replicas

## Cold vs warm timing
- cold_checkout=7.907940s
- warm_checkout=0.043764s

## Observation
- cold request was slower due to scale-up, container startup, and readiness delay
- warm request was faster because the pod was already running
