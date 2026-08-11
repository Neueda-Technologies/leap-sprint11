# Lab 1 — Deployment Plan

## Part 1: Map each piece

| Piece | Moves to AWS? | AWS service | Network |
|---|---|---|---|
| `mission-ui` | Yes | S3 (static build) + CloudFront CDN | Public — needs to be reachable by any browser on the internet |
| `mission-service` | Yes | ECS Fargate, behind an Application Load Balancer | ALB is public; ECS task is in a private subnet, only reachable via ALB |
| `sprint8-auth-service` | Yes | ECS Fargate, behind an ALB | Same pattern as mission-service — private subnet, fronted by load balancer |
| Postgres | Yes | RDS | Private subnet only — never exposed to the internet, only reachable from ECS tasks in the same VPC |
| Kafka | **Stays local** | N/A | Not part of this sprint — AWS MSK is expensive and complex for a training sprint |

Kafka is the odd one out. It stays local on Docker Compose because AWS's managed Kafka (MSK) requires multiple broker nodes at minimum and is a multi-day topic on its own — deliberately out of scope.

## Part 2: One "submit order" request traced

```
Loading the UI (separate from API calls):
  Browser → CloudFront (public internet, TLS) → S3 (static HTML/JS/CSS files)

Submitting an order (once UI is loaded):
  Browser
    → ALB (public internet)
      → ECS Fargate: sprint8-auth-service [private subnet] (validates JWT)
      → ECS Fargate: mission-service [private subnet] (processes order)
        → RDS Postgres [private subnet] (persists order)
```

Public internet hops: browser→CloudFront and browser→ALB.
Private (inside AWS VPC): ECS tasks talking to RDS — never leaves AWS's network.

## Part 3: Where secrets live

Both the JWT signing secret and database password should live in **AWS Secrets Manager**, fetched by the ECS task at startup.

Hardcoding in the container image is wrong because:
1. Anyone with ECR read access can pull the image and extract the secret — that's a wider audience than it looks
2. Rotating a baked-in secret requires rebuilding and redeploying the entire image — Secrets Manager lets credentials change without touching the running task at all
