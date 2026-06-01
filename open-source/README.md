---
icon: code-branch
description: The open protocol underneath Self Enterprise, for teams that want to self-host.
---

# Open source & protocol

Self Enterprise is a managed service built on an **open protocol**. Everything Enterprise does for you, issuing credentials, verifying zero-knowledge proofs, maintaining the certificate registry, is also available as open-source SDKs and on-chain contracts you can run yourself.

{% hint style="info" %}
**Most teams should start with [Self Enterprise](../self-enterprise/get-started/what-is-self-enterprise.md).** It removes the infrastructure: no verifier to host, no contracts to deploy, no config store to manage, plus a dashboard, audit log, webhooks, and support. The open-source products below are for teams that specifically need to self-host or build on the lower layers.
{% endhint %}

## The products

### Self Pass

The underlying identity-verification protocol. Render a QR code with the frontend SDK, verify the proof with the backend SDK or on-chain with the contracts. This is what Self Enterprise manages on your behalf.

[Browse Self Pass →](../self-pass/README.md)

### Self Connect

An open protocol that maps off-chain identifiers, phone numbers, email addresses, social handles, to on-chain blockchain addresses, so users can transact with familiar names instead of hex addresses.

[Browse Self Connect →](../self-connect/introduction-and-overview.md)

### Self Agent ID

On-chain proof-of-human identity for AI agents. Each agent gets a soulbound ERC-721 NFT backed by a ZK passport verification, implementing the ERC-8004 standard.

[Browse Self Agent ID →](../agent-id/overview.md)

## Enterprise vs. self-hosted

| | Self Enterprise | Open-source SDKs |
| --- | --- | --- |
| Verifier infrastructure | Managed | You host it |
| Flow configuration | Dashboard | Your `ConfigStore` code |
| Credential storage & audit | Built in | You build it |
| Webhook delivery & retries | Built in | You build it |
| Billing & SLA | Yes | None (free, self-run) |
| Smart contracts | Not required | You deploy & maintain |
| Best for | Product teams shipping fast | Teams that must self-host |

If you're migrating from the open-source SDK to Enterprise, see **[Migrate → From the open-source SDK](../self-enterprise/migration/from-self-pass-sdk.md)**.
