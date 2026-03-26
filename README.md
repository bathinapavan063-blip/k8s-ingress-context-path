# k8s-ingress-context-path
# k8s-ingress-context-path-routing



Demonstrates Kubernetes Ingress Context-Based Path Routing on AWS EKS using the AWS Load Balancer Controller. A single AWS ALB routes traffic to three different applications based purely on the URL path — no separate load balancer per service.

---

## How Routing Works

| Path | Routes To |
|------|-----------|
| `/` | App V1 — default backend |
| `/app1/` | App1 service (port 80) |
| `/app2/` | App2 service (port 80) |

---

## Architecture

```
Internet → AWS ALB (auto-provisioned by ALB Controller)
              ↓
         Ingress rules (path-based)
              ↓
   ┌──────────┬──────────┬──────────┐
 /app1/     /app2/      /
   ↓           ↓         ↓
 App1 Pod   App2 Pod   App3 Pod
```

---

## Files

```
├── app1-deployment.yml   # Deployment + NodePort service for App1
├── app2-deployment.yml   # Deployment + NodePort service for App2
├── app3-deployment.yml   # Deployment + NodePort service for App3 (default)
└── ingress.yml           # Ingress resource with path-based routing rules
```

---

## Pre-requisites

- AWS EKS cluster running
- AWS Load Balancer Controller installed via Helm
- IRSA configured for the controller
- Subnets tagged with `kubernetes.io/role/elb: 1`

---

## Deploy

```bash
kubectl apply -f app1-deployment.yml
kubectl apply -f app2-deployment.yml
kubectl apply -f app3-deployment.yml
kubectl apply -f ingress.yml

# Wait for ADDRESS — that's your ALB DNS
kubectl get ingress
```

---

## Cleanup — Important ⚠️

```bash
# Always delete Ingress FIRST — this tells the controller to delete the ALB
kubectl delete ingress ingress-cpr-demo

kubectl delete -f app1-deployment.yml
kubectl delete -f app2-deployment.yml
kubectl delete -f app3-deployment.yml
```

> **Warning:** Always delete the Ingress before deleting the cluster or nodes. If you skip this step, the AWS ALB keeps running and incurs charges even after the cluster is gone.

---

## Why Ingress Over LoadBalancer Service

| LoadBalancer Service | Ingress |
|----------------------|---------|
| 1 AWS LB per service | 1 AWS LB for all services |
| Expensive at scale | Cost efficient |
| No path/host routing | Full path and host-based routing |
| Hard to manage | Single entry point, clean rules |

Ingress is the standard pattern in production Kubernetes deployments.

---

