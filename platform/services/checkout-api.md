---
type: service
title: Checkout API
description: Public HTTP API that turns a cart into an order and hands payment off to the payments worker.
resource: urn:pulumi:prod::platform-eks::kubernetes:apps/v1:Deployment::checkout-api
tags: [checkout, kubernetes, public-facing]
timestamp: 2026-07-14T09:00:00Z
owner: storefront-team
oncall: "#oncall-storefront"
---

# Overview

The Checkout API is the public entry point for order placement. It validates
the cart, creates the order record, and enqueues a payment job. It does not
talk to the payments database directly — all payment execution happens in the
[Payments Worker](/services/payments-worker.md).

# Operations

| Property   | Value                                            |
|------------|--------------------------------------------------|
| Runtime    | EKS, `prod` cluster, `checkout` namespace        |
| Deployed by| Pulumi project `platform-eks`, stack `prod`      |
| Ingress    | ALB, `checkout.example.com`                      |
| SLO        | 99.9% availability, p99 latency under 400 ms     |
| Scaling    | HPA on CPU, 4–24 replicas                        |

# Dependencies

* Enqueues payment jobs for the [Payments Worker](/services/payments-worker.md) via SQS.
* Runs on the shared EKS cluster, so it is affected by
  [Why we pin the VPC CNI version](/decisions/why-we-pin-the-vpc-cni-version.md).
