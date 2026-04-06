# Step 7 Notes - Checkout to Postgres

## Changes made
- Added PostgreSQL client to checkout-fn
- Added DB table initialization
- Added insert logic for checkout records

## Deployment changes
- Built image: ead/checkout-fn:v2
- Imported image into K3s
- Updated checkout Deployment with DB environment variables

## Verification
- checkout-fn started successfully
- table checkout_records created
- happy-path checkout inserted a confirmed row
- out-of-stock checkout inserted an out_of_stock row
