# People

**Settings → People**. Manage who has access to the org.

> 📸 _**Screenshot:** People page with the members list and the **Invite member** button._

## Invite a teammate

Click **Invite member**:

* **Email**: they'll receive a sign-in link.
* **Role**: `owner`, `admin`, or `member` (see [Roles](#roles)).

The invite expires after 7 days. You can revoke a pending invite at any time.

## Roles

Every member has one of three roles:

| Role | Can do |
| --- | --- |
| **`owner`** | Everything, including billing actions (change plan, toggle the credit gate) and managing other owners. |
| **`admin`** | Manage flows, API keys, webhook endpoints, and invite members. Cannot perform owner-only billing actions. |
| **`member`** | Read access — view flows, the activity log, usage, and webhook delivery history. Cannot create or revoke keys, manage webhooks, or invite people. |

Only `owner` and `admin` can issue/revoke API keys and manage webhook endpoints. Billing's credit-gate toggle is `owner`-only.

## Members list

Each row shows:

| Column | Notes |
| --- | --- |
| Name + email | Captured at sign-up; updated when they accept. |
| Role | `owner` / `admin` / `member`. |
| Last active | Last dashboard session. |
| Joined | When they accepted. |

## Removing a member

**Remove** revokes their session and removes them from the org. Their personal sign-in identity is not affected; they just lose access to this org.

API keys they created stay valid, keys are owned by the org, not the member. Revoke keys separately if needed.

## Single sign-on (SSO)

For enterprise plans, the org can be wired to your SSO provider (SAML, OIDC, Google Workspace, Okta, etc.). Contact support@self.xyz to enable.

When SSO is enforced, invite-by-email is replaced by directory provisioning, new members appear automatically on first login from your IdP.

## Audit

Member additions, removals, and role changes are recorded in the org audit log (visible to admins; export available via the [Audit tab](../dashboard/billing.md)).
