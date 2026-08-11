# Lab 12 Answers

## Part 1: Account Audit

**ECS services:** none running  
**RDS instances:** none  
**NAT gateways (available):** none  
**Load balancers:** none  

All billed resources from earlier labs have been torn down. Resources left in place (zero cost at idle):
- Cluster `lab07-cluster`, task definitions `lab07-auth-service:1–3` and `lab09-db-check:1`
- ECR repositories `leap-auth-service` and `leap-mission-service`
- Execution role `lab07-ecs-execution-role`, Jenkins deploy role `lab11-jenkins-deploy-role`
- CloudWatch log group `ecs-lab07-auth-service`, dashboard `lab10-ecs-dashboard`, alarm `lab10-ecs-cpu-high`
- Budget `lab10-monthly-budget`
- VPC, subnets, security groups, IGW, route tables, S3 gateway endpoint `vpce-06af9fe7c2077f6c0`
- DB subnet group `lab09-db-subnet-group`, RDS security group `sg-0c271840d76d658dc`

**CloudWatch alarm state:** moved from `INSUFFICIENT_DATA` → `OK` at 14:01 UTC.
Reason: "no datapoints received for 2 periods and 2 missing datapoints treated as [NonBreaching]."
This is the expected transition — with `--treat-missing-data notBreaching`, after two evaluation
periods with no data (service deleted, no metrics published), the alarm concludes the threshold
is not being crossed and enters `OK`.

## Part 2: Pipeline Applied to Capstone

**Sprint 10 capstone** lives within the existing mission-ui / mission-service codebases (not a
separate service) — the Sprint 9 Angular additions (auth flow, holdings, place-order) are the
capstone extension.

No new ECR repository or task definition required — `leap-auth-service` and `lab07-auth-service`
already cover the backend. The Jenkinsfile from Lab 11 applies as-is with its existing environment
variables.

**New Playwright test added:** `mission-ui/e2e/orders.spec.ts`
Three checks covering the Sprint 9 capstone feature (place-order route):
1. Authenticated user can navigate to `/orders` via the nav link
2. Place Order form does not submit when required fields are empty (stays on `/orders`)
3. Unauthenticated visit to `/orders` redirects to `/login` (auth guard coverage)

## Part 3: Showcase Narrative

**Copilot suggestion that needed correcting:**  
During Sprint 9, Copilot suggested `expect(...).toBeTrue()` and `expect(...).toBeFalse()` in the
Vitest unit tests. These are Jasmine matchers — they don't exist in Vitest. The tests compiled
fine but would fail at runtime with "toBeTrue is not a function." Caught by running the test suite
(`npm test`) before committing; corrected to `toBeTruthy()` / `toBeFalsy()`.

**What pipeline automation prevented:**  
The Jenkinsfile's "render task definition from live" step — fetching the current registered
definition and swapping only the image — prevents a category of bug that manual deployment often
misses: deploying a stale task definition that's missing a secrets entry or environment variable
added after the last deploy. If someone added `DB_HOST` via the console in production, a manual
deployment copying from a checked-in JSON file would silently drop that setting. The pipeline
always starts from the live state, so nothing is lost.

## Reflection

The pipeline applies cleanly to the baseline mission because the auth service is a single
stateless container with no migrations. A real capstone extension that added a database schema
migration would require a genuine decision: run the migration as a separate ECS task before
`update-service`, or inline it in the container startup? The baseline never has to answer that
question — a migration step would be the one place the pipeline's shape genuinely needs changing,
not just the environment variables at the top.
