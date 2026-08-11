# Lab 7 Answers

## Part 1

- Cluster: `lab07-cluster` (eu-west-2)
- Execution role: `lab07-ecs-execution-role` (arn:aws:iam::369274594770:role/lab07-ecs-execution-role)
  with `AmazonECSTaskExecutionRolePolicy` attached
- Task definition: `lab07-auth-service:1` — Fargate, 256 CPU / 512 MB, image `leap-auth-service:d7a4c8b`
  logs → CloudWatch group `ecs-lab07-auth-service`

**Architecture check**: image is `amd64` (built on Windows x86-64) — no `runtimePlatform` override needed.

## Part 2

- Security group: `sg-0fa5a6eadb011ab80` (lab07-auth-task-sg) — TCP 3000 from 0.0.0.0/0
- Ran task in public subnet `subnet-030668c9a6336d586` (eu-west-2a) with public IP assigned
- Task reached `RUNNING` status in ~15 seconds
- Public IP: `35.178.156.27`

**CloudWatch logs confirmed startup:**
```
[NestApplication] Nest application successfully started
Listening on http://localhost:3000
```

**HTTP request from local machine:**
```
curl http://35.178.156.27:3000/health → HTTP 200
```

## Cleanup

Task stopped. Cluster, task definition, execution role, log group, and security group left in
place for Lab 8.
