# Quartz Scheduler Locking Mechanism

## Overview

Quartz uses database locking to ensure that only one scheduler instance executes a trigger at a time in a clustered environment.

## Key Quartz Tables

| Table | Purpose |
|---------|----------|
| QRTZ_JOB_DETAILS | Job definitions |
| QRTZ_TRIGGERS | Trigger metadata |
| QRTZ_FIRED_TRIGGERS | Currently executing triggers |
| QRTZ_LOCKS | Scheduler locks |
| QRTZ_SCHEDULER_STATE | Cluster node heartbeats |

## Locking Flow

1. Scheduler polls for ready triggers.
2. Acquires `TRIGGER_ACCESS` lock from `QRTZ_LOCKS`.
3. Selects eligible triggers.
4. Marks triggers as `ACQUIRED`.
5. Inserts execution record into `QRTZ_FIRED_TRIGGERS`.
6. Releases lock.
7. Executes job.

## Trigger Acquisition

Typical query:

```sql
SELECT *
FROM QRTZ_TRIGGERS
WHERE NEXT_FIRE_TIME <= current_time
AND TRIGGER_STATE='WAITING';
```

## Cluster Recovery

When a scheduler crashes:

1. Other nodes detect missing heartbeat.
2. Failed instance is identified.
3. Recoverable jobs are reassigned.
4. Execution continues on another node.

## Misfire Handling

Common options:

- Fire Immediately
- Ignore Misfires
- Do Nothing

Configuration:

```properties
org.quartz.jobStore.misfireThreshold=60000
```

## Batch Trigger Acquisition

```properties
org.quartz.scheduler.batchTriggerAcquisitionMaxCount=50
```

Quartz acquires triggers in batches to improve throughput.

## Recommended Configuration

```properties
org.quartz.jobStore.isClustered=true
org.quartz.jobStore.clusterCheckinInterval=15000
org.quartz.scheduler.batchTriggerAcquisitionMaxCount=50
org.quartz.scheduler.idleWaitTime=30000
```

## Best Practices

- Enable clustering.
- Use recoverable jobs for critical workloads.
- Monitor `QRTZ_SCHEDULER_STATE`.
- Index Quartz tables.
- Tune batch acquisition size based on workload.

## Summary

Quartz ensures single execution of triggers through database locking. In clustered deployments it provides fault tolerance, trigger recovery, and misfire handling, making it suitable for enterprise-grade scheduling workloads.
