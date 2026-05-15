# Microservice Communication Flow

## Online Boutique Connectivity

frontend
├── productcatalogservice
├── recommendationservice
├── cartservice
├── checkoutservice
└── adservice

checkoutservice
├── paymentservice
├── shippingservice
├── emailservice
├── cartservice
├── productcatalogservice
└── currencyservice

recommendationservice
└── productcatalogservice

cartservice
└── redis-cart

paymentservice
└── external payment processing simulation

---

# Important Service Ports

| Service | Port |
|---|---|
| frontend | 80 |
| recommendationservice | 8080 |
| paymentservice | 50051 |
| redis-cart | 6379 |
| productcatalogservice | 3550 |
| checkoutservice | 5050 |
