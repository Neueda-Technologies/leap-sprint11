# Lab 8 Answers

## Infrastructure Created

### VPC Endpoints (private subnets, endpoint SG sg-0fb3dfd0887ac02d1)
- `vpce-0142b5f842071b75c` — com.amazonaws.eu-west-2.ecr.api (interface)
- `vpce-0f74c922e8b8feb06` — com.amazonaws.eu-west-2.ecr.dkr (interface)
- `vpce-0988b70d59dea06a6` — com.amazonaws.eu-west-2.logs (interface)
- `vpce-06af9fe7c2077f6c0` — com.amazonaws.eu-west-2.s3 (gateway, on private route table rtb-0760d5fbb4365eaae)

Interface endpoints needed because the ECS task runs in private subnets with no internet access.
The S3 gateway endpoint is required by the ECR image pull process (ECR stores image layers in S3).

### Application Load Balancer
- Name: `lab08-auth-alb`
- DNS: `lab08-auth-alb-110782465.eu-west-2.elb.amazonaws.com`
- Subnets: public (subnet-030668c9a6336d586, subnet-0840db5f5f84c74a4)
- Security group: `sg-01b813c72a5489560` (LB SG — TCP 80 from 0.0.0.0/0)
- Listener: port 80 → target group `lab08-auth-tg`
- Target group: IP type, port 3000, health check GET /health

### ECS Service
- Name: `lab08-auth-service` on cluster `lab07-cluster`
- Task definition: `lab07-auth-service:1`
- Launch type: FARGATE, desired count: 1
- Subnets: private (subnet-05d13fe57dfc105c9, subnet-03783f13db6c23823)
- Security group: `sg-0fa5a6eadb011ab80` (app SG)
- Public IP: DISABLED (traffic enters via ALB only)
- Health check grace period: 30s

## Verification

**Target group health:**
```
IP: 10.1.4.136  Port: 3000  State: healthy
```

**HTTP via ALB:**
```
curl http://lab08-auth-alb-110782465.eu-west-2.elb.amazonaws.com/health → HTTP 200
```

**Force redeployment:** `aws ecs update-service --force-new-deployment` → 2 deployments active, then stabilised back to 1.

## Why Private Subnets + ALB?

The task runs in private subnets — no public IP, no direct internet access. This is the production pattern:
- Attack surface is the ALB only (one public endpoint, SG-restricted)
- Container-to-ECR and container-to-CloudWatch traffic stays on the AWS private network via VPC endpoints
- ALB handles TLS termination (port 443 in production), health checks, and horizontal scaling

## Cleanup

Deleted (hourly-billed):
- ECS service `lab08-auth-service`
- ALB listener, ALB `lab08-auth-alb`, target group `lab08-auth-tg`
- Interface VPC endpoints: vpce-0142b5f842071b75c, vpce-0f74c922e8b8feb06, vpce-0988b70d59dea06a6

Left in place for subsequent labs:
- Cluster `lab07-cluster`, task definition `lab07-auth-service:1`, execution role, log group
- ECR repos, S3 gateway endpoint `vpce-06af9fe7c2077f6c0`, private route table `rtb-0760d5fbb4365eaae`
- VPC, all subnets, security groups, IGW
