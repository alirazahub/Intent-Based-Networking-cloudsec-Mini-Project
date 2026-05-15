# MP4 – Intent-Based Networking with AI

## Overview

This project explores the generation and enforcement of Kubernetes NetworkPolicies from high-level natural-language security intents using AI-assisted workflows.

The project evaluates whether AI-generated Kubernetes NetworkPolicies can correctly enforce least-privilege communication constraints in a realistic microservice environment.

The implementation includes:

- Kubernetes cluster deployment
- Multi-service application deployment
- Runtime topology analysis
- AI-assisted NetworkPolicy generation
- Connectivity verification
- Security evaluation and validation

---

# Environment

## Kubernetes Environment

| Component | Value |
|---|---|
| Kubernetes Distribution | Kind |
| Kubernetes Version | v1.33.1 |
| Cluster Type | Multi-node |
| Container Runtime | containerd |

---

# Applications

The following microservices from the Online Boutique application were deployed:

- frontend
- cartservice
- checkoutservice
- paymentservice
- recommendationservice
- productcatalogservice
- currencyservice
- emailservice
- shippingservice
- redis-cart
- adservice

---

# Tools and Technologies

- Kubernetes
- Kind
- kubectl
- Helm
- KubeSonde
- BusyBox
- Kubernetes NetworkPolicy
- OpenAI / LLM prompting
- Netcat (nc)

---

# Project Structure

```text
mp4-intent-networking/
├── evaluation/
├── policies/
├── screenshots/
├── slides/
├── topology/
├── manifests/
├── scripts/
├── microservices-demo/
├── kubesonde/
├── README.md
└── kind-config.yaml
```

---

# Cluster Setup

## Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind
```

Verify installation:

```bash
kind version
```

---

## Verify Kubernetes CLI

```bash
kubectl version --client
```

---

## Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify:

```bash
helm version
```

---

# Create Kubernetes Cluster

Create the Kind cluster:

```bash
kind create cluster --name mp4-cluster --config kind-config.yaml
```

Verify cluster:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                        STATUS   ROLES           AGE   VERSION
mp4-cluster-control-plane   Ready    control-plane   ...
mp4-cluster-worker          Ready    <none>          ...
mp4-cluster-worker2         Ready    <none>          ...
```

---

# Deploy Online Boutique

Clone repository:

```bash
git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
```

Deploy application:

```bash
cd microservices-demo

kubectl apply -f ./release/kubernetes-manifests.yaml
```

Verify deployment:

```bash
kubectl get pods
kubectl get svc
```

---

# Deploy KubeSonde

Clone KubeSonde:

```bash
git clone https://github.com/kubesonde/kubesonde
```

Deploy:

```bash
cd kubesonde

kubectl apply -f kubesonde.yaml
```

Verify:

```bash
kubectl get pods -A
```

---

# Access Application

Expose frontend:

```bash
kubectl port-forward svc/frontend-external 8080:80
```

Access application:

```text
http://localhost:8080
```

---

# Security Intents

The following security intents were defined in natural language.

## Intent 1

```text
Redis-cart must only accept traffic from cartservice.
```

---

## Intent 2

```text
Recommendationservice can only be accessed by frontend.
```

---

## Intent 3

```text
Paymentservice may only accept traffic from checkoutservice.
```

---

## Intent 4

```text
Productcatalogservice may only be accessed by:
- frontend
- checkoutservice
- recommendationservice
```

---

# AI-Generated Network Policies

All policies are located in:

```text
policies/
```

---

# Policy 1 – Redis Isolation Policy

File:

```text
policies/redis-policy.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-cart-allow-only-cartservice
  namespace: default

spec:
  podSelector:
    matchLabels:
      app: redis-cart

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: cartservice

      ports:
        - protocol: TCP
          port: 6379
```

Apply policy:

```bash
kubectl apply -f policies/redis-policy.yaml
```

---

# Policy 2 – Recommendationservice Restriction

File:

```text
policies/recommendation-policy.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: recommendationservice-allow-only-frontend
  namespace: default

spec:
  podSelector:
    matchLabels:
      app: recommendationservice

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend

      ports:
        - protocol: TCP
          port: 8080
```

Apply policy:

```bash
kubectl apply -f policies/recommendation-policy.yaml
```

---

# Policy 3 – Paymentservice Restriction

File:

```text
policies/payment-policy.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: paymentservice-allow-only-checkout
  namespace: default

spec:
  podSelector:
    matchLabels:
      app: paymentservice

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: checkoutservice

      ports:
        - protocol: TCP
          port: 50051
```

Apply policy:

```bash
kubectl apply -f policies/payment-policy.yaml
```

---

# Policy 4 – Product Catalog Restriction

File:

```text
policies/productcatalog-policy.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: productcatalogservice-allow-only-selected-services
  namespace: default

spec:
  podSelector:
    matchLabels:
      app: productcatalogservice

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend

        - podSelector:
            matchLabels:
              app: checkoutservice

        - podSelector:
            matchLabels:
              app: recommendationservice

      ports:
        - protocol: TCP
          port: 3550
```

Apply policy:

```bash
kubectl apply -f policies/productcatalog-policy.yaml
```

---

# Connectivity Verification

Connectivity testing was performed using temporary BusyBox pods and `nc`.

---

# Testing Methodology

## Allowed Traffic Test

Example:

```bash
kubectl run cart-test \
  --rm -it \
  --image=busybox \
  --labels="app=cartservice" \
  --restart=Never \
  -- sh
```

Inside pod:

```bash
nc -zv redis-cart 6379
```

Expected result:

```text
open
```

---

## Blocked Traffic Test

Example:

```bash
kubectl run frontend-test \
  --rm -it \
  --image=busybox \
  --labels="app=frontend" \
  --restart=Never \
  -- sh
```

Inside pod:

```bash
nc -zv -w 5 redis-cart 6379
```

Expected result:

```text
Connection timed out
```

---

# Validation Results

## Redis Isolation Validation

| Source | Destination | Result |
|---|---|---|
| cartservice | redis-cart:6379 | Allowed |
| frontend | redis-cart:6379 | Blocked |

---

## Recommendationservice Validation

| Source | Destination | Result |
|---|---|---|
| frontend | recommendationservice:8080 | Allowed |
| paymentservice | recommendationservice:8080 | Blocked |

---

## Paymentservice Validation

| Source | Destination | Result |
|---|---|---|
| checkoutservice | paymentservice:50051 | Allowed |
| frontend | paymentservice:50051 | Blocked |

---

## Product Catalog Validation

| Source | Destination | Result |
|---|---|---|
| frontend | productcatalogservice:3550 | Allowed |
| checkoutservice | productcatalogservice:3550 | Allowed |
| recommendationservice | productcatalogservice:3550 | Allowed |
| paymentservice | productcatalogservice:3550 | Blocked |

---

# Overall Evaluation Results

| Intent | Allowed Traffic | Unauthorized Traffic | Result |
|---|---|---|---|
| Redis isolation | Allowed | Blocked | Passed |
| Recommendation restriction | Allowed | Blocked | Passed |
| Payment restriction | Allowed | Blocked | Passed |
| Product catalog restriction | Allowed | Blocked | Passed |

---

# Observations and Findings

## Successful Findings

- Kubernetes NetworkPolicies successfully enforced least-privilege communication.
- AI-generated policies correctly isolated internal services.
- Microservice segmentation improved security boundaries.
- Connectivity verification confirmed correct enforcement behavior.

---

# Challenges Encountered

## Distroless Containers

Some application containers did not contain shell binaries such as:

```text
/bin/sh
```

This complicated direct debugging and connectivity testing.

---

## Hidden Dependencies

Some services relied on undocumented internal communication patterns.

---

## NetworkPolicy Complexity

Correct policy generation required:
- accurate pod labels
- correct service ports
- topology awareness

---

# Limitations

- Policies were manually verified.
- Dynamic scaling scenarios were not evaluated.
- DNS policies were not deeply analyzed.
- Egress restrictions were not fully implemented.

---


# Screenshots

Screenshots demonstrating:
- cluster deployment
- running services
- NetworkPolicy enforcement
- successful and blocked connectivity tests

are stored in:

```text
screenshots/
```

---

# Evaluation Files

Saved evaluation outputs:

```text
evaluation/
```

Including:
- pods.txt
- services.txt
- networkpolicies.txt

---

# References

- Kubernetes Documentation
- Kind Documentation
- Kubernetes NetworkPolicy Documentation
- Online Boutique Microservices Demo
- KubeSonde

---

# Author

Ali Raza  
Master’s Student – Computer Science  
Conservatoire national des arts et métiers (CNAM Paris)
