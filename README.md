# AWS Cost Optimization — EBS Snapshot Lifecycle Cleaner

Purpose
- Automated, safe pruning of stale/unattached Amazon EBS snapshots to reduce recurring storage costs while preserving data integrity and auditability.

TL;DR
- What: Scheduled snapshot cleaner (Lambda) that discovers candidate snapshots, applies retention and tag-based exemptions, and deletes only when safety gates pass.
- Safety: Dry-run mode, configurable age threshold, tag whitelist, and paginated API use.
- Benefits: predictable cost reduction, centralized audit trail, minimal privileges.

Design summary
- Discovery: paginate DescribeSnapshots (OwnerIds=self) and correlate with DescribeVolumes / DescribeInstances.
- Safety gates:
  - Age threshold (configurable, default 30 days)
  - Tag-based exemptions (e.g., `retain:true`, `env:prod`)
  - Snapshot must reference a source VolumeId and that volume must not be attached
  - Skip AMI or cross-account snapshots unless explicitly enabled
- Reliability: paginators, exponential backoff on throttling, idempotent deletions, conservative failure handling.
- Observability: CloudWatch Logs + custom metrics (SnapshotsExamined, SnapshotsDeleted, SnapshotDeleteErrors) + alarms.

Configuration (env vars)
- DRY_RUN=true|false
- SNAPSHOT_AGE_DAYS=30
- EXEMPT_TAGS=retain:true,env:prod
- REGIONS=us-east-1,us-west-2
- MAX_CONCURRENT_DELETES=5
- ENABLE_AUDIT_DDB=true|false

Least-privilege IAM (minimum)
- ec2:DescribeSnapshots
- ec2:DescribeVolumes
- ec2:DescribeInstances
- ec2:DeleteSnapshot
- plus CloudWatch Logs permissions; scope by account/region where possible.

Deployment & rollout
1. Deploy to a non-production account/region with DRY_RUN=true.
2. Run daily for 7–14 days, review logs/metrics and tune EXEMPT_TAGS/AGE_DAYS.
3. Move to controlled maintenance window and flip DRY_RUN=false.
4. Disable EventBridge rule to pause.

Observability & runbook (abridged)
- Metrics: monitor deletion spikes and errors; alert on SnapshotDeleteErrors > 0.
- Pause automation: disable EventBridge rule or set DRY_RUN=true.
- Incident: if deletion is accidental, consult logs for snapshot IDs; recovery depends on backups/replicas—treat deletes as irreversible.

Where to find implementation
- Implementation, tests, and infra templates live in the repository (see lambda/snapshot-cleaner and infra/ folders). README omits full source intentionally—open the implementation module for exact logic and infra artifacts.
