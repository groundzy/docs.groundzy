# Bouncie OAuth 2.0 and API authentication

Source: [Bouncie API — Authorization](https://docs.bouncie.dev/#/) section of the official docs.

## Model

- Bouncie uses **OAuth 2.0 authorization code** flow: a **Bouncie account holder** grants your app access to their data.
- After consent, you receive an **authorization `code`**, exchange it for an **access token** and **refresh token**.
- REST requests use that **access token** (per authorized user). Webhooks are configured separately in the portal and use a **shared secret** header (see [webhooks.md](./webhooks.md)).

## Step 1 — Authorization request

Redirect the user to:

`https://auth.bouncie.com/dialog/authorize`

Query parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `client_id` | Yes | From Developer Portal |
| `response_type` | Yes | Must be `code` |
| `redirect_uri` | Yes | Must **exactly** match a URI registered in the portal (including slashes and casing) |
| `state` | No | CSRF protection; returned on redirect |
| `code_challenge` | No | PKCE: Base64URL-encoded `code_verifier` ([RFC 7636](https://datatracker.ietf.org/doc/html/rfc7636)) |
| `code_challenge_method` | No | `S256` (recommended) or `plain`; defaults to `S256` if `code_challenge` is set |
| `resource` | No | Protected resource URI (e.g. `https://api.bouncie.dev/v1/`); echoed as `aud` in the token ([RFC 8707](https://datatracker.ietf.org/doc/html/rfc8707)) |

Example:

```http
https://auth.bouncie.com/dialog/authorize?client_id=my-app&redirect_uri=https://app.example.com/oauth/bouncie&response_type=code&state=random-state
```

On success, the user is redirected to your `redirect_uri` with:

| Parameter | Meaning |
|-----------|---------|
| `code` | One-time authorization code (invalid if user re-authorizes and a new code is issued) |
| `state` | Echo of `state` if you sent it |

**Note:** After consent, if webhooks are configured for your app, you can start receiving events for that user even before token exchange (per official docs).

## Step 2 — Exchange code for tokens

`POST https://auth.bouncie.com/oauth/token`

Headers:

- `Content-Type: application/json`

JSON body:

| Field | Required | Description |
|-------|----------|-------------|
| `client_id` | Yes | Application client ID |
| `client_secret` | Yes | Application secret |
| `grant_type` | Yes | `authorization_code` |
| `code` | Yes | Code from step 1 |
| `redirect_uri` | Yes | Same registered URI (validation only; no redirect) |
| `code_verifier` | If using PKCE | Original verifier matching `code_challenge` |

**Alternative:** send `client_id` and `client_secret` using `Authorization: Basic <base64(client_id:client_secret)>` instead of putting them in the JSON body.

Successful response JSON:

| Field | Type | Description |
|-------|------|-------------|
| `access_token` | string | Use on REST API |
| `refresh_token` | string | Use to obtain new access tokens |
| `expires_in` | number | Access token lifetime in seconds |
| `token_type` | string | Always `Bearer` (but see REST header note below) |

## Step 3 — Refresh token

`POST https://auth.bouncie.com/oauth/token` with JSON:

| Field | Value |
|-------|--------|
| `grant_type` | `refresh_token` |
| `refresh_token` | Current refresh token |
| `client_id` | Required |
| `client_secret` | Required |

Response shape matches token exchange. **Each refresh invalidates the previous refresh token.** Unused refresh tokens eventually expire; re-run the authorization code flow if refresh fails.

## Calling the REST API

Official example pattern:

```http
Authorization: <access_token_only>
Content-Type: application/json
```

**Important (FAQ):** Do **not** prefix the access token with `Bearer ` for the Bouncie REST API — send the raw token string only.

## Troubleshooting 401

See [faq-and-operations.md](./faq-and-operations.md) — common causes are expired access token, wrong header format, or revoked app access for that user in the Developer Portal.
