# Step 9 Notes - Request Correlation

## Request used
RID=prashidi-123

## Curl command
curl -i http://localhost/api/checkout \
  -H "X-Request-Id: prashidi-123" \
  -H 'Content-Type: application/json' \
  -d '{"sku":1,"subtotal":100}'

## Evidence
- checkout-fn log showed prashidi-123
- pricing-fn log showed prashidi-123
- inventory-fn log showed prashidi-123

```
student@ead-server:~/eadesign-ca1$ RID=prashidi-123

curl -i http://localhost/api/checkout \
  -H "X-Request-Id: $RID" \
  -H 'Content-Type: application/json' \
  -d '{"sku":1,"subtotal":100}'
HTTP/1.1 200 OK
Content-Length: 161
Content-Type: application/json; charset=utf-8
Date: Mon, 06 Apr 2026 11:16:56 GMT
Etag: W/"a1-oYYdLaQIlYtKduHOZT8D+i6As1Q"
X-Keda-Http-Cold-Start: false
X-Powered-By: Express
X-Request-Id: prashidi-123

{"ok":true,"requestId":"prashidi-123","sku":1,"price":{"subtotal":100,"taxRate":0.23,"tax":23,"total":123},"stock":{"sku":1,"inStock":true},"status":"confirmed"}student@ead-server:~/eadesign-ca1$ 
student@ead-server:~/eadesign-ca1$ 
student@ead-server:~/eadesign-ca1$ kubectl logs -l app=checkout-fn --tail=200 | grep prashidi-123
[rid=prashidi-123] POST /api/checkout
[checkout-fn] POST /checkout request_id=prashidi-123 sku=1 subtotal=100
[rid=prashidi-123] POST /api/checkout
[checkout-fn] POST /checkout request_id=prashidi-123 sku=1 subtotal=100
student@ead-server:~/eadesign-ca1$ 
student@ead-server:~/eadesign-ca1$ 
student@ead-server:~/eadesign-ca1$ kubectl logs -l app=pricing-fn --tail=200 | grep prashidi-123
[pricing-fn] POST /price request_id=prashidi-123 subtotal=100
[pricing-fn] POST /price request_id=prashidi-123 subtotal=100
student@ead-server:~/eadesign-ca1$ 
student@ead-server:~/eadesign-ca1$ kubectl logs -l app=inventory-fn --tail=200 | grep prashidi-123
[inventory-fn] GET /stock/1 request_id=prashidi-123
[inventory-fn] GET /stock/1 request_id=prashidi-123
```
## Conclusion
The same request ID was propagated across multiple services, providing tracing-lite observability.
