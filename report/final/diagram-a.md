# Diagram A - System architecture (components + routing)

```mermaid
flowchart TD
    Client[Browser / curl] --> Ingress[Ingress (Traefik)]
    Ingress --> Gateway[gateway-svc (NGINX/UI)]
    Gateway --> Checkout[checkout-svc]

    Checkout --> Pricing[pricing-svc]
    Checkout --> Inventory[inventory-svc]
    Checkout --> Postgres[postgres-svc (PVC)]