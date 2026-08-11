# Lab 3 Answers

## Part 1: CIDR notation

1. A `/24` block has 2^(32-24) = 2^8 = **256** addresses. A `/16` has 2^(32-16) = 2^16 = **65,536** addresses.
   `/16` is bigger. A smaller suffix means fewer bits are fixed (the network prefix), so more bits are
   free for host addresses — the range is exponentially larger.

2. `10.0.4.0/24` covers `10.0.4.0`–`10.0.4.255`. `10.0.5.0/24` covers `10.0.5.0`–`10.0.5.255`.
   The third octet differs (4 vs 5), so these ranges are **completely separate** with no overlap.

## Part 2: Existing VPCs

Two VPCs found in eu-west-2:

| VPC ID | CIDR | Notes |
|--------|------|-------|
| vpc-0ceb4694bd9f34187 | 10.0.0.0/16 | CloudFormation-managed (WatchelmStack — tags show `aws:cloudformation:stack-name`) |
| vpc-0d246fcb944bff9e7 | 172.31.0.0/16 | Default VPC (IsDefault: true, no tags) |

## Part 3: New VPC

- **VPC**: `vpc-0cd8897f0637bc68f` (10.1.0.0/16) — avoids 10.0.0.0/16 and 172.31.0.0/16

**Subnets:**
| Subnet ID | CIDR | AZ | Intended |
|-----------|------|----|----------|
| subnet-030668c9a6336d586 | 10.1.1.0/24 | eu-west-2a | Public |
| subnet-05d13fe57dfc105c9 | 10.1.2.0/24 | eu-west-2a | Private |
| subnet-0840db5f5f84c74a4 | 10.1.3.0/24 | eu-west-2b | Public |
| subnet-03783f13db6c23823 | 10.1.4.0/24 | eu-west-2b | Private |

- **IGW**: `igw-0f5d75ecfef7ba7b5` — attached to vpc-0cd8897f0637bc68f, State: available ✓
- **Public route table**: `rtb-065923c658f4df9d1` — route `0.0.0.0/0 → igw-0f5d75ecfef7ba7b5`
- Associated with: subnet-030668c9a6336d586, subnet-0840db5f5f84c74a4 only

**Verification (route-table check only):**
- subnet-030668c9a6336d586 → PUBLIC (associated with rtb-065923c658f4df9d1 which has IGW route)
- subnet-05d13fe57dfc105c9 → PRIVATE (no explicit association; falls to main VPC route table — local only)
- subnet-0840db5f5f84c74a4 → PUBLIC (associated with rtb-065923c658f4df9d1 which has IGW route)
- subnet-03783f13db6c23823 → PRIVATE (no explicit association; falls to main VPC route table — local only)

## Part 4: Security Groups

- **SG1** (`sg-0c8420369c7e47142`, lab03-web-sg): port 80 open to 0.0.0.0/0
- **SG2** (`sg-01c3556bb4757e0bf`, lab03-app-sg): port 8080 allowed from SG1 only
  - `IpRanges`: [] (empty — no CIDR)
  - `UserIdGroupPairs`: [{ GroupId: sg-0c8420369c7e47142 }]

For a database SG (Part 4 question): allow SG2 (the app SG), not SG1. SG1 is open to the
entire internet on port 80 — allowing it to reach the database would mean any internet request
that reached the load balancer could potentially reach the DB. SG2 is only reachable from SG1,
meaning only the application tier can talk to the database.

## Part 5: Private subnets needing outbound access (conceptual)

To let instances in the private subnets pull from ECR, you'd need a **NAT Gateway** placed in
one of the *public* subnets (it needs an internet path itself), then a route `0.0.0.0/0 →
<nat-gateway-id>` added to the **private route table** (not the public one — the public subnets
already have their IGW route).

**NAT Gateway vs PrivateLink:**
- NAT Gateway gives general outbound internet access from private subnets — any destination.
  Useful when you need to reach arbitrary external services (package registries, third-party APIs).
- PrivateLink (VPC Interface Endpoints) keeps traffic entirely within the AWS network — no
  internet path at all. Better for AWS services (ECR, S3, Secrets Manager) where you want to
  enforce that traffic never leaves AWS, and avoid NAT Gateway data-processing costs.
