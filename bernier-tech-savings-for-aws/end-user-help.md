# Bernier Tech Savings for AWS (Standard Edition)

Offline, customer-run AWS cloud savings CLI.

Disclaimer: AWS is a trademark of Amazon Web Services, Inc. Bernier Tech Ltd is not affiliated with or endorsed by Amazon Web Services, Inc.

## Guarantees

- No telemetry
- No external network calls except AWS APIs
- No automatic remediation
- Customer data stays local except output reports

## Editions

- `free` (default when no license is configured)
	- Full scan still runs across all rules/resources.
	- Output is gated to top 10 findings only.
	- Reports include: `FREE MODE: showing 10 of N findings`.
- `standard`
	- Full findings output.
- `pro`
	- Remediation guidance included, AWS CLI help

## CLI

```bash
bernier-tech-savings-for-aws scan --regions eu-west-1,us-east-1 --output ./out --formats html,json,csv,pdf --config ./config.yaml --since-days 14 --max-concurrency 5

# optional: override rules_file from config.yaml
bernier-tech-savings-for-aws scan --config ./config.yaml --rules ./rules.yaml --regions us-east-1 --output ./out
```

Authentication resolution (CLI + UI jobs) supports all standard AWS methods:

- Default AWS SDK chain (environment variables, shared config, AWS SSO, EC2/ECS role credentials, web identity)
- `--profile` shared config profile
- `--role-arn` with optional `--external-id` (assume role on top of resolved base credentials)
- Explicit credentials flags: `--access-key-id`, `--secret-access-key`, optional `--session-token`

Example with explicit temporary credentials:

```bash
bernier-tech-savings-for-aws scan --regions us-east-1 --access-key-id <id> --secret-access-key <secret> --session-token <token> --output ./out --config ./config.yaml
```

`config.yaml` can reference an external rule catalog:

```yaml
rules_file: ./rules.yaml
```

Recommended baseline `config.yaml` (production starter):

```yaml
rules_file: ./rules.yaml
currency_default: USD

thresholds:
	since_days: 14

	ec2_idle_cpu_p95: 5
	ec2_idle_net_mb_day: 50
	ec2_underutil_cpu_p95: 20
	ec2_overutil_cpu_p95: 80
	stopped_days: 30

	ebs_unattached_days: 14
	snapshot_retention_days: 90

	elb_idle_req_per_day: 10
	nat_idle_gb_day: 1

	cwlogs_retention_days_default: 30
	s3_mpu_abort_days: 7

	rds_idle_cpu_p95: 10
	rds_idle_conn_avg: 1
	rds_underutil_cpu_p95: 20

	sp_coverage_min: 50
	sp_utilization_min: 90
	ri_util_min: 80

	elasticache_idle_cpu_p95: 5
	elasticache_idle_conn_p95: 5
	elasticache_idle_net_mb_day: 100
	elasticache_underutil_cpu_p95: 40
	elasticache_underutil_conn_p95: 100
	elasticache_ri_util_min: 80
	elasticache_nonprod_offhours_factor: 0.65
	elasticache_tiering_cpu_p95_max: 60

tag_keys:
	owner: ["Owner", "owner"]
	cost_center: ["CostCenter", "cost-center"]
	env: ["Environment", "env"]

pricing:
	cache_dir: ".bernier-tech-savings-cache"
	ttl: 24h
```

Calibration guidance:

- Lower `*_idle_*` thresholds to reduce false positives.
- Raise `sp_utilization_min` / `ri_util_min` as commitment discipline improves.
- Tune ElastiCache thresholds per engine/workload (Redis often needs different connection and CPU limits by service tier).

## Web UI (Global language coverage)

Launch the built-in local UI:

```bash
bernier-tech-savings-for-aws ui --config ./config.yaml --rules ./rules.yaml --host 127.0.0.1 --port 8080 --output-base ./out/ui
```

Then open `http://127.0.0.1:8080` in your browser.

UI features:

- Start scans without command-line usage
- Track run history and status (queued/running/succeeded/failed)
- Download generated reports (HTML, JSON, CSV, PDF)
- Switch language live from the selector: English, Mandarin Chinese, Hindi, Spanish, Japanese, German, Italian, French, Arabic, Bengali, Portuguese, Korean, Dutch, Russian, Urdu

Supported flags:

- `--profile`
- `--access-key-id`
- `--secret-access-key`
- `--session-token`
- `--role-arn`
- `--external-id`
- `--regions`
- `--output`
- `--formats`
- `--config`
- `--rules` (optional override for `rules_file`)
- `--since-days`
- `--max-concurrency`
- `--only-rules`
- `--exclude-rules`
- `--min-savings`
- `--verbose`
- `--fail-fast`

UI command flags:

- `ui --host`
- `ui --port`
- `ui --config`
- `ui --rules`
- `ui --output-base`

License command flags:

- `license status`

## Local license store

- Path: `${XDG_CONFIG_HOME}/bernier-tech-savings/license.json`
- Fallback path: `~/.config/bernier-tech-savings/license.json`

Commands:

```bash
bernier-tech-savings-for-aws license status
bernier-tech-savings-for-aws license set --edition standard
bernier-tech-savings-for-aws license set --key <string>
bernier-tech-savings-for-aws license set --dev-key <string>
bernier-tech-savings-for-aws license clear
```

Exit codes:

- `0` success
- `2` config error
- `3` auth/permissions (no regions scanned)
- `4` partial scan (some regions failed, report written)

For exit code `3`, provide credentials with one of:

- default AWS SDK chain (env vars, shared config/SSO, instance/container role)
- `--profile <name>`
- `--role-arn <arn>` (with optional `--external-id`)
- explicit keys: `--access-key-id <id> --secret-access-key <secret> [--session-token <token>]`

## Commercial distribution checklist

For paid/professional distribution, include and review these files in every shipped archive:

- `EULA.txt`
- `PROPRIETARY-NOTICE.txt`
- `THIRD-PARTY-NOTICES.txt`
- `LEGAL-NOTICE.txt`

## How realistic are savings calculations?

Short answer: **mixed by rule type**.

- Higher realism: deterministic waste rules (e.g., unattached EBS, unused EIP, gp2->gp3).
- Medium realism: rightsizing rules using CPU-only heuristics.
- Lower realism: advisory rules needing telemetry not always available in standard collector paths.

Practical calibration ranges:

- Deterministic resource-price rules: often around ±10% to ±25%
- CPU-only rightsizing heuristics: often around ±20% to ±40%
- Advisory findings: directional signal, not a firm savings amount

Use report fields `confidence` and `estimated_monthly_savings.basis` to prioritize implementation.
