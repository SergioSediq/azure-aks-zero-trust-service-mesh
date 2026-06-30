# Architecture — Zero-Trust AKS Service Mesh

## Problem

Microservices at scale need **fine-grained segmentation**. Without a service mesh, any compromised pod can reach any other service — a poor fit for regulated environments (banking, healthcare).

## Solution components

| Component | Role |
|-----------|------|
| **Azure Kubernetes Service** | Hosts microservices with Azure CNI + network policy |
| **Istio (AKS add-on)** | mTLS, traffic management, AuthorizationPolicies |
| **Azure DNS Private Zones** | Internal DNS for `*.company.internal` service names |
| **Application Gateway** | North-south ingress with optional WAF |
| **Log Analytics** | Central logs, Container Insights, alerting |

## Traffic flow

1. External client → **Application Gateway** (WAF inspection)
2. App Gateway → **Istio ingress gateway**
3. Ingress → **frontend** namespace (authorized HTTP methods only)
4. Frontend → **backend** via private DNS + mTLS (AuthorizationPolicy allows `frontend` namespace)
5. Backend → **database** namespace (only `backend` may connect)

## Zero-trust principles applied

- **Default deny** at NetworkPolicy layer
- **Explicit allow** at Istio AuthorizationPolicy layer
- **Mutual TLS** between all meshed services
- **No public database exposure** — ClusterIP services only
- **Centralized audit** via Azure Monitor

## Cost notes

- AKS + App Gateway + Log Analytics: budget **~$200–400/month** for a dev/lab cluster
- Tear down with `terraform destroy` when not in use

Full deployment steps: [LAB_GUIDE.md](LAB_GUIDE.md)
