---
type: service
title: Payments Worker
description: Queue consumer that executes payment transactions against the payments Postgres instance.
resource: urn:pulumi:prod::platform-eks::kubernetes:apps/v1:Deployment::payments-worker
tags: [payments, kubernetes, postgres]
timestamp: 2026-07-14T09:00:00Z
owner: payments-team
oncall: "#oncall-payments"
---

# Overview

The Payments Worker consumes payment jobs from SQS and executes them against
the payments Postgres instance (RDS,
`urn:pulumi:prod::payments::aws:rds/instance:Instance::payments-db`). It is
the only workload with write access to that database.

# Operations

| Property    | Value                                           |
|-------------|-------------------------------------------------|
| Runtime     | EKS, `prod` cluster, `payments` namespace       |
| Deployed by | Pulumi project `payments`, stack `prod`         |
| Database    | RDS Postgres `payments-db`, credentials from Pulumi ESC |
| Scaling     | KEDA on queue depth, 2–16 replicas              |

# Database credentials

Credentials are injected from a Pulumi ESC environment at deploy time; the
worker never stores them. Rotation is a routine, zero-downtime procedure —
see [Rotate the payments database credentials](/runbooks/rotate-database-credentials.md).

# Dependencies

* Consumes payment jobs enqueued by the [Checkout API](/services/checkout-api.md).
* Runs on the shared EKS cluster, so it is affected by
  [Why we pin the VPC CNI version](/decisions/why-we-pin-the-vpc-cni-version.md).
