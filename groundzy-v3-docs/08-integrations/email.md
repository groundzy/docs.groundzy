# Email (Resend)

## What it does

**Transactional email** via **Resend** in the **main app** codebase: tier **welcome** mail, **workflow document** delivery (quotes, invoices, jobs, requests), and **intelligence alert** mail (storm × elevated-risk trees). Shared HTML primitives live under **`lib/email/layout.ts`** (`wrapGroundzyTransactionalEmail`, `buildTeamDocumentEmail`, escaping helpers).

The **auth** signup app sends **welcome** email only, using the same Groundzy shell (`auth/lib/email/layout.ts`). When changing welcome visuals or legal/footer links, reconcile **both** repositories in one change set.

## Modules (paths relative to app repo unless noted)

| Flow | Location | Notes |
|------|-----------|--------|
| Welcome | `lib/email.ts` | Fire-and-forget after profile creation; tiers include **Enterprise**. Uses layout wrapper + preheader. |
| Documents + PDF | `lib/email.ts` (`sendDocumentEmail`), `app/api/documents/email/route.ts` | Team-branded shell (“Powered by Groundzy”). Primary CTA opens the **public document portal** at `/documents/{token}` (minted on send in `document_portal_tokens`, same idea as SMS / `portal-issue`). PDF is attached; the portal supports online PDF, quote approval when scoped, and Client Hub. **Reply-To:** verified Firebase **sender email** when present, else team **`businessEmail`**, else workflow **`documentsEmail.senderEmail`**. |
| Intelligence | `lib/notifications/intelligence-email-resend.ts`, `lib/notifications/intelligence-copy.ts` | Specific subjects/lines + branded HTML + “View tree” CTA. |

**Auth repo:** `auth/lib/email.ts` + `auth/lib/email/layout.ts` — welcome only (trimmed layout without document helpers).

## Environment

| Variable | Role |
|----------|------|
| `RESEND_API_KEY` | Required for sends (welcome skips silently if missing; document API errors if missing). |
| `RESEND_FROM_EMAIL` | Verified sender domain (e.g. `Groundzy <noreply@app.groundzy.com>`). |
| `NEXT_PUBLIC_APP_URL` | App base URL for welcome CTAs and **portal links** minted on document email/SMS (`https://…/documents/{token}`). |

See [`reference/environment-variables.md`](../../reference/environment-variables.md) § Email.

## Document delivery CTAs

Workflow document emails (quote, invoice, work order, service request) use a **fresh portal token per send**. The HTML button points at `{NEXT_PUBLIC_APP_URL}/documents/{token}` so recipients without a Groundzy account can open the PDF flow and (for quotes) approve when the token includes `approve_quote`. Observability: server logs `[document-portal]` with event **`portal_issue_email`** when minting from email.

## Document email merge tokens

Subjects and body templates (team workflow **`documentsEmail`**) support:

`{{companyName}}`, `{{clientName}}`, `{{documentType}}`, `{{documentTitle}}`, `{{documentNumber}}`, `{{totalAmount}}`, `{{currency}}`, `{{dueDate}}`, `{{validUntil}}`, `{{propertyAddress}}`

**Client name** comes from document **`recipient.name`** (CRM snapshot), not the recipient email local-part.

Team Settings exposes quote, invoice, **work order**, and **service request** subject templates.

## Risks and follow-ups

- **Deliverability:** SPF/DKIM in Resend; domain alignment with links in body.
- **Intelligence:** Prefer actionable copy (storm titles reference tree label + wind/alerts). Future: unsubscribe / prefs deep link when product exposes a stable URL.
- **Firebase Auth templates** (password reset, verification): edited in **Firebase Console**, not Resend — see [`deployment/firebase-auth-email-templates.md`](../../deployment/firebase-auth-email-templates.md).

## Related

- [`deployment/pdf-generation-app-hosting-retrospective.md`](../../deployment/pdf-generation-app-hosting-retrospective.md)
- [`07-systems/delivery-preferences-and-routing.md`](../07-systems/delivery-preferences-and-routing.md)
