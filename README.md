```python
# Snapshot cleaner (short snippet for README)
# Paste this snippet in the README to show the implemented behavior (not full source)

import os
import logging
from datetime import datetime, timezone, timedelta
import boto3
from botocore.config import Config

logger = logging.getLogger()
logger.setLevel(logging.INFO)

DRY_RUN = os.getenv("DRY_RUN", "true").lower() == "true"
AGE_DAYS = int(os.getenv("SNAPSHOT_AGE_DAYS", "30"))
EXEMPT_TAGS = dict(kv.split(":", 1) for kv in os.getenv("EXEMPT_TAGS", "").split(",") if kv)
REGION = os.getenv("AWS_REGION", "us-east-1")

ec2 = boto3.client("ec2", config=Config(retries={"max_attempts": 6}), region_name=REGION)

def is_exempt(snapshot):
    tags = {t["Key"]: t["Value"] for t in snapshot.get("Tags", [])} if snapshot.get("Tags") else {}
    return any(tags.get(k) == v for k, v in EXEMPT_TAGS.items())

def snapshot_age_days(snapshot):
    return (datetime.now(timezone.utc) - snapshot["StartTime"]).days

def volume_attached(volume_id):
    resp = ec2.describe_volumes(VolumeIds=[volume_id])
    vols = resp.get("Volumes", [])
    if not vols:
        return False
    return any(att.get("InstanceId") for att in vols[0].get("Attachments", []))

def handler(event, context):
    deleted = []
    paginator = ec2.get_paginator("describe_snapshots")
    for page in paginator.paginate(OwnerIds=["self"]):
        for snap in page.get("Snapshots", []):
            sid = snap["SnapshotId"]
            if is_exempt(snap):
                logger.debug("exempt %s", sid); continue
            if snapshot_age_days(snap) < AGE_DAYS:
                logger.debug("young %s", sid); continue
            vol = snap.get("VolumeId")
            if not vol:
                logger.debug("no volume %s", sid); continue
            if volume_attached(vol):
                logger.debug("attached %s (vol=%s)", sid, vol); continue
            if DRY_RUN:
                logger.info("[DRY RUN] would delete %s", sid)
            else:
                logger.info("deleting %s", sid)
                ec2.delete_snapshot(SnapshotId=sid)
            deleted.append(sid)
    return {"deleted_count": len(deleted), "deleted": deleted}
```
