# Lab 5 Answers

## Part 1

- Bucket: `leap-zacktodd-mission-ui-cf` (eu-west-2) — Block Public Access left fully on
- Direct access confirmed denied: `<Code>AccessDenied</Code>`

## Part 2

- OAC: `E1Z38IA3FD7L17`
- Distribution: `E1E65RUK1OYGLX` — domain `d3evqj21bc2ui0.cloudfront.net`
- Bucket policy scoped to `cloudfront.amazonaws.com` service principal with
  `AWS:SourceArn: arn:aws:cloudfront::369274594770:distribution/E1E65RUK1OYGLX`

**Prediction for `put-bucket-policy` with Block Public Access still on:**
Block Public Access blocks policies that grant access to `Principal: *` (the anonymous public).
A `Principal: {"Service": "cloudfront.amazonaws.com"}` scoped to a specific distribution ARN
is NOT a public policy — it grants access to an AWS service under specific conditions.
Prediction: **succeeds**. Actual result: **succeeded** — prediction correct.

## Part 3

- Direct S3 after distribution created: still `AccessDenied` ✓ (bucket policy has no wildcard principal)
- CloudFront: `HTTP/1.1 200 OK` ✓
- Mission Control login screen renders over HTTPS ✓

## The question worth sitting with

Both are true simultaneously because the bucket policy grants access to the **CloudFront service
principal** (an AWS identity), not to `Principal: *` (the public). The bucket never becomes
public — it only allows requests that CloudFront signs with its OAC credentials. Visitors get
the content via CloudFront's HTTPS endpoint, but they can never reach the bucket directly;
Block Public Access stays fully on the whole time. "Publicly accessible website" and "private
bucket" aren't contradictory when CloudFront is the only authorised reader.

## Cleanup

Distribution disabled, waited for deploy, then deleted. Bucket emptied and deleted.
