# Sanitize Before You Share

The Task 5.1 sample log is fake, but I treated every secret, hostname, and phone number as if it were real.

Goal: make a version that is safe to paste into a hosted AI chat, while keeping enough detail to troubleshoot.

---

## Original problems in the sample log

The raw log had:

- AWS access key and secret key
- Database user and password in a URL
- Internal DB hostname and private IP (`10.20.4.17`)
- Customer email, phone, and order id
- GitHub-style bearer token (`ghp_...`)
- Internal payments gateway hostname

---

## Sanitized log (safe to paste)

```text
2026-11-02T09:11:03Z deploy[4121]: starting release 1.4.2 for orders-api
2026-11-02T09:11:04Z deploy[4121]: aws configure set aws_access_key_id <AWS_ACCESS_KEY_ID>
2026-11-02T09:11:04Z deploy[4121]: aws configure set aws_secret_access_key <AWS_SECRET_ACCESS_KEY>
2026-11-02T09:11:06Z orders-api[4188]: DATABASE_URL=postgres://<DB_USER>:<DB_PASSWORD>@<DB_HOST>:5432/orders
2026-11-02T09:11:07Z orders-api[4188]: connection to <DB_HOST> (<DB_IP>) failed: FATAL: password authentication failed for user "<DB_USER>"
2026-11-02T09:11:09Z orders-api[4188]: order #<ORDER_ID> failed for customer <CUSTOMER_EMAIL>, phone <CUSTOMER_PHONE>
2026-11-02T09:11:09Z orders-api[4188]: Authorization: Bearer <GITHUB_TOKEN>
2026-11-02T09:11:12Z orders-api[4188]: retry 3/5 against <PAYMENTS_HOST>:8443 timed out after 30s
```

---

## What I replaced, and why

| Original kind of value | Placeholder | Why |
| --- | --- | --- |
| AWS access key id | `<AWS_ACCESS_KEY_ID>` | Stops account takeover |
| AWS secret access key | `<AWS_SECRET_ACCESS_KEY>` | Same |
| Database user | `<DB_USER>` | Hides account name; reused in the FATAL line |
| Database password | `<DB_PASSWORD>` | Stops DB login with stolen creds |
| Database hostname | `<DB_HOST>` | Hides internal name; same placeholder on both lines |
| Private IP `10.20.4.17` | `<DB_IP>` | Hides internal network detail |
| Order id | `<ORDER_ID>` | Customer-related identifier |
| Customer email | `<CUSTOMER_EMAIL>` | Personal data |
| Customer phone | `<CUSTOMER_PHONE>` | Personal data |
| Bearer `ghp_...` token | `<GITHUB_TOKEN>` | Stops API / repo access |
| Payments gateway host | `<PAYMENTS_HOST>` | Different host from DB, so different placeholder |

I did **not** delete whole lines. I only replaced the sensitive values.

---

## What I left in (so the log still helps)

- Timestamps and process ids (`4121`, `4188`)
- Release `1.4.2` and service name `orders-api`
- Database name `orders`
- Ports `5432` and `8443`
- Error text: `password authentication failed`
- `retry 3/5` and `timed out after 30s`
- `Authorization: Bearer` wording (shows a token was logged, without the token)

**Story that remains:** deploy of `orders-api` 1.4.2 → DB password auth failed for the configured user → then a call to the payments host timed out after 30s.

---

## How I kept the log meaningful

- Same real value → same placeholder every time (`<DB_USER>` in the URL and in the FATAL line).
- Different hosts → different placeholders (`<DB_HOST>` vs `<PAYMENTS_HOST>`).
- Structure of the log stays the same, so an AI (or a teammate) can still reason about the failure.

---

## If I already pasted a real credential into a chat

Deleting the chat message is **not** enough.

1. **Revoke / disable the leaked key first** (AWS access key, GitHub token, etc.).
2. **Rotate the other secrets** that were in the same paste (DB password, and any other tokens).
3. **Check audit logs** (CloudTrail, GitHub audit log) for use after the paste time.
4. Then clean the chat / ticket text.

The secret may already sit in the provider’s store or in training/retention on the AI side. Revoke first, then clean up.
