# Evaluation Summary

## Redis Isolation

- cartservice → redis-cart = Allowed
- frontend → redis-cart = Blocked

Result: Passed

---

## Recommendationservice Restriction

- frontend → recommendationservice = Allowed
- paymentservice → recommendationservice = Blocked

Result: Passed

---

## Paymentservice Restriction

- checkoutservice → paymentservice = Allowed
- frontend → paymentservice = Blocked

Result: Passed

---

## Product Catalog Restriction

- frontend → productcatalogservice = Allowed
- checkoutservice → productcatalogservice = Allowed
- recommendationservice → productcatalogservice = Allowed
- paymentservice → productcatalogservice = Blocked

Result: Passed
