---
type: decision
title: Why we pin the VPC CNI version
description: The EKS VPC CNI add-on is pinned to an explicit version and upgraded deliberately, after a March 2026 networking incident.
resource: urn:pulumi:prod::platform-eks::aws:eks/addon:Addon::vpc-cni
tags: [kubernetes, networking, incident]
timestamp: 2026-03-20T16:00:00Z
status: accepted
---

# Context

On 2026-03-14, during a routine node group rotation, the VPC CNI add-on
auto-upgraded to the latest compatible version. New nodes came up with a CNI
regression: pods stuck in `ContainerCreating` with IP allocation failures.
Both the [Checkout API](/services/checkout-api.md) and the
[Payments Worker](/services/payments-worker.md) lost capacity until the
add-on was manually downgraded. Total impact: 41 minutes of degraded
checkout availability.

The root cause was not the specific CNI bug. It was that the add-on version
was not part of the reviewed infrastructure: `addonVersion` was unset, so
upgrades happened implicitly, outside any pull request.

# Decision

Pin `addonVersion` explicitly in the `platform-eks` Pulumi program. Upgrades
happen on our schedule: quarterly, proposed as a pull request, soaked in
staging for 48 hours before production.

# Consequences

* CNI upgrades are now visible in `pulumi preview` and reviewed like any
  other change.
* We accept a lag behind the newest CNI release, including for fixes; the
  quarterly cadence is a deliberate trade.
* The pin must actually be maintained — a stale pin is its own risk.

# Revisit when

* EKS offers staged/canary add-on rollouts for the CNI, or
* two consecutive quarterly upgrades are no-ops.
