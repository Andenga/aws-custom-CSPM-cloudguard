# CloudGuard CSPM

A cloud security posture management (CSPM) tool with a pluggable, custom detection rules engine — not just another script that lists your AWS resources.

## Why this exists

Most beginner cloud security projects stop at "call the API, print what's public." That tells you *what* exists, not *whether it's a problem*. CloudGuard is built around a configurable rules engine so detection logic can be added, tuned, and mapped to real security frameworks — the same way commercial CSPM tools (Wiz, Prisma Cloud, etc.) work, just scoped down to a learnable size.

## What it does

- Scans a defined set of AWS services for common misconfigurations (see [Scope](#scope))
- Evaluates findings against a rules engine defined in YAML/JSON — no code changes needed to add a new check
- Maps each rule to a CIS Benchmark control (or MITRE ATT&CK Cloud tactic, where relevant)
- Scores findings by severity and produces a report (CLI output / dashboard — pick what you build)
- Suggests remediation (a CLI command or Terraform snippet) alongside each finding

## Scope

Intentionally narrow rather than trying to cover everything:

- **S3** — public buckets, missing encryption, missing versioning/logging
- **IAM** — overly permissive policies, unused access keys, missing MFA
- **EC2 / Security Groups** — open ingress (0.0.0.0/0) on sensitive ports
- **RDS** — public accessibility, unencrypted storage

## How the rules engine works

Each rule is defined declaratively, for example:

```yaml
id: S3-001
title: S3 bucket allows public read access
service: s3
severity: high
framework_mapping: CIS-2.1.5
check: bucket_policy.public_access == true
remediation: "aws s3api put-public-access-block --bucket <name> --public-access-block-configuration ..."
```

This means the detection logic is data, not code — new checks can be added without touching the scanner itself.

## Architecture

```
scanner/        # AWS API calls per service
rules/          # YAML rule definitions
engine/         # Loads rules, evaluates resources, produces findings
report/         # Formats findings (CLI / JSON / dashboard)
```

## Getting started

```bash
git clone https://github.com/<your-username>/cloudguard-cspm.git
cd cloudguard-cspm
pip install -r requirements.txt
python scan.py --profile <aws-profile>
```

## Sample output

```
[HIGH]   S3-001  my-public-bucket        Public read access enabled
[MEDIUM] IAM-004 ci-deploy-user          Access key unused for 90+ days
[LOW]    EC2-002 sg-0123456              SSH open to 0.0.0.0/0
```

## What I'd add next

- Multi-account / AWS Organizations support
- Historical scan storage to show configuration drift over time
- Scheduled scanning via Lambda + EventBridge

## Limitations

This is a learning/portfolio project, not a production security tool. It covers a deliberately narrow slice of AWS and hasn't been tested against a large, live production environment.

## License

MIT
