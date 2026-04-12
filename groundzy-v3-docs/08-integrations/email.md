# Email (Resend)

## What it does

**Transactional email** via **Resend** — currently used for **welcome** emails after signup (tier-specific templates), with **fire-and-forget** behavior so signup is not blocked on send failure (`lib/email.ts`).

## How it’s used

| Piece | Detail |
|-------|--------|
| **SDK** | `resend` package (`package.json`) |
| **Module** | `lib/email.ts` — `Resend(process.env.RESEND_API_KEY)`, `resend.emails.send` |
| **From address** | `RESEND_FROM_EMAIL` or default `Groundzy <noreply@app.groundzy.com>` |
| **Links** | Uses `NEXT_PUBLIC_APP_URL`, terms/privacy URLs in templates |

**Callers:** Signup / onboarding flows that invoke welcome send (search for `sendWelcomeEmail` or imports of `@/lib/email`).

## Risks & constraints

| Risk | Note |
|------|------|
| **Domain verification** | Comment in code: from address must use **verified domain** in Resend (e.g. `app.groundzy.com`) |
| **Missing API key** | Sends may no-op or error; errors logged, not thrown (signup path) |
| **Deliverability** | Depends on Resend reputation, SPF/DKIM as configured in Resend dashboard |
| **Scope** | Not a full marketing automation suite — transactional only in surveyed module |

## Related

- `pro_contact_requests` / Cloud Functions may send **other** emails — verify `firebase/functions` or server code if present outside `lib/email.ts`
- `docs/reference/environment-variables.md` § Email
