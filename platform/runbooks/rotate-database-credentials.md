---
type: runbook
title: Rotate the payments database credentials
description: Zero-downtime credential rotation for the payments Postgres instance.
resource: urn:pulumi:prod::payments::aws:rds/instance:Instance::payments-db
tags: [payments, postgres]
timestamp: 2026-07-01T09:30:00Z
---

# When to run

* On the regular 90-day rotation schedule.
* Immediately, if the credentials may have been exposed.

Rotation is zero-downtime because the database has two application users
(`app_user_a` and `app_user_b`) and only one is active at a time. Rotating
means resetting the password of the inactive user, switching the
[Payments Worker](/services/payments-worker.md) over to it, and then
resetting the old one.

# Steps

1. Identify the inactive user: check the `payments-db/active_user` value in
   the Pulumi ESC environment `acme/prod/payments-db`.
1. Rotate the inactive user's password in ESC:

   ```console
   pulumi env rotate acme/prod/payments-db
   ```

   The environment's rotator resets the inactive user's password in RDS and
   writes the new secret; nothing consuming the active user is touched.
1. Flip `active_user` to the freshly rotated user in the same environment.
1. Roll the worker so it picks up the new credentials at startup:

   ```console
   pulumi up --stack prod
   ```

   in the `payments` project. The deployment rolls pods gradually; old pods
   keep working on the still-valid old credentials until they terminate.
1. Verify: queue depth returns to baseline and the worker's database
   connection errors stay at zero for 15 minutes.
1. Rotate the now-inactive user's password the same way, so no credential
   issued before this window remains valid.

# If something goes wrong

Flip `active_user` back and run `pulumi up` again — the previous credentials
remain valid until step 6, which is what makes the rollback safe.

# Citations

[1] [Pulumi ESC secret rotation documentation](https://www.pulumi.com/docs/esc/operations/rotation/)
