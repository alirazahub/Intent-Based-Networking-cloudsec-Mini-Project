# Topology Observations

## Initial Findings

- Multiple internal microservices communicate over ClusterIP services.
- Redis-cart is used exclusively by cartservice.
- recommendationservice depends on productcatalogservice.
- checkoutservice communicates with multiple backend services.

## Runtime Challenges

- Some dependencies were implicit and not directly visible from deployment manifests.
- emailservice initially experienced CrashLoopBackOff behavior.
- Distroless containers prevented direct shell debugging.

## Security-Relevant Findings

- Least-privilege communication paths could be isolated successfully using Kubernetes NetworkPolicies.
- Unauthorized lateral movement between services was blocked successfully after policy enforcement.
