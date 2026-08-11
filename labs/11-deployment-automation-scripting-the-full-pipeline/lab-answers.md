# Lab 11 Answers

## Part 1: Trust Without Stored Credentials

**IAM role:** `lab11-jenkins-deploy-role`
- Trust policy: `ec2.amazonaws.com` only
- Inline policy: `lab11-deploy-permissions` — scoped to specific resources:
  - `ecr:GetAuthorizationToken` on `*` (required — no resource-level scoping exists for this action)
  - ECR push actions on `arn:aws:ecr:eu-west-2:369274594770:repository/leap-auth-service` only
  - `ecs:RegisterTaskDefinition` on `*` (no resource ARN available until revision is registered)
  - `ecs:UpdateService` / `ecs:DescribeServices` on the one service ARN
  - `iam:PassRole` on `arn:aws:iam::369274594770:role/lab07-ecs-execution-role` only

**Instance profile:** `lab11-jenkins-deploy-profile` with `lab11-jenkins-deploy-role` attached.

## Part 2: Jenkinsfile

See `Jenkinsfile` in this directory. Key design decisions:

- **No `aws configure`** — the instance profile vends credentials automatically via IMDS.
- **Commit SHA tagging** — `git rev-parse --short HEAD` produces a deterministic, traceable tag.
- **Live task definition rendering** — the Deploy stage fetches the **current live** task definition
  via `describe-task-definition` and swaps only the image field before registering a new revision.
  This preserves all existing settings (secrets, environment, log config, CPU/memory).

## Part 3: Steps Verified Directly

**ECR login:**
```
aws ecr get-login-password --region eu-west-2 | docker login --username AWS ...
→ Login Succeeded
```

**Render + register task definition:**
- Fetched `lab07-auth-service` current live task def
- Swapped image to `:test-pipeline-run`, stripped read-only fields
- Registered as `lab07-auth-service:3` — confirmed correct image in output

**Stale task definition risk (Part 3, step 6):**
The pipeline fetches the **current live** definition (`describe-task-definition --task-definition lab07-auth-service`
returns the latest active revision). This is the correct approach.

A stale checked-in copy would carry forward any settings that diverged since it was written —
for example: a `secrets` entry pointing to a rotated or deleted secret ARN (causes task launch failure),
an old `DB_HOST` environment variable pointing to a deleted RDS instance, or missing a new required
environment variable added via the console. These failures only surface at deploy time, not at
Jenkinsfile authoring time, making them difficult to diagnose in production.

## Reflection: Instance Profile vs Long-Lived Key

With a long-lived access key stored in Jenkins:
- Someone must manually rotate it (AWS recommends every 90 days)
- If the key holder leaves the team, IT must track it down and revoke it
- If Jenkins is decommissioned, someone must remember to delete the key from IAM
- The key is static — if leaked in a log or config dump, it remains valid until manually deleted

With an instance profile:
- Credentials are temporary (15-minute STS tokens, auto-rotated by IMDS)
- When the EC2 instance is terminated, access stops automatically — no key to track down
- No credentials are stored anywhere — nothing to leak, rotate, or revoke manually
- IAM access can be revoked by detaching the instance profile or deleting the role policy
