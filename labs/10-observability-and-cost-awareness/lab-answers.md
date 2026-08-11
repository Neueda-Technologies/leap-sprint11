# Lab 10 Answers

## Part 1: What CloudWatch Already Knows

**ECS metrics published (namespace AWS/ECS):**
- `CPUUtilization` — ClusterName=lab07-cluster, ServiceName=lab08-auth-service
- `MemoryUtilization` — same dimensions
- `LiveTaskCount` — same dimensions

These exist even with no task currently running because ECS publishes metric metadata once a service
is (or was) active. The data points themselves may be absent, but the metric dimensions remain.

**Logs Insights query (last 7 days, log group `ecs-lab07-auth-service`):**
```
fields @timestamp, @message
| filter @message like /started/
| sort @timestamp desc
| limit 5
```

Results confirmed: 3 startup events from today's lab work —
`[NestApplication] Nest application successfully started` at 12:24, 12:20, and 11:20 UTC.

## Part 2: Alarm and Dashboard

**CPU alarm:** `lab10-ecs-cpu-high`
- Metric: `AWS/ECS CPUUtilization` for lab07-cluster / lab08-auth-service
- Threshold: > 80% average over 2 × 60s periods
- `--treat-missing-data notBreaching`

**Expected state:** `INSUFFICIENT_DATA` on creation (no datapoints yet), transitioning to `OK`
after the first evaluation with missing data treated as not breaching. Confirmed: initial state
was `INSUFFICIENT_DATA` — "Unchecked: Initial alarm creation".

**Dashboard:** `lab10-ecs-dashboard`
- Widget 1 (metric): ECS CPUUtilization + MemoryUtilization time series
- Widget 2 (log): Logs Insights query for startup events, table view

No validation errors on create.

## Part 3: Cost Data

**14-day spend by service (relevant lab work):**

| Service | Cost | Module |
|---------|------|--------|
| Amazon Virtual Private Cloud | $0.010 | Lab 3 (VPC) + Lab 8 (PrivateLink endpoints) |
| Amazon CloudFront | ~$0.000 | Lab 5 (CloudFront + OAC) |
| Amazon EC2 Container Registry | $0.003 | Lab 6 (ECR image storage) |
| Amazon Elastic Container Service | $0.015 | Labs 7–8 (Fargate task/service hours) |
| Amazon Elastic Load Balancing | $0.045 | Lab 8 (ALB hours) |
| Amazon Relational Database Service | $0.007 | Lab 9 (RDS instance hours) |
| AWS Secrets Manager | $0.052 | Lab 9 (managed password secret) |
| Amazon Simple Storage Service | $1.049 | Lab 4 (S3 static hosting) + pre-existing |
| EC2 - Other | $1.059 | Pre-existing infrastructure (NAT Gateway etc.) |

**Hourly rates vs. scaled 14-day figures (eu-west-2 approximations based on us-east-1 published rates):**

| Resource | AWS hourly rate | 730h/month cost | 14-day × 30/14 |
|----------|----------------|-----------------|----------------|
| NAT Gateway | $0.045/hr | $32.85 | N/A (not used — VPC endpoints instead) |
| ALB | $0.008/hr + LCU | ~$8/mo min | ~$0.10 |
| ECS Fargate (256 CPU/512 MB) | ~$0.012/hr | ~$8.76 | ~$0.03 |
| VPC interface endpoints (×3) | $0.01/hr each | $21.90 | ~$0.02 |
| RDS db.t3.micro | $0.016/hr | $11.68 | ~$0.01 |

**Which is more useful for planning?** The published hourly rate × 730 is far more useful.
The 14-day figure is tiny because resources were torn down minutes after verification — it captures
"lab exercise" usage patterns, not production patterns. A real production budget requires knowing
what continuous 24/7 operation actually costs. The scaled figure ($0.01 × 30/14 = $0.021) would
wildly underestimate the true $11.68/month for RDS.

## Budget

Created `lab10-monthly-budget`:
- Limit: $10/month (COST, MONTHLY)
- Alert: 80% of actual spend → email notification to zack.todd@watchelm.com

## Reflection: VPC Endpoints vs NAT Gateway

If a teammate's account shows `EC2 - Other` (NAT Gateway hours) and no `Amazon Virtual Private
Cloud` (PrivateLink) charge, it means they used a NAT Gateway to give private subnet tasks
outbound internet access for ECR pulls — the simpler but more expensive approach. VPC endpoints
keep traffic on the AWS private network and cost less per hour for high-throughput workloads,
but have a higher fixed per-endpoint cost for three services (~$0.03/hr). For this training
account, either approach works; in production, PrivateLink is preferred for security
(traffic never leaves the AWS network) regardless of cost.
