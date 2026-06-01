---
icon: hand-wave
cover: .gitbook/assets/110ft QuickZip Straight.png
coverY: 0
description: Verify real-world identity in your product, without ever holding the data.
---

# Welcome to Self

Self verifies real-world identity using **zero-knowledge proofs**. Your users prove a fact about themselves, they're over 18, they're a unique human, they're not on a sanctions list, and you get a yes/no answer. You never see their passport, their date of birth, or their name.

**[Self Enterprise](self-enterprise/get-started/what-is-self-enterprise.md)** is the fastest way to use it: a dashboard, one SDK call, and a signed webhook. No smart contracts, no verifier to host, no PII to store.

{% hint style="success" %}
**New here?** Go from zero to a verified user in ten minutes with the **[Quickstart](self-enterprise/get-started/quickstart.md)**.
{% endhint %}

## Start with Self Enterprise

The managed platform. You configure a verification flow in the dashboard, call `sessions.create(...)`, hand the user a link, and receive a signed `verification.completed` webhook when they're done.

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Quickstart</strong></td><td>Verified user in ten minutes.</td><td><a href="self-enterprise/get-started/quickstart.md">quickstart.md</a></td></tr><tr><td><strong>Core concepts</strong></td><td>Orgs, flows, sessions, keys, webhooks.</td><td><a href="self-enterprise/get-started/concepts.md">concepts.md</a></td></tr><tr><td><strong>How verification works</strong></td><td>The zero-knowledge model, in plain language.</td><td><a href="self-enterprise/get-started/how-it-works.md">how-it-works.md</a></td></tr><tr><td><strong>SDK</strong></td><td>The <code>@selfxyz/enterprise-sdk</code> client.</td><td><a href="self-enterprise/sdk/nodejs.md">nodejs.md</a></td></tr><tr><td><strong>Webhooks</strong></td><td>Signed events, signature verification, idempotency.</td><td><a href="self-enterprise/webhooks/overview.md">overview.md</a></td></tr><tr><td><strong>API reference</strong></td><td>The REST endpoints behind the SDK.</td><td><a href="self-enterprise/api-reference/overview.md">overview.md</a></td></tr></tbody></table>

## What you can build

* **Sybil resistance**, admit only unique, real humans, without collecting personal data.
* **Age gates**, restrict access by age without ever learning a date of birth.
* **Geographic & sanctions compliance**, allow or deny by nationality, screen against OFAC.
* **Reusable KYC**, verify a user once, trust the attestation later.
* **Marketplace trust**, verify both sides of a trade without holding either party's PII.

## Open source & protocol

Self Enterprise runs on top of an open protocol. If you'd rather self-host the verifier, deploy your own contracts, or build on the lower layers directly, those products are open source and documented under **Open source & protocol** in the sidebar:

* **Self Pass**, the underlying identity protocol: frontend & backend SDKs, smart contracts, and the zero-knowledge proof system. (Self Enterprise is the managed version of this.)
* **Self Connect**, map off-chain identifiers (phone, email, social handles) to on-chain addresses.
* **Self Agent ID**, on-chain proof-of-human identity for AI agents (ERC-8004).

{% hint style="info" %}
These SDKs are being superseded by Self Enterprise for most use cases. They remain fully supported and open source, start with Enterprise unless you specifically need to self-host.
{% endhint %}

## Resources

* [Dashboard](https://dashboard.self.xyz), sign up and configure your first flow.
* [Interactive coverage map](https://map.self.xyz/), which documents and countries are supported.
* [Self Builder Group](https://t.me/+d2TGsbkSDmgzODVi), Telegram community for developers.
* [Status](https://status.self.xyz), live service status.
