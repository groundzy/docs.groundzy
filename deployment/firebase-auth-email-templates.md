# Firebase Authentication email templates

Firebase sends **verification**, **password reset**, **email change**, and related messages using templates in the **Firebase Console** (Authentication → Templates). These are **not** sent through Resend and do **not** use `lib/email.ts`.

## Why document this

Product branding should feel consistent across **Resend** transactional mail (welcome, documents, intelligence) and **Firebase** security mail so users trust links and recognize Groundzy.

## Checklist (manual)

1. Open **Firebase Console** → **Authentication** → **Templates** for each relevant project (dev/staging/prod as applicable).
2. For **each template** (Email verification, Password reset, Email address change, etc.):
   - Set **sender name** and **reply-to** per your support policy.
   - Align **tone** with [`email.md`](../groundzy-v3-docs/08-integrations/email.md) (concise, professional).
   - Use **Action URL** domains that match your hosted auth/app URLs (verify authorized domains under Authentication → Settings).
3. Optional: add **custom HTML** with the same palette as transactional mail — header `#2d5a4a`, accent button `#8BC34A`, footer links to Terms and Privacy (`https://www.groundzy.com/terms.html`, `https://www.groundzy.com/privacy.html`).
4. Smoke-test each flow end-to-end (link opens correct environment, completes auth step).

## Related

- Groundzy transactional mail (Resend): [`groundzy-v3-docs/08-integrations/email.md`](../groundzy-v3-docs/08-integrations/email.md)
