# Compose Test Notes

## Services
- pricing-fn
- inventory-fn
- checkout-fn
- gateway

## Basic checks
- /api/arch worked
- /api/ping worked
- /api/checkout happy path worked
- out-of-stock returned 409
- invalid input returned 400

## Partial failure
- stopped inventory-fn
- checkout failed
- gateway remained available

## Sample timing
- ping_total=...
- checkout_total=...
