---
type: runbook
title: Rotate the payments database credentials
description: Zero-downtime credential rotation for the payments Postgres instance.
resource: urn:pulumi:prod::payments::aws:rds/instance:Instance::payments-db
tags: [payments, postgres]
timestamp: 2026-07-01T09:30:00Z
---

# How rotation works here

The ESC environment `acme/prod/payments-db` uses the `fn::rotate::postgres`
rotator with two database users (`app_user_a` and `app_user_b`). The rotator
owns the pair: at any time one user's credentials are exposed as `current`
and the other's as `previous`. A rotation resets the `previous` user's
password and promotes it to `current`; the old `current` becomes `previous`
and **stays valid**. The [Payments Worker](/services/payments-worker.md)
always reads `current` at startup, which is what makes rotation
zero-downtime.

# When to run

* On the regular 90-day rotation schedule.
* Immediately, if the credentials may have been exposed.

# Steps

1. Rotate the environment:

   ```console
   pulumi env rotate acme/prod/payments-db
   ```

   The rotator resets the `previous` user's password in RDS and swaps it to
   `current`. Nothing running is affected: pods hold the old credentials,
   which are now `previous` and still valid.
1. Roll the worker so new pods start with the new `current` credentials:

   ```console
   kubectl -n payments rollout restart deployment/payments-worker
   ```

   The rollout is gradual; old pods keep working on the still-valid
   `previous` credentials until they terminate.
1. Verify: queue depth returns to baseline and the worker's database
   connection errors stay at zero for 15 minutes.
1. Do **not** rotate again until every consumer has restarted. A second
   rotation would reset the credentials the old pods are still using —
   never rotate more often than your workloads refresh their configuration.

# If something goes wrong

The old credentials remain valid as `previous` until the *next* rotation,
so the rollback is to stop the rollout, fix the worker, and roll again —
no password needs to be restored.

# Citations

[1] [Pulumi ESC postgres rotator](https://www.pulumi.com/docs/esc/providers/rotators/postgres/)
[2] [Database user setup for rotation](https://www.pulumi.com/docs/esc/operations/rotation/db-user-setup/)
