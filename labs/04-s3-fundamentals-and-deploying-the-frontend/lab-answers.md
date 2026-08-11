# Lab 4 Answers

## Part 1

**Bucket created**: `leap-zacktodd-mission-ui` (eu-west-2)

**Default public access block** — all four settings are `true`:
- BlockPublicAcls, IgnorePublicAcls, BlockPublicPolicy, RestrictPublicBuckets

AWS made this the default because public buckets have been the cause of many high-profile data
breaches (exposed credentials, customer data, internal configs). Opt-in to public is safer than
opt-out — it forces a deliberate, explicit decision every time.

## Part 2

- Disabled Block Public Access, applied `s3:GetObject` to `*` policy, enabled static website hosting
- Built with `ng build --configuration production`
- Synced 12 files to `s3://leap-zacktodd-mission-ui`
- Verified: `curl http://leap-zacktodd-mission-ui.s3-website.eu-west-2.amazonaws.com` → **HTTP 200**
- Login screen renders in browser ✓

## Part 3

All objects landed in **STANDARD** storage class. This is the right choice for a frontend build —
files are served on every page load so low-latency retrieval matters. Classes like GLACIER or
DEEP_ARCHIVE trade retrieval latency for lower storage cost, which makes no sense for assets
that must respond in milliseconds on every request.

**Three components of an S3 bill:**
1. **Storage** — GB-months stored
2. **Requests** — GET, PUT, LIST operations (per 1,000)
3. **Data transfer out** — bytes served to the internet

For a widely-visited website, **data transfer out** dominates (lots of GETs serving JS/CSS to
users worldwide). For a rarely-visited one, **storage** dominates (you pay to keep the data
sitting there whether anyone requests it or not).

## Cleanup

Bucket emptied and deleted after verification (per lab instructions — public bucket should not
be left running).
