# Step 6 Notes - PostgreSQL, PVC, Toolbox

## Secret
- db-creds created successfully

## PVC
- postgres-pvc status: Bound

## Postgres
- postgres pod running
- postgres-svc created

## Toolbox checks
- nslookup postgres-svc worked
- nc -zv postgres-svc 5432 worked

## Persistence proof
- created table persistence_test
- inserted row: hello pvc
- restarted postgres pod
- queried again and row still existed
