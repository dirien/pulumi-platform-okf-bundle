# Pulumi Platform OKF Bundle

An [Open Knowledge Format](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) (OKF v0.1) knowledge bundle for a Pulumi platform team: the runbooks, decision records, and service pages that infrastructure as code never captures, in a format any coding agent can traverse.

Companion repo to the Pulumi blog post [Knowledge as Code: The Memory File Just Got a Standard](https://www.pulumi.com/blog/knowledge-as-code-the-memory-file-just-got-a-standard/).

## What's inside

The bundle root is [`platform/`](platform/):

```
platform/
├── index.md                              # Progressive disclosure: what's in the bundle
├── log.md                                # Append-only update history
├── services/
│   ├── index.md
│   ├── checkout-api.md
│   └── payments-worker.md
├── runbooks/
│   ├── index.md
│   └── rotate-database-credentials.md
└── decisions/
    ├── index.md
    └── why-we-pin-the-vpc-cni-version.md
```

Every concept document carries YAML frontmatter with the one field OKF requires (`type`) plus the recommended ones (`title`, `description`, `resource`, `tags`, `timestamp`). Where a concept describes a real piece of infrastructure, `resource` holds the Pulumi URN of that resource, so an agent can walk from a planned change to the runbook and the decision record that constrain it.

## Use it with your agent

1. **Teach your agent the standard.** Paste the [OKF spec](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) into your coding agent and tell it this is the format the bundle follows.
2. **Point it at the bundle.** Clone this repo (or hand the agent the repo URL) and tell it the bundle root is `platform/`. It should read `platform/index.md` first and drill down only when a concept is relevant.
3. **Ask questions.** For example:
   - "Which services touch the payments database?"
   - "Walk me through rotating the payments database credentials."
   - "Why is the VPC CNI version pinned, and when should we revisit that?"

## Notes

The content is illustrative: a fictional platform (one EKS cluster, two payment services, one Postgres instance). The shapes are real, the incident dates are not. Use it as a starting point for your own bundle — the fastest way is to hand your agent the spec plus your existing docs folder and have it refactor them into this structure.

Licensed under the [MIT License](LICENSE).
