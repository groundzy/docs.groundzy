# End-client invoice payments (Phase 4C architecture)

Groundzy’s existing Stripe **Customer Portal** ([`create-portal-session`](https://github.com/groundzy)) is for **subscriber billing** (Teams/Plus, etc.). **Client invoice payments** (homeowner pays an arborist invoice) must use a **separate** Stripe surface and webhook path so:

- Subscription webhooks in **auth.groundzy** are not mixed with job invoice lifecycle.
- PCI scope and dispute handling stay explicit.

## Recommended options (pick one product track)

1. **Stripe Connect (Standard or Express)**  
   Each team is a connected account; invoices create `PaymentIntent` or `Checkout` on the connected account. Best when you need payouts to the contractor’s bank and platform fees.

2. **Stripe Payment Links / Hosted Invoice** (simpler)  
   Staff or automation creates a one-off payment link for an `Invoice` total. Fewer moving parts; weaker embedded “hub” UX unless you iframe or redirect.

3. **Non-Stripe**  
   ACH instructions, “mail a check”, or external PSP — product-only; minimal engineering beyond displaying terms.

## Portal scope

When implemented, add `pay_invoice` to [`DocumentPortalScope`](c:/Groundzy/app/types/document-portal-token.ts) only on tokens minted for invoices in a **payable** state, and validate amount/status server-side before creating any payment object.

## Stub API

`POST /api/client-hub/invoice-pay` currently returns **503** with `invoice_pay_not_configured` until a PSP path is chosen and implemented.
