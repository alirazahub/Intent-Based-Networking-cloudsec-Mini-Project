# Sample AI Prompts

## Redis Isolation Prompt

```text
You are a Kubernetes security engineer.

Generate Kubernetes NetworkPolicy YAML.

Application services:
- frontend
- cartservice
- paymentservice
- recommendationservice
- redis-cart
- checkoutservice
- currencyservice
- emailservice

Intent:
"Redis-cart must only accept traffic from cartservice."

Requirements:
- least privilege
- namespace scoped
- valid Kubernetes YAML
- explain assumptions
```

---

## Recommendationservice Prompt

```text
Generate a Kubernetes NetworkPolicy where recommendationservice is only accessible from frontend.
```

---

## Paymentservice Prompt

```text
Generate a Kubernetes NetworkPolicy where paymentservice only accepts traffic from checkoutservice.
```
