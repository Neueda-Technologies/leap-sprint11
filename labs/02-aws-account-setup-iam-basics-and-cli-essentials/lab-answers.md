# Lab 2 — AWS Account Setup, IAM Basics & CLI Essentials

## Part 1: What is AWS

1. AWS is "a collection of independent services" because each service (S3, EC2, IAM, ECS, RDS,
   etc.) is maintained, scaled, and billed independently — you use and pay for only what you
   need, and one service can be used without the others.
   - **S3**: object storage — stores files/blobs (static websites, images, backups)
   - **ECS**: container orchestration — runs Docker containers at scale
   - **IAM**: identity & access management — controls who can do what in the account

2. Switching region in the console causes the resource lists to change — resources are
   region-scoped, so an S3 bucket or ECS cluster in eu-west-2 is invisible when viewing
   eu-west-1.

3. AWS splits a region into multiple AZs so that a failure in one data centre (power outage,
   network issue) doesn't take down everything. A database in AZ-a keeps serving if AZ-b
   goes down.

4. eu-west-2 has **3 AZs**: eu-west-2a, eu-west-2b, eu-west-2c — all `available`.

5. "Which region" = which geographic area (London, Ireland, etc.).
   "Which AZ" = which physical data centre within that region.
   A database cares about AZ specifically because it needs to place replicas in *different*
   AZs to survive a single data-centre failure — region alone doesn't give you that guarantee.

## Part 2: Console exploration

6. Used the top search bar to navigate directly to S3, EC2, and IAM.

7. IAM user ARN: `arn:aws:iam::369274594770:user/zacktodd`

## Part 3: CLI access confirmed

8. `aws sts get-caller-identity` output:
   - Account: `369274594770`
   - ARN: `arn:aws:iam::369274594770:user/zacktodd`
   - Matches the console.

9. `aws configure list` shows active profile, eu-west-2 region, access key masked to last 4
   characters only.

10. CLI default region matches console region selector (eu-west-2).

## Part 4: IAM permissions

11. **Policies** are the only IAM building block that actually contains permissions.
    - Users: an identity (a person or app)
    - Groups: a collection of users — policies attach to the group, not the users directly
    - Roles: an identity assumed temporarily (by a service, Lambda, ECS task, etc.)
    - All three are just targets that policies get attached to.

12. Policy attached: `AdministratorAccess` — the name suggests full, unrestricted access to
    everything.

13. `"Action": "*"` means any AWS action is allowed. `"Resource": "*"` means on any resource
    in the account. Together: do anything to everything.

14. User belongs to no IAM Groups — all permissions come from the directly-attached policy.

## Part 5: Real commands

15. `aws s3 ls` — several buckets exist (shared training account).

16. VPCs in eu-west-2:
    - `vpc-0ceb4694bd9f34187` — 10.0.0.0/16 (training VPC)
    - `vpc-0d246fcb944bff9e7` — 172.31.0.0/16 (default VPC)

17. `aws ecs list-clusters` — returned empty list. Nothing deployed yet. ✅

## Reflection question

`AdministratorAccess` is wrong for an ECS task credential because the task only needs to
talk to RDS and Secrets Manager — giving it full admin access means a compromise of that
task credential gives an attacker full control of the entire AWS account.

It's wrong for a CI/CD pipeline credential for the same reason — a deploy pipeline only needs
permission to push to ECR and update ECS task definitions, not to delete databases or modify
IAM policies. Least-privilege applies to both.
