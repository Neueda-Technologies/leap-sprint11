# Lab 6 Answers

## Part 1

Built both images, tagged each with `latest` and the git commit SHA:
- Mission service (`025eeeb`): `369274594770.dkr.ecr.eu-west-2.amazonaws.com/leap-mission-service:latest` and `:025eeeb`
- Auth service (`d7a4c8b`): `369274594770.dkr.ecr.eu-west-2.amazonaws.com/leap-auth-service:latest` and `:d7a4c8b`

**`describe-images` on leap-mission-service — 3 entries, 1 tagged:**
- sha256:ed34ad... — tags: ['latest', '025eeeb'] (the real image; two tags, same digest)
- sha256:febe0f... — no tags
- sha256:545ef9... — no tags

The 2 untagged entries are intermediate build-stage manifests from the multi-stage Dockerfile
(the Maven build stage). `docker build` creates layer manifests for each stage internally;
these don't get named, so they land in ECR untagged. This is not an error — it's normal
multi-stage build behaviour.

## Part 2

Lifecycle policy applied to both repos: expire untagged images after 1 day.

Why it matters: each build push from a CI pipeline produces one or more untagged intermediate
manifests. After months of daily CI runs, hundreds of untagged images accumulate. Each one is
small, but collectively they add up to meaningful storage cost, and ECR bills per GB-month.
The policy keeps the repo clean automatically without touching any tagged (deployable) image.

## Part 3

- Deleted local `leap-auth-service:d7a4c8b` and pulled from ECR — digest matched
- Container started: logs showed `Nest application successfully started` and
  `Listening on http://localhost:3000`
